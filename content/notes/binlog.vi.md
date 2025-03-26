---
title: Lỗi Database Connection do Binlog MariaDB trong EasyEngine
linkTitle: Lỗi Binlog Database
weight: 4
type: docs
prev: deploying
next: ssl-redirect
---

Tôi không rõ lý do vì sao nhóm phát triển EasyEngine cấu hình Global MariaDB theo cách hiện tại, nhưng có một lỗi mà rất nhiều người gặp phải khi triển khai EasyEngine. Do nó vốn dĩ dễ dàng như tên gọi, người dùng thường nhanh chóng triển khai nhưng sau đó lại gặp tình trạng server không thể hoạt động.  

Biểu hiện phổ biến là website hiển thị lỗi `'Error establishing a database connection'`, trong khi container database liên tục khởi động lại.  

Kinh nghiệm với LEMP stack truyền thống cũng khó giúp bạn khắc phục sự cố này. Lỗi này phổ biến đến mức nó trở thành một trong những nguyên nhân chính làm lu mờ nỗ lực và sự xuất sắc của EasyEngine 4.  

![Binlog MariaDB](/images/binlog-mariadb.svg)

## Binary Log (Binlog) của MariaDB  

MariaDB có một cơ chế ghi log hiện đại gọi là Binary Log (Binlog), giúp ghi nhận mọi thay đổi đối với dữ liệu, ngoại trừ các truy vấn chỉ đọc như `SELECT`. Binlog chủ yếu được sử dụng cho:  

- **Replication**: Đồng bộ dữ liệu giữa master và slave trong mô hình master-slave replication.  
- **Khôi phục dữ liệu**: Hỗ trợ phục hồi dữ liệu bằng cách phát lại các giao dịch từ binlog sau khi khôi phục từ backup.  
- **Theo dõi thay đổi**: Ghi lại lịch sử thay đổi dữ liệu để phân tích hoặc kiểm tra bảo mật.  

## Binlog Chiếm Hết Dung Lượng Ổ Cứng  

Binlog bao gồm nhiều tệp nhị phân (`.000001`, `.000002`,...) và một tệp index (`.index`) lưu danh sách các file binlog. Các file này được lưu tại thư mục:  

```bash
/var/lib/docker/volumes/global-db_db_logs/_data
```  

Cấu hình mặc định của EasyEngine tạo các file binlog có kích thước gần 100MB. Theo thời gian, binlog chiếm một dung lượng lớn, và trong nhiều trường hợp, nó làm đầy ổ cứng, khiến container database liên tục khởi động lại.  

## Xóa Các File Binlog Để Giải Phóng Bộ Nhớ  

Không thể xóa binlog thủ công vì điều này sẽ khiến MariaDB không hoạt động. Thay vào đó, chúng ta phải xóa bằng các lệnh SQL. Vì EasyEngine lưu database trong global services, chúng ta cần thực hiện các lệnh này trong container database.  

### 1. Truy cập vào container MariaDB  

EasyEngine lưu mật khẩu root của MariaDB trong biến `MYSQL_ROOT_PASSWORD`, ta có thể đăng nhập vào container bằng lệnh:  

```bash
cd /opt/easyengine/services && docker-compose exec global-db bash -c 'mysql -uroot -p${MYSQL_ROOT_PASSWORD}'
```  

### 2. Xóa các file binlog cũ bằng SQL  

```sql
-- Xem danh sách các file binlog
SHOW BINARY LOGS;  

-- Xác định file binlog hiện tại (file này không được xóa)
SHOW MASTER STATUS;  

-- Xóa các file binlog cũ hơn `mariadb-bin.000200` để giải phóng dung lượng
PURGE BINARY LOGS TO 'mariadb-bin.000200';  
```  

## Cấu Hình Lại MariaDB Trong EasyEngine Để Tránh Lặp Lại Lỗi  

Để ngăn binlog tiếp tục chiếm đầy ổ cứng, chúng ta cần điều chỉnh cấu hình MariaDB của EasyEngine.  

### 1. Chỉnh sửa file cấu hình MariaDB  

Mở file cấu hình:  

```bash
nano /var/lib/docker/volumes/global-db_db_conf/_data/conf.d/ee.cnf
```  

### 2. Vô hiệu hóa binlog hoặc giới hạn thời gian lưu trữ  

Thêm dấu `#` trước các dòng liên quan đến `log_bin` và `sync_binlog`. Đồng thời, đặt `expire_logs_days = 1` để tự động xóa binlog sau 1 ngày:  

```ini
[mysqld]
# log_bin                 = /var/log/mysql/mariadb-bin
log_bin_index           = /var/log/mysql/mariadb-bin.index
binlog_format           = statement
# Không tốt cho hiệu suất, nhưng an toàn hơn
# sync_binlog            = 1
expire_logs_days        = 1
max_binlog_size         = 100M
```  

Sau khi chỉnh sửa, khởi động lại dịch vụ MariaDB để áp dụng thay đổi:  

```bash
docker-compose restart global-db
```  

## Tham Khảo  

- [Hướng dẫn sử dụng Binary Log trong MySQL](https://snapshooter.com/learn/mysql/enable-and-use-binary-log-mysql)  
- [Tài liệu chính thức của MariaDB về Binary Log](https://mariadb.com/kb/en/binary-log/)  