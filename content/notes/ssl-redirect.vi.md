---
title: Thiết Lập SSL, Domain và Chuyển hướng Redirect
linkTitle: SSL, Redirects
weight: 5
type: docs
prev: binlog
next: cache
---

Tất cả các vấn đề liên quan đến SSL, Domain, Alias domain đều được EasyEngine thiết lập sẵn, điều khiển với các dòng lệnh.  

## Thiết lập SSL  

SSL được thiết lập ngay với cờ `--ssl=le`, cùng với các thiết lập wildcard, self… Ví dụ:  

```bash
# Thiết lập SSL Let's Encrypt
ee site create example.com --ssl=le --wildcard  

# Thiết lập SSL cho website đã tồn tại  
ee site update example.com --ssl=le  
```  

Tham khảo: [EasyEngine Site Create](https://easyengine.io/commands/site/create/)  

## Quản lý Domain  

**Thêm domain:** Thông qua lệnh tạo website, domain cũng được thiết lập tự động.  

**Đổi tên miền:** Nếu bạn muốn đổi domain sang một tên mới, thực tế không có lệnh này, nhưng bạn có thể sử dụng lệnh `clone` với tên mới và sau đó `delete` website cũ.  

```bash
# Clone site cũ sang site mới  
ee site clone old-domain.com new-domain.com  

# Xóa site cũ  
ee site delete old-domain.com  
```  

Tham khảo:  

- [EasyEngine Site Clone](https://easyengine.io/commands/site/clone/)  
- [EasyEngine Site Delete](https://easyengine.io/commands/site/delete/)  

## **www, non-www**  

Khi bạn tạo trang với www hoặc non-www, EasyEngine sẽ tự động tạo alias cho tên còn lại. Nếu bạn muốn thêm một alias khác:  

```bash
ee site update example.com --add-alias-domains='a.com,*.a.com,b.com,c.com'
```  

## Redirect  

Nếu bạn muốn thêm các redirect, hãy chèn chúng vào `user.conf` hoặc tạo một file `.conf` mới trong thư mục `nginx/custom/`. Những thay đổi này sẽ được giữ nguyên khi cập nhật EasyEngine.  

```bash
nano /opt/easyengine/sites/sample.com/config/nginx/custom/user.conf
```  

Sau khi chỉnh sửa, restart Nginx:  

```bash
ee site reload sample.com
```  

Lưu ý rằng file cấu hình này sẽ được chèn vào khối `server` của Nginx cho từng site cụ thể.  

