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
vi /opt/easyengine/sites/YOUR-SITE.COM/config/nginx/conf.d/main.conf
```

Replacing `Redis NGINX CONFIGURATION` with the provided snippet and using `YOUR-SITE.COM` as a placeholder:

```nginx
## # CUSTOM Redis NGINX CONFIGURATION
# Cache with tracking parameters like Facebook, Google, UTM, etc.

# Initialize cache control flags
set $skip_fetch 0;  # 0 = allow fetching from cache, 1 = skip cache
set $skip_store 0;  # 0 = allow storing to cache, 1 = skip storing to cache

# Rule 1: Completely skip cache for POST requests
if ($request_method = POST) {
    set $skip_fetch 1;
    set $skip_store 1;
}

# Rule 2: Skip cache for common query strings
# (Only allow caching for empty query strings or tracking parameters)
if ($query_string !~* "(^$|srsltid=|utm_.*=|fbclid=|_gl=|gclid=|gad_source=|dclid=|wbraid=|gbraid=|gclsrc=)") {
    set $skip_fetch 1;
    set $skip_store 1;
}

# Rule 3: Special handling for tracking parameters
# - Allow fetching from cache but do not store to cache
if ($query_string ~* "(srsltid=|utm_.*=|fbclid=|_gl=|gclid=|gad_source=|dclid=|wbraid=|gbraid=|gclsrc=)") {
    set $skip_store 1;  # Prevent storing to cache
    # $skip_fetch remains = 0 to allow fetching from cache
}

# Rule 4: Skip cache for admin pages
if ($request_uri ~* "(/wp-admin/|/wp-login.php|/xmlrpc.php|wp-.*.php|index.php|/feed/|sitemap(_index)?.xml)") {
    set $skip_fetch 1;
    set $skip_store 1;
}

# Rule 5: Skip cache for logged-in users
if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_no_cache|wordpress_logged_in|woocommerce_items_in_cart") {
    set $skip_fetch 1;
    set $skip_store 1;
}

# Create $clean_uri variable (removes query string)
set $clean_uri $request_uri;
if ($request_uri ~ "^(.*)\?.*$") {
    set $clean_uri $1;
}

# Main request handling
location / {
    try_files $uri $uri/ /index.php?$args;
}

# Endpoint for fetching cache from Redis (internal)
location /redis-fetch {
    internal;
    set $redis_key $args;
    redis_pass redis;
}

# Endpoint for storing cache to Redis (internal)
location /redis-store {
    internal;
    set_unescape_uri $key $arg_key;
    redis2_query set $key $echo_request_body;
    redis2_query expire $key 86400;  # Cache TTL 24 hours
    redis2_pass redis;
}

# PHP processing with caching mechanism
location ~ \.php$ {
    # Create cache key in the required format
    set $key "YOUR-SITE.COM_page:http$request_method$host$clean_uri";
    
    # Adjust key for HTTPS connection
    if ($HTTP_X_FORWARDED_PROTO = "https") {
        set $key "YOUR-SITE.COM_page:https$request_method$host$clean_uri";
    }

    try_files $uri =404;

    # Apply cache control flags
    srcache_fetch_skip $skip_fetch;
    srcache_store_skip $skip_store;

    srcache_response_cache_control off;
    set_escape_uri $escaped_key $key;

    # Perform cache fetch/store
    srcache_fetch GET /redis-fetch $key;
    srcache_store PUT /redis-store key=$escaped_key;

    # Debug headers
    more_set_headers 'X-SRCache-Fetch-Status $srcache_fetch_status';
    more_set_headers 'X-SRCache-Store-Status $srcache_store_status';

    include fastcgi_params;
    fastcgi_pass php;
}
```
