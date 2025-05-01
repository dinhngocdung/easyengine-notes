---
title: Hướng dẫn di chuyển server EasyEngine sang server mới
linkTitle: Chuyển server
weight: 12
type: docs
prev: development-stage-production
next: review
---

Đây là cách di chuyển server chạy **EasyEngine** từ OLD-SERVER-IP sang server mới (NEW-SERVER-IP), đảm bảo các website WordPress (ví dụ: `YOUR-SITE.COM`) hoạt động bình thường. Quy trình sử dụng `rsync` để đồng bộ dữ liệu và khởi động lại các dịch vụ Docker. 

**Tổng quan về quy trình**

Di chuyển server EasyEngine yêu cầu sao chép các thư mục chính (`/opt/easyengine` và `/var/lib/docker/volumes/`) từ server cũ sang server mới. Các bước bao gồm:

1. Thiết lập server mới.
2. Cài đặt EasyEngine, Docker, và kiểm tra `rsync`.
3. Di chuyển dữ liệu `/opt/easyengine`  và `/var/lib/docker/volumes/` từ server cũ sang mới.
4. Chạy các container Docker để khởi động dịch vụ và website.

## 1. Thiết lập server mới

1. **Khởi tạo server mới**:
    
    Tạo máy chủ mới với Ubuntu/Debian, update `apt update && apt upgrade -y`
    
2. **Cấu hình SSH**:
    - Đảm bảo truy cập server mới qua SSH (`root@NEW-SERVER-IP`).
    - (Tùy chọn) Tạo cặp khóa SSH để đồng bộ dữ liệu an toàn:
        
        ```bash
        ssh-keygen -t rsa
        ssh-copy-id root@OLD-SERVER-IP
        ```
        

## 2. Cài đặt EasyEngine, Docker, và kiểm tra rsync

1. **Cài đặt EasyEngine**:
    - EasyEngine tự động cài đặt Docker và phụ thuộc. Chạy lệnh:
        
        ```bash
        wget -qO ee https://rt.cx/ee4 && sudo bash ee
        ```
        
    - Kiểm tra cài đặt:
        
        ```bash
        ee --version
        docker --version
        docker-compose --version
        ```
        
    
    **Lưu ý**: Không tạo site mới hoặc chạy lệnh EasyEngine nào khác trước khi đồng bộ dữ liệu.
    
2. **Kiểm tra và cài đặt rsync**:
    - Kiểm tra `rsync`: `rsync --version`
    - Nếu chưa có, cài đặt:
        
        ```bash
        apt update
        apt install -y rsync
        ```
        


## 3. Đồng bộ dữ liệu từ server cũ sang server mới

Đồng bộ hai thư mục: `/opt/easyengine` (cấu hình EasyEngine) và `/var/lib/docker/volumes/` (dữ liệu Docker như database, file website) bằng `rsync`.

1. **Đồng bộ** `/opt/easyengine`:
    
    ```bash
    rsync -avz --progress root@OLD-SERVER-IP:/opt/easyengine/ /opt/easyengine/
    
    ```
    
    - `a`: Lưu giữ quyền, thời gian, và cấu trúc thư mục.
    - `v`: Hiển thị chi tiết.
    - `z`: Nén dữ liệu để truyền nhanh hơn.
    - `-progress`: Hiển thị tiến độ.
2. **Đồng bộ** `/var/lib/docker/volumes/`:
    
    ```bash
    rsync -avz --progress root@OLD-SERVER-IP:/var/lib/docker/volumes/ /var/lib/docker/volumes/
    ```
    
    Xác minh các thư mục tại server mới:
    
    ```bash
    ls -l /opt/easyengine
    ls -l /var/lib/docker/volumes
    ```
    
3. **Khởi động lại**
    
    Khởi động lại để hệ thống nhận diện thay đổi : `reboot`, Đợi vài phút và đăng nhập lại qua SSH (`root@NEW-SERVER-IP`).
    

## 4. Chạy các container Docker

1. **Khởi động dịch vụ EasyEngine**:
    - Chạy file `docker-compose.yml` chính:
        
        ```bash
        docker-compose -f /opt/easyengine/services/docker-compose.yml up -d
        ```
        
    - Khởi động dịch vụ toàn cục (Nginx proxy, MariaDB, Redis).
2. **Khởi động các site**:
    - Chạy các file `docker-compose.yml` của từng site:
        
        ```bash
        for compose in /opt/easyengine/sites/*/docker-compose.yml; do
            docker-compose -f $compose up -d
        done
        ```
        
    - Khởi động container cho các website (Nginx, PHP, WordPress).
3. **Kiểm tra trạng thái**:
    - Xác minh container đang chạy, bạn sẽ thấy tất cả các service của docker compose đều đang chạy, kể cả các service bạn hiếm hoặc hoang toàn không dùng, như New Relic, Mailhog, Postfix:
        
        ```bash
        docker ps
        ```
    - Stop các container nếu bạn không dùng, Mailhog, New Relic, Postfix:

        New Relic
         ```bash
        docker-compose -f /opt/easyengine/services/docker-compose.yml stop global-newrelic-daemondocker-compose -f /opt/easyengine/services/docker-compose.yml stop global-newrelic-daemon
        ```

        Mailhog
         ```bash
        for compose in /opt/easyengine/sites/*/docker-compose.yml; do
            website=$(basename "$(dirname "$compose")")
            ee mailhog disable $website
        done
        ```

        Postfix
        ```bash
        for compose in /opt/easyengine/sites/*/docker-compose.yml; do
            docker-compose -f $compose stop postfix
        done
        ```
    - Để xoá hoàn toan các container dư thừa này, bằng cách sử dụng các lệnh tương tự, chỉ cần thay thế lệnh `docker-compose stop` bằng `docker-compose rm -v`.
        
    - Kiểm tra site:
        
        ```bash
        ee site list
        ```
        


## Một số lưu ý

- **DNS**: Cập nhật DNS để trỏ `YOUR-SITE.COM` sang NEW-SERVER-IP.
- **SSL**: Kiểm tra chứng chỉ SSL
- **Kiểm tra website**: Truy cập `YOUR-SITE.COM` để đảm bảo hoạt động.


## Kết luận

Di chuyển server EasyEngine từ OLD-SERVER-IP sang NEW-SERVER-IP là một quy trình đơn giản với các bước: thiết lập server mới, cài đặt EasyEngine và Docker, đồng bộ dữ liệu bằng `rsync`, và khởi động container. Quy trình này đảm bảo các website WordPress như `YOUR-SITE.COM` hoạt động liền mạch trên server mới.

Credit: [Stephen](https://github.com/EasyEngine/easyengine/discussions/1895)