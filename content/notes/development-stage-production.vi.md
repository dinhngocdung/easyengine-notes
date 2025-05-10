---
title: Hướng dẫn triển khai Development, Stage và Production với EasyEngine 4 với macOS
linkTitle: Development, Stage và Production
weight: 11
type: docs
prev: borgmatic
next: migrate
---

## 1. Giới thiệu quy trình Dev-Stage-Production và EasyEngine 4

Quy trình **Development (Dev)**, **Stage**, và **Production** là tiêu chuẩn trong DevOps để phát triển và triển khai website hiệu quả:

- **Development**: Môi trường cục bộ để thử nghiệm plugin, theme, hoặc chức năng mới.
- **Stage**: Môi trường kiểm tra giống Production trước khi triển khai (có thể bỏ qua cho triển khai đơn giản).
- **Production**: Môi trường trực tiếp phục vụ người dùng.

**EasyEngine 4** tích hợp các lệnh `ee site clone`, `ee site sync`, và `ee site share` để hỗ trợ quy trình này. Bài viết hướng dẫn sử dụng EasyEngine 4 trên macOS, với các cờ chính xác cho `clone` và `sync`, sử dụng ảnh placeholder khi không đồng bộ thư mục `uploads` lớn, và triển khai cho website tại **YOUR-SITE.COM** trên server **YOUR-SERVER-IP**.

![Development, Stage, và Production](/images/development.svg)

## 2. Tạo môi trường Development cục bộ trên macOS

- Cài đặt **Docker Desktop** đã cài đặt trên macOS (tải tại Docker Desktop).
- Cài đặt **Homebrew** (nếu chưa có):
    
    ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```
    
- Cài đặt **EasyEngine 4** qua Homebrew:
    
    ```bash
    brew install easyengine
    ee --version
    ```
    
- Quyền SSH vào máy chủ Production (`root@YOUR-SERVER-IP`), với cặp khóa SSH để đồng
    
    ```bash
    ssh-keygen -t rsa
    ssh-copy-id root@YOUR-SERVER-IP
    ```
    

## 3. Tạo site Development trên macOS để phát triển

Để phát triển plugin, theme, hoặc chức năng, tạo site Dev cục bộ (`dev.YOUR-SITE.test`) từ site Production (`YOUR-SITE.COM`).

### 3.1. Clone site từ Production sang Dev

Theo EasyEngine Clone, lệnh `ee site clone` hỗ trợ các cờ:

- `-db`: Chỉ clone database.
- `-files`: Chỉ clone file (trừ `uploads`).
- `-uploads`: Chỉ clone thư mục `uploads`.
- Không có cờ: Clone toàn bộ (database, file, và `uploads`).

Để tránh lỗi xác thực SSL, thêm cờ `--ssl=off`. Nếu chỉ cần database và file (không cần `uploads` lớn):

```bash
ee site clone root@YOUR-SERVER-IP:YOUR-SITE.COM dev.YOUR-SITE.test --db --files --ssl=off
```

**Lưu ý**:

- Lệnh yêu cầu cấu hình SSH đến `root@YOUR-SERVER-IP`.

Cấu hình DNS cục bộ:

```bash
sudo nano /etc/hosts
```

Thêm dòng:

```
127.0.0.1 dev.YOUR-SITE.test
```

Kiểm tra site Dev tại `http://dev.YOUR-SITE.test`.

### 3.2. Thêm Placeholder cho hình ảnh

Nếu không clone `uploads`, hình ảnh trên Dev có thể bị lỗi. Thêm code placeholder để thay thế ảnh không tồn tại.

Mở file `functions.php` của theme con:

```bash
nano ~/easyengine/sites/dev.YOUR-SITE.test/app/htdocs/wp-content/themes/blocksy-child/functions.php
```

Thêm đoạn code:

```php
// IMAGE PLACEHOLDER
function replace_broken_images_with_placeholder($content) {
    $placeholder = 'https://sanantoniosports.org/wp-content/uploads/2022/07/placeholder-image.jpeg';

    // Sử dụng regex để thay thế ảnh không tồn tại
    return preg_replace_callback('/<img[^>]+src=["\']([^"\']+)["\'][^>]*>/i', function($matches) use ($placeholder) {
        $src = $matches[1];

        // Kiểm tra xem ảnh có tồn tại trên server hay không
        $path = ABSPATH . str_replace(site_url('/'), '', $src);
        if (!file_exists($path)) {
            // Thay bằng ảnh placeholder
            return str_replace($src, $placeholder, $matches[0]);
        }

        return $matches[0];
    }, $content);
}
add_filter('the_content', 'replace_broken_images_with_placeholder');

// Nếu ảnh sản phẩm không tồn tại, trả về ảnh placeholder
function custom_wc_product_image_placeholder($image, $attachment_id, $size, $icon) {
    $file_path = get_attached_file($attachment_id);
    $placeholder_url = 'https://sanantoniosports.org/wp-content/uploads/2022/07/placeholder-image.jpeg';

    if (!file_exists($file_path)) {
        // Trả về ảnh placeholder nếu file không tồn tại
        $image = [$placeholder_url, 600, 600, false];
    }

    return $image;
}
add_filter('wp_get_attachment_image_src', 'custom_wc_product_image_placeholder', 10, 4);

add_filter('woocommerce_placeholder_img_src', function() {
    return 'https://sanantoniosports.org/wp-content/uploads/2022/07/placeholder-image.jpeg';
});
```

Lưu file và thoát.

### 3.3. Đồng bộ từ Production sang Dev (nếu cần)

Khi Production có thay đổi, dùng `ee site sync` để đồng bộ. Theo EasyEngine Sync, cờ giống `clone`:

- `-db`: Chỉ đồng bộ database.
- `-files`: Chỉ đồng bộ file (trừ `uploads`).
- `-uploads`: Chỉ đồng bộ thư mục `uploads`.
- Không có cờ: Đồng bộ toàn bộ.

Thông thường, chỉ đồng bộ database:

```bash
ee site sync root@YOUR-SERVER-IP:YOUR-SITE.COM dev.YOUR-SITE.test --db
```

### 3.4. Xóa cache trên Dev

Xóa cache để áp dụng thay đổi:

```bash
ee site clean dev.YOUR-SITE.test

```


## 4. Triển khai từ Dev cục bộ lên Production

Khi plugin, theme, hoặc chức năng trên Dev sẵn sàng, bạn có thể triển khai trực tiếp từ Dev sang Production (bỏ qua Stage cho các dự án đơn giản).

### 4.1. Đồng bộ database

Đồng bộ database từ Dev sang Production:

```bash
ee site sync dev.YOUR-SITE.test root@YOUR-SERVER-IP:YOUR-SITE.COM --db
```

### 4.2. Xóa Placeholder trước khi đồng bộ file

Nếu có thay đổi files và cần đồng bộ, Code placeholder chỉ dùng cho Dev, nên xóa trước:

```bash
nano ~/easyengine/sites/dev.YOUR-SITE.test/app/htdocs/wp-content/themes/YOUR-THEME/functions.php
```

Xóa đoạn code placeholder.

### 4.3. Đồng bộ file

Đồng bộ file, bỏ qua `uploads` nếu không cần:

```bash
ee site sync dev.YOUR-SITE.test root@YOUR-SERVER-IP:YOUR-SITE.COM --files

```

### 4.4. Xóa cache trên Production

Xóa cache để áp dụng thay đổi:

```bash
ee site clean YOUR-SITE.COM
```


## 5. Tùy chọn: Thiết lập môi trường Stage

Môi trường Stage dùng để kiểm tra trước khi triển khai, nhưng có thể bỏ qua cho các dự án đơn giản. Nếu cần Stage, có hai phương án:

### 5.1. Triển khai trên server remote

Tạo site Stage trên server remote, ví dụ **STAGE-SERVER-IP**:

- Thiết lập server tương tự môi trường local:
    - Chạy Ubuntu/Debian và cài đặt EasyEngine 4.
    - Clone/Sync đồng bộ từ Dev:
        
        ```bash
        ee site clone dev.YOUR-SITE.test root@STAGE-SERVER-IP:stage.YOUR-SITE.COM --db --files --ssl=off
        ```
        
    - Sync `uploads` từ Production nếu cần:
        
        ```bash
        ee site sync root@YOUR-SERVER-IP:YOUR-SITE.COM stage.YOUR-SITE.COM --uploads
        ```
        
- Kểm tra trên `stage.YOUR-SITE.COM`.

### 5.2. Share trực tiếp từ macOS với ngrok

Với nhu cầ đơn giản, có thể kiểm thử trực tiếp bằng môi trường dev với khách hàng bằng cách share.

Tạo URL tạm thời từ môi trường Dev trên macOS để kiểm tra nhanh:

```bash
ee site share dev.YOUR-SITE.test
```

- Tạo URL công khai qua ngrok.
- Giới hạn: Không hỗ trợ tên miền tùy chỉnh, thời gian tồn tại mặc định 24 giờ.
- Giới hạn 1 giờ với `-expire`:
    
    ```bash
    ee site share dev.YOUR-SITE.test --expire=3600
    ```
    


## 6. Tổng kết

EasyEngine 4, cài đặt qua `brew install easyengine`, giúp triển khai và quản lý môi trường Dev, Stage, và Production trên macOS. Các lệnh `ee site clone` và `ee site sync` với cờ `--db`, `--files`, `--uploads`, cùng `--ssl=off` cho clone, cho phép kiểm soát dữ liệu, kết hợp ảnh placeholder để tối ưu khi không cần `uploads`. Lệnh `ee site share` hỗ trợ kiểm tra nhanh trên Stage. Với các dự án đơn giản, bạn có thể triển khai trực tiếp từ Dev (`dev.YOUR-SITE.test`) sang Production (`YOUR-SITE.COM` trên `YOUR-SERVER-IP`), bỏ qua Stage.

Hãy làm quen với quy trình Dev-Stage-Production để trở thành nhà triển khai website chuyên nghiệp. Với EasyEngine 4, bạn sẽ tiết kiệm thời gian và đảm bảo chất lượng triển khai.


**Nguồn**:
- [EasyEngine Clone](https://easyengine.io/commands/site/clone/)
- [EasyEngine Sync](https://easyengine.io/commands/site/sync/)
- [EasyEngine Share](https://easyengine.io/commands/site/share/)