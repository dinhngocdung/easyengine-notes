---
title: Triển khai WordPress với EasyEngine 4
linkTitle: Triển khai với Wordpress
weight: 3
type: docs
prev: differences
next: binlog
---

Đến đây, bạn đã tự tin lựa chọn và có đủ kiến thức cơ bản để hình dung về cách hoạt động của EasyEngine 4. Bây giờ, chúng ta có thể bắt đầu từng bước triển khai.

## Cài đặt EasyEngine 4

EasyEngine 4 có thể cài đặt trên bấy kỳ hệ điều hành nào, miễn có docker/docker-compose, PHP. Nhưng riêng Ubuntu/Debian, Easyengine cung cấp một trình cài đặt dễ dàng để cài đặt luôn các phụ thuộc cần thiết. Ở đây tôi mặc định dùnng Debian 12, (nhóm phát triển thử nghiệm trên debian/ubuntu).

Cài đặt bằng một lệnh đơn giản:

```bash
wget -qO ee https://rt.cx/ee4 && sudo bash ee
```

Bạn có thể cài đặt tính năng tự động hoàn thành lệnh (Tab Completion) để sử dụng EasyEngine 4 thuận tiện hơn:

```bash
wget -qO ~/.ee-completion.bash https://raw.githubusercontent.com/EasyEngine/easyengine/master/utils/ee-completion.bash
echo 'source ~/.ee-completion.bash' >> ~/.bash_profile
source ~/.ee-completion.bash
```
[**Cài đặt trên RHEL/CentOS, openSUSE, và các bản phân phối khác**](/vi/notes/rhel-cenos-opensuse/)

Tham khảo: [**Installation Guide**](https://easyengine.io/handbook/install/)

## Tạo website WordPress

EasyEngine tự động cài đặt, cấu hình và kết nối các container của site và các dịch vụ toàn cục.

```bash
ee site create YOUR-SITE.COM --type=wp --ssl=le --cache
```

Giải thích:

- `YOUR-SITE.COM`: thay bằng domain/subdomain của bạn.
- `--type=wp`: tạo website WordPress (mặc định, EasyEngine tạo website HTML đơn giản).
- `--ssl=le`: tự động cài đặt SSL với Let’s Encrypt.
- `--cache`: cài đặt full-page cache + Redis object cache.

Tham khảo thêm về lệnh tạo website: [ee site create](https://easyengine.io/commands/site/create/)

## Quản lý vòng đời WordPress

**Liệt kê danh sách website đang có trên máy chủ:**
```bash
ee site list
```

**Thêm alias domain:**
```bash
ee site alias YOUR-SITE.COM YOUR-ALIAS.COM
```

**Tạm dừng (disable) một website:**
```bash
ee site disable YOUR-SITE.COM
```

**Khởi động lại các dịch vụ Nginx, PHP mà không khởi động lại container:**
```bash
ee site reload YOUR-SITE.COM
```

**Khởi động lại website và cập nhật chỉnh sửa `docker-compose.yaml`:**
```bash
ee site refresh YOUR-SITE.COM
```

**Cập nhật toàn bộ server (bao gồm dịch vụ toàn cục và site):**
```bash
ee cli update
```

**Clone một website, ví dụ tạo `dev.YOUR-SITE.COM` từ `YOUR-SITE.COM` để thử nghiệm tính năng mới:**
```bash
ee site clone YOUR-SITE.COM dev.YOUR-SITE.COM
```

**Truy cập vào shell trong container của website (để chạy lệnh quản lý WordPress qua WP-CLI):**
```bash
ee shell YOUR-SITE.COM
```

**Xem thông tin chi tiết của website (vị trí lưu trữ, user, password, database…):**
```bash
ee site info YOUR-SITE.COM
```

**Xóa website:**
```bash
ee site remove YOUR-SITE.COM
```

Tham khảo danh sách các lệnh quản lý website: [ee site](https://easyengine.io/commands/site/)

## Chuyển website WordPress có sẵn vào EasyEngine

{{% steps %}}

### Chuẩn bị dữ liệu

- **Source site**: thư mục mã nguồn website, thường là `/wp-content/`.
- **Database**: file database (tốt nhất là sử dụng `wp db export` để xuất file này).

### Tạo website WordPress trên EasyEngine (không bật SSL để tránh lỗi)

```bash
ee site create YOUR-SITE.COM --type=wp --ssl=no --cache
```

### Sao chép source vào thư mục EasyEngine

```bash
# Sao chép source
rsync -avhP /path/to/source/wp-content/ /opt/easyengine/sites/YOUR-SITE.COM/app/htdocs/wp-content/
```

Sau khi sao chép, kiểm tra và đảm bảo quyền sở hữu là `www-data:www-data`.

### Nhập database vào website EasyEngine

```bash
# Sao chép file database vào thư mục htdocs
rsync -avhP /path/to/source/database.sql /opt/easyengine/sites/YOUR-SITE.COM/app/htdocs/

# Truy cập vào shell container
ee shell YOUR-SITE.COM

# Import database sử dụng WP-CLI
wp db import database.sql

# Flush cache và thoát khỏi shell
wp cache flush && exit
```

### Cấu hình lại website

- Kiểm tra lại file `wp-config.php`.
- Kiểm tra quyền sở hữu trên thư mục `wp-content/` (phải là `www-data:www-data`).
- Cập nhật DNS về server EasyEngine.

### Kích hoạt SSL

```bash
ee site update YOUR-SITE.COM --ssl=le
```

### Kiểm tra và kích hoạt cache

EasyEngine sử dụng **nginx-helper plugin**, đảm bảo plugin này được bật và đang sử dụng **Redis**.

{{% /steps %}}


Tham khảo thêm:  
[EasyEngine Commands](https://easyengine.io/commands/)