---
title: Mẹo nhỏ Bảo mật và SEO với EasyEngine
linkTitle: Security & SEO
weight: 9
type: docs
prev: seo-security
next: borgmatic
---

Một số biện pháp bảo mật và tối ưu tôi đã áp dụng cho hệ thống

## Bảo mật xmlrpc.php  

`xmlrpc.php` vẫn cần thiết cho các chức năng của WooCommerce, nhưng nó thường bị tấn công. Dù không nguy hiểm, nó vẫn có thể tạo tải không đáng có cho server. Cách tôi làm là chỉ cho phép IP của Jetpack truy cập endpoint này.  

Chỉnh sửa file `user.conf`:  

```bash
nano /opt/easyengine/sites/YOUR-SITE.COM/config/nginx/custom/user.conf
```

Thêm đoạn sau vào:  

```nginx
location = /xmlrpc.php {
    # Cho phép các IP từ Jetpack
    allow 122.248.245.244;
    allow 54.217.201.243;
    allow 54.232.116.4;
    allow 192.0.80.0/20;
    allow 192.0.96.0/20;
    allow 192.0.112.0/20;
    allow 195.234.108.0/22;
    allow 192.0.64.0/18;

    # Chặn tất cả IP khác
    deny all;

    # Trả về trang 404 khi IP khác truy cập xmlrpc.php
    try_files $uri =404;
}
```

Reload lại Nginx của site:  

```bash
ee site reload YOUR-SITE.COM 
```

## Bảo mật SSH  

Khóa cổng 22 và chỉ cho phép một mình bạn đăng nhập SSH. Nhớ thay thế IP của bạn vào:  

```bash
nft add rule ip filter INPUT ip saddr 123.123.123.123 tcp dport 22 accept
nft add rule ip filter INPUT tcp dport 22 drop
```

Khi địa chỉ IP của bạn thay đổi, hãy thêm IP mới vào, ví dụ `124.124.124.124`:  

```bash
nft insert rule ip filter INPUT ip saddr 124.124.124.124 tcp dport 22 accept
```

Xóa IP cũ khỏi whitelist nếu không dùng nữa:  

```bash
# Liệt kê danh sách rule
nft -a list chain ip filter INPUT 

# Xác định handle của IP cần xóa, ví dụ handle là 1
nft delete rule ip filter INPUT handle 1 
```

## Tối ưu cache khi chạy quảng cáo  

EasyEngine Redis cache hoạt động tốt, nhưng khi chạy quảng cáo, Google và Facebook thường thêm các query string vào URL, ví dụ `fbclid=`, `gclid=`, `_gl=`,... Điều này gây ra hai vấn đề:  

1. Khách truy cập từ quảng cáo không được cache do URL thay đổi liên tục.  
2. Redis có thể lưu cache với các query string này, dẫn đến quá nhiều bản cache trùng nội dung.  

Giải pháp: Chỉ sử dụng URL gốc để kiểm tra cache trong Redis. Nếu đã có cache, Redis phục vụ nội dung từ cache, nếu chưa có, cache sẽ được lưu với key là URL gốc.  

Chỉnh sửa file `main.conf` (lưu ý các thay đổi này sẽ bị mất khi cập nhật `ee cli update`, cần lưu lại để áp dụng lại sau):  

```bash
nano /opt/easyengine/sites/YOUR-SITE.COM/config/nginx/conf.d/main.conf
```

Thêm đoạn sau (thay `YOUR-SITE.COM` bằng domain của bạn):  

```nginx
# Bỏ qua cache với các query string theo dõi từ Google, Facebook, quảng cáo
if ($query_string !~* "(^$|srsltid=|utm_.*=|fbclid=|_gl=|gclid=|gad_source=|dclid=|wbraid=|gbraid=|gclsrc=)") {
    set $skip 1;
}

# Tạo biến $clean_uri bằng cách loại bỏ query string khỏi $request_uri
set $clean_uri $request_uri;
if ($request_uri ~ "^(.*)\\?.*$") {
    set $clean_uri $1;
}

# Cấu hình lưu cache trong Redis
location /redis-store {
    internal;
    set_unescape_uri $key $arg_key;
    redis2_query set $key $echo_request_body;
    # Chỉnh thời gian cache, ví dụ tăng lên 7 ngày
    redis2_query expire $key 604800;
    redis2_pass redis;
}

# Tạo key cache chỉ dựa trên URL gốc đã xử lý
set $key "YOUR-SITE.COM_page:http$request_method$host$clean_uri"; 

# Nếu sử dụng HTTPS, cập nhật key cho phù hợp
if ($http_x_forwarded_proto = "https") {
    set $key "YOUR-SITE.COM_page:https$request_method$host$clean_uri";
}
```
