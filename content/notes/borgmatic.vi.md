---
title: "Borgmatic Docker: Sao lưu website nhanh, rẻ, an toàn"
linkTitle: Sao lưu với Borgmatic
weight: 10
type: docs
prev: seo-security
---

**Borg/Borgmatic** là giải pháp sao lưu hiệu quả cho server web. Nó giúp nén và loại bỏ dữ liệu trùng lặp để tiết kiệm dung lượng, mã hóa để bảo vệ dữ liệu, và chỉ sao lưu phần thay đổi để tăng tốc độ. Borgmatic tự động hóa backup, kiểm tra tính toàn vẹn và khôi phục dễ dàng.  

**BorgBase** là dịch vụ lưu trữ rẻ, chuyên dụng cho Borg, hỗ trợ SSH key, giám sát và sao lưu dự phòng. Kết hợp với Borgmatic giúp bảo vệ dữ liệu web an toàn, tiết kiệm chi phí.  

![Borgmatic Docker on EasyEngine](/images/borgmatic-docker-easyengine.svg)

Một lần nữa chúng ta sẽ triển khai Borgmatic trên Docker theo đúng tinh thần EasyEngine. 

## Tạo Borgmatic container

Tôi dùng hình ảnh Docker chính thức từ Borgmatic.  

Tạo thư mục cho Borgmatic và thao tác trong thư mục này:  

```bash
mkdir -p /opt/borgmatic/data/{borgmatic.d,repository,.config,.ssh,.cache}
```

Download các file  `docker-compose.yml` và `borgmatic.d/config.yaml` cho container Borgmatic, thay đổi `volumes` trong `docker-compose.yml` theo nhu cầu backup của bạn. Ví dụ, nếu bạn không dùng Fail2Ban, có thể bỏ dòng đó. Mặc định của tôi ở dây là sử dụng backup các website, borgmatic, fail2ban, crontab, database

```bash
curl -o /opt/borgmatic/docker-compose.yml https://raw.githubusercontent.com/dinhngocdung/easyengine-stack/refs/heads/main/borgmatic/docker-compose.yml
curl -o /opt/borgmatic/data/borgmatic.d/config.yaml https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/borgmatic/data/borgmatic.d/config.yaml
```

Download và chỉnh sửa file `.env`, hoặc copy-paste từ local (Password, repo, passphare repo...)
```bash
curl -o /opt/borgmatic/.env -L https://github.com/dinhngocdung/easyengine-dock-stack/raw/refs/heads/main/borgmatic/.env

# Chỉnh sửa các biến phù hợp
vi /opt/borgmatic/.env
```

Khởi tạo Borgmatic Docker:  

```bash
sudo docker compose -f /opt/borgmatic/docker-compose.yml up -d
```

## Kết nối Borgmatic với BorgBase  

Trước khi nghĩ đến backup như thế nào, chúng ta cần chọn nơi lưu trữ backup. Với quy mô một website, dữ liệu khoảng 200GB, thì BorgBase là lựa chọn hợp lý với gói $2/tháng. Nếu trang web nhỏ, họ miễn phí luôn 10GB.  

Các bước thiết lập BorgBase:  

1. Đăng ký và tạo tài khoản nếu chưa có: [https://www.borgbase.com/register](https://www.borgbase.com/register)  
2. Đăng nhập và tạo kho lưu trữ **Repositories**, lưu lại URL của kho, thường có dạng:  
   ```
   ssh://XXXXX@XXXXX.repo.borgbase.com/./repo
   ```
3. Tạo khóa **SSH** trên Borgmatic container:  

    ```bash
    # Kết nối shell Borgmatic Docker
    cd /opt/borgmatic && sudo docker compose exec borgmatic bash

    # Tạo khóa SSH, bỏ qua khi nhắc passphrase
    ssh-keygen -o -a 100 -t ed25519

    # Xem khóa và lưu lại
    
    cat ~/.ssh/id_ed25519.pub
    ```

4. Thêm **public key** vào BorgBase: 
  Nhấn **Add Key** trong mục SSH Keys, dán public key vừa tạo vào, đặt tên tùy ý và lưu lại.  
5. Kết nối key vừa thêm vào repository: 
  Nhấn nút chỉnh sửa, trong phần **Access**, chọn key vừa thêm, lưu lại và hoàn tất. 
6. Với repo borgbase mới hoàn toàn, khởi tạo lần đầu bằng cách:
    ```bash
    cd /opt/borgmatic && \
    sudo docker compose up -d
    borgmatic init -e repokey-blake2
    borgmatic create
    ```

## Các lệnh quản lý Borgmatic  

Xem danh sách backup:  

```bash
sudo docker compose exec borgmatic borgmatic list
```

Xem các database được lưu trữ trong bản mới nhất:  

```bash
sudo docker compose exec borgmatic borgmatic list --archive latest --find *borgmatic/*_databases
```

Trích xuất file `path/1` trong bản lưu mới nhất và lưu vào `/restore`:  

```bash
sudo docker compose exec borgmatic borgmatic extract --archive latest --path path/1 --destination /restore
```

## Lên lịch chạy định kỳ  

### Dùng cron có sẵn trong Docker image  

Thêm lịch chạy vào file cấu hình cron của container:  

```bash
nano /opt/borgmatic/data/borgmatic.d/crontab.txt
```

Thêm dòng:  

```bash
0 3 * * * PATH=$PATH:/usr/local/bin /usr/local/bin/borgmatic --stats -v 0 2>&1
```

### Chỉ chạy khi cần  

Nếu chỉ backup một lần mỗi ngày, cách tối ưu hơn là tắt container sau khi backup xong.  

Tắt Borgmatic container:  

```bash
cd /opt/borgmatic && sudo docker compose down
```

Chỉnh sửa cron jobs:  

```bash
crontab -e
```

Thêm lịch trình:  

```bash
0 3 * * * cd /root/borgmatic/ && sudo /usr/local/bin/docker compose run --rm borgmatic borgmatic >> /root/borgmatic/cron.log 2>&1
```

Như vậy, 3 AM mỗi ngày, Docker sẽ chạy backup, sau đó container tự xóa.  

Tham khảo:  
- [Borgmatic](https://torsion.org/borgmatic/)  
- [BorgBase](https://www.borgbase.com/)  
- [Docker Borgmatic](https://github.com/borgmatic-collective/docker-borgmatic)
