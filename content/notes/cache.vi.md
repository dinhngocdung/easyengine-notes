---
title: "Redis & Proxy-Cache: Giải Pháp Cache Toàn Diện Cho WordPress"
type: docs
prev: ssl-redirect
next: fail2ban
---

EasyEngine đã khôn ngoan khi chỉ sử dụng một công nghệ cache là Redis. Điều này giúp họ giảm công sức quản lý các lựa chọn khác không mấy khác biệt và ngày càng tối ưu hơn trong việc xử lý cache.  

Khi sử dụng EasyEngine, bạn không cần dùng thêm bất kỳ công nghệ/plugin cache nào khác. Nó đã quá đủ và tốt. Hãy giữ website của bạn đơn giản như vậy.  

## **Fullpage Cache Redis**  

Fullpage cache là công nghệ mạnh nhất, lưu trữ toàn bộ trang web dưới dạng HTML trong RAM, được quản lý bởi Redis. Nếu URL được kích hoạt, truy xuất sẽ không cần đến PHP/MariaDB.  

```mermaid
graph LR
    A([Users]) <--> B((Internet))
    B -- "request (port 80,443)" --> C{Nginx Proxy}

    C -- "1 forward request Site A" --> D{Nginx Site A}
    D <-."2 cache Site A".-> E[(Global Redis)]
    D <-- "3 no cache" --> I[PHP Site A]
    I <-- "request data Site A" --> F[(Global MariaDB)]
    D -- "4 return response" --> C
    C -- "delivery response" --> B

    C -- "1 forward request Site B" --> D2{Nginx Site B}
    D2 <-."2 cache Site B".-> E[(Global Redis)]
    D2 <-- "3 no cache" --> I2[PHP Site B]
    I2 <-- "request data Site B" --> F[(Global MariaDB)]
    D2 -- "4 return response" --> C

    %% Màu sắc theo nhóm, giữ nguyên số lượng màu gốc
    style C fill:#FFDD57,stroke:#333,stroke-width:2px,color:#000 
    style D fill:#A6C8FF,stroke:#333,stroke-width:2px,color:#000 
    style D2 fill:#A6C8FF,stroke:#333,stroke-width:2px,color:#000  
    style I fill:#A6C8FF,stroke:#333,stroke-width:2px,color:#000 
    style I2 fill:#A6C8FF,stroke:#333,stroke-width:2px,color:#000  
    style F fill:#FFDD57,stroke:#333,stroke-width:2px,color:#000 
    style E fill:#FFDD57,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5,color:#000  

    subgraph DOCKER HOST
        C
        E
        F
        subgraph Wordpress Site App A
            D
            I
        end
        subgraph Wordpress Site App B
            D2
            I2
        end
    end
```

## **Object Cache Redis**  

Các trang động sẽ bị loại trừ khỏi fullpage cache. Lúc này, object cache hoạt động thay vì truy xuất database. Các object được lưu sẵn trong RAM và truy xuất nhanh hơn. Nó hữu ích với các trang động và khu vực admin.  

## **Kích hoạt Redis Cache**  

Với cờ `--cache` khi tạo website, cả fullpage cache và object cache đều được cài đặt và cấu hình sẵn cho website WordPress.  

```bash
ee site create example.com --cache
```

Nó được thiết lập ngay khi tạo site và không có lệnh sẵn để bật/tắt cho website đã có sẵn.  

Có hai plugin được cài sẵn khi bạn bật `--cache` để hỗ trợ chức năng này:  

1. **nginx-helper**: Giúp bạn bật/tắt, xóa cache từng trang/toàn bộ Redis cache ngay trong admin, thiết lập tự động xóa cache khi có update…  
2. **object-cache.php**: Được thiết lập dưới dạng mu-plugin, thực hiện chức năng object cache. Bạn không cần thao tác gì với nó, và nó cũng không có giao diện điều khiển trong admin.  

## **Sử dụng Redis-CLI**  

Truy cập vào Global Redis để sử dụng các lệnh Redis-CLI:  

```bash
# Vào container global redis
cd /opt/easyengine/services && docker-compose exec global-redis bash
# Thực hiện các lệnh redis-cli, ví dụ: xem keys của sample.com
redis-cli KEYS "sample.com_page:*"
```

## **Proxy Cache**  

Trong các trang web có lưu lượng truy cập cao, khi cache hết hạn, nhiều yêu cầu có thể cùng lúc gửi đến PHP để tạo lại cùng một trang, dẫn đến quá tải và cạn kiệt tài nguyên PHP.  

Để ngăn chặn tình trạng này, cơ chế **proxy cache** cho phép NGINX-PROXY phục vụ phiên bản cache đã hết hạn cho các yêu cầu tiếp theo trong khi chỉ gửi một yêu cầu duy nhất đến PHP để làm mới cache.  

Bộ nhớ đệm proxy-cache mặc định là 1 giây, sau đó hết hạn nhưng không bị xóa ngay. Khi bộ nhớ đệm hết hạn, Nginx-Proxy container tiếp tục phục vụ bộ nhớ đệm cũ trong khi chỉ có một yêu cầu được gửi đến Nginx để làm mới cache.  

```mermaid
graph LR
    B((Internet))-- "request
    (port 80,443)" --> C{NGINX PROXY}

    C -- "2
    request build cache" --> D{SITE NGINX}
    C <--"1
    cache"--> E[(PROXY CACHE)]
    D <-- "3
    cache" --> I[(REDIS CACHE)]

    %% Màu sắc theo nhóm, giữ nguyên số lượng màu gốc
    style C fill:#FFDD57,stroke:#333,stroke-width:2px,color:#000 
    style D fill:#A6C8FF,stroke:#333,stroke-width:2px,color:#000 
    style I fill:#FFDD57,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5,color:#000  
    style E stroke-dasharray: 5 5 
```

**Quy trình kiểm tra cache:**  

1. **Nếu cache còn** → phục vụ từ bộ nhớ đệm.  
2. **Nếu cache hết hạn** → phục vụ từ bộ nhớ đệm cũ và chuyển yêu cầu đến trang web Nginx.  
3. **Nếu cache không tồn tại** → gửi yêu cầu đến site Nginx.  

**Kích hoạt Proxy-cache:**  

```bash
ee site update example.com --proxy-cache=on
```

Tham khảo: [Proxy Cache in EasyEngine](https://easyengine.io/handbook/proxy-cache-as-stale-on-nginx-proxy-in-easyengine-v4/)  
