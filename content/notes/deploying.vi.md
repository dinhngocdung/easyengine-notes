---
title: Triển khai WordPress với EasyEngine 4
type: docs
prev: differences
next: binlog
---

Đến đây, bạn đã tự tin lựa chọn và có đủ kiến thức cơ bản để hình dung về cách hoạt động của EasyEngine 4. Bây giờ, chúng ta có thể bắt đầu từng bước triển khai.

## Cài đặt EasyEngine 4

EasyEngine 4 có thể cài đặt trên bất kỳ hệ điều hành nào, miễn là có Docker, Docker Compose và PHP. Tuy nhiên, trong hướng dẫn này, tôi sử dụng **Debian 12** (đội ngũ phát triển EasyEngine thử nghiệm chủ yếu trên Debian/Ubuntu).

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

Tham khảo: [**Installation Guide**](https://easyengine.io/handbook/install/)

## Tạo website WordPress

EasyEngine tự động cài đặt, cấu hình và kết nối các container của site và các dịch vụ toàn cục.

```bash
ee site create sample.com --type=wp --ssl=le --cache
```

Giải thích:

- `sample.com`: thay bằng domain/subdomain của bạn.
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
ee site alias example.com alias.com
```

**Tạm dừng (disable) một website:**
```bash
ee site disable example.com
```

**Khởi động lại các dịch vụ Nginx, PHP mà không khởi động lại container:**
```bash
ee site reload sample.com
```

**Khởi động lại website và cập nhật chỉnh sửa `docker-compose.yaml`:**
```bash
ee site refresh sample.com
```

**Cập nhật toàn bộ server (bao gồm dịch vụ toàn cục và site):**
```bash
ee cli update
```

**Clone một website, ví dụ tạo `sample.test` từ `sample.com` để thử nghiệm tính năng mới:**
```bash
ee site clone sample.com sample.test
```

**Truy cập vào shell trong container của website (để chạy lệnh quản lý WordPress qua WP-CLI):**
```bash
ee shell sample.com
```

**Xem thông tin chi tiết của website (vị trí lưu trữ, user, password, database…):**
```bash
ee site info sample.com
```

**Xóa website:**
```bash
ee site remove sample.com
```

Tham khảo danh sách các lệnh quản lý website: [ee site](https://easyengine.io/commands/site/)

## Chuyển website WordPress có sẵn vào EasyEngine

**1. Chuẩn bị dữ liệu**

- **Source site**: thư mục mã nguồn website, thường là `/wp-content/`.
- **Database**: file database (tốt nhất là sử dụng `wp db export` để xuất file này).

**2. Tạo website WordPress trên EasyEngine (không bật SSL để tránh lỗi)**

```bash
ee site create sample.com --type=wp --ssl=no --cache
```

**3. Sao chép source vào thư mục EasyEngine**

```bash
# Sao chép source
rsync -avhP /path/to/source/wp-content/ /opt/easyengine/sites/sample.com/app/htdocs/wp-content/
```

Sau khi sao chép, kiểm tra và đảm bảo quyền sở hữu là `www-data:www-data`.

**4. Nhập database vào website EasyEngine**

```bash
# Sao chép file database vào thư mục htdocs
rsync -avhP /path/to/source/database.sql /opt/easyengine/sites/sample.com/app/htdocs/

# Truy cập vào shell container
ee shell sample.com

# Import database sử dụng WP-CLI
wp db import database.sql

# Flush cache và thoát khỏi shell
wp cache flush && exit
```

**5. Cấu hình lại website**

- Kiểm tra lại file `wp-config.php`.
- Kiểm tra quyền sở hữu trên thư mục `wp-content/` (phải là `www-data:www-data`).
- Cập nhật DNS về server EasyEngine.

**6. Kích hoạt SSL**

```bash
ee site update sample.com --ssl=le
```

**7. Kiểm tra và kích hoạt cache**

EasyEngine sử dụng **nginx-helper plugin**, đảm bảo plugin này được bật và đang sử dụng **Redis**.

---

Tham khảo thêm:  
[EasyEngine Commands](https://easyengine.io/commands/)