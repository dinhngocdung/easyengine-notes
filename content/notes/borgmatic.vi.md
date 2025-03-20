---
title: "Borgmatic: Sao lưu server nhanh, rẻ, an toàn"
linkTitle: Borgmatic Backup
weight: 10
type: docs
prev: seo-security
---

**Borg/Borgmatic** là giải pháp sao lưu hiệu quả cho server web. Nó giúp nén và loại bỏ dữ liệu trùng lặp để tiết kiệm dung lượng, mã hóa để bảo vệ dữ liệu, và chỉ sao lưu phần thay đổi để tăng tốc độ. Borgmatic tự động hóa backup, kiểm tra tính toàn vẹn và khôi phục dễ dàng.  

**BorgBase** là dịch vụ lưu trữ rẻ, chuyên dụng cho Borg, hỗ trợ SSH key, giám sát và sao lưu dự phòng. Kết hợp với Borgmatic giúp bảo vệ dữ liệu web an toàn, tiết kiệm chi phí.  

Một lần nữa chúng ta sẽ triển khai Borgmatic trên Docker theo đúng tinh thần EasyEngine.  

---

## Tạo Borgmatic container

Tôi dùng hình ảnh Docker chính thức từ Borgmatic.  

Tạo thư mục cho Borgmatic và thao tác trong thư mục này:  

```bash
mkdir ~/borgmatic
mkdir -p data/{borgmatic.d,repository,.config,.ssh,.cache}
cd ~/borgmatic
```

Tạo file `docker-compose.yml` cho container Borgmatic:  

```bash
nano docker-compose.yml
```

Chép nội dung này vào, thay đổi `volumes` theo nhu cầu backup của bạn. Ví dụ, nếu bạn không dùng Fail2Ban, có thể bỏ dòng đó:  

```yaml
services:
  borgmatic:
    image: ghcr.io/borgmatic-collective/borgmatic
    container_name: borgmatic
    volumes:
      - /opt/easyengine:/mnt/source/easyengine:ro            
	      # backup EasyEngine docker-compose.yml
      - /var/lib/docker/volumes:/mnt/source/volumes:ro       
	      # backup volumes Docker data
      - /root/borgmatic:/mnt/source/borgmatic:ro             
	      # backup Borgmatic config, docker-compose.yml
      - /root/fail2ban:/mnt/source/fail2ban:ro               
	      # backup Fail2Ban config, docker-compose.yml
      - /root/restore:/restore                               
	      # restore data  
      - ./data/repository:/mnt/borg-repository               
	      # backup target
      - ./data/borgmatic.d:/etc/borgmatic.d/                 
	      # Borgmatic config file(s) + crontab.txt
      - ./data/.config/borg:/root/.config/borg               
	      # config và keyfiles
      - ./data/.ssh:/root/.ssh                               
	      # SSH key cho remote repositories
      - ./data/.cache/borg:/root/.cache/borg                 
	      # checksums dùng cho deduplication
      - /etc/localtime:/etc/localtime:ro  
	      # Đồng bộ múi giờ với host
      - /etc/timezone:/etc/timezone:ro    
	      # (Tùy chọn) Đồng bộ timezone
    environment:
		    # Thiết lập múi giờ cụ thể
      - TZ=Asia/Ho_Chi_Minh  
	      # Passphrase tự tạo
      - BORG_PASSPHRASE="passwordconnectborg"
    networks:
      global-backend-network:

networks:
  global-backend-network:
    external: true
    name: ee-global-backend-network
```

Khởi tạo Borgmatic Docker:  

```bash
docker-compose up -d
```

---

## Kết nối Borgmatic với BorgBase  

Trước khi nghĩ đến backup như thế nào, chúng ta cần chọn nơi lưu trữ backup. Với quy mô một website, dữ liệu khoảng 200GB, thì BorgBase là lựa chọn hợp lý với gói $2/tháng. Nếu trang web nhỏ, họ miễn phí luôn 10GB.  

Các bước thiết lập BorgBase:  

1. Đăng ký và tạo tài khoản nếu chưa có: [https://www.borgbase.com/register](https://www.borgbase.com/register)  
2. Đăng nhập và tạo kho lưu trữ **Repositories**, lưu lại URL của kho, thường có dạng:  
   ```
   ssh://123abc@def45678.repo.borgbase.com/./repo
   ```
3. Tạo khóa **SSH** trên Borgmatic container:  

```bash
# Kết nối shell Borgmatic Docker
cd ~/borgmatic && docker-compose exec borgmatic bash

# Tạo khóa SSH, bỏ qua khi nhắc passphrase
ssh-keygen -o -a 100 -t ed25519

# Xem khóa và lưu lại
cat ~/.ssh/id_ed25519.pub
```

4. Thêm **public key** vào BorgBase: Nhấn **Add Key** trong mục SSH Keys, dán public key vừa tạo vào, đặt tên tùy ý và lưu lại.  
5. Kết nối key vừa thêm vào repository: Nhấn nút chỉnh sửa, trong phần **Access**, chọn key vừa thêm, lưu lại và hoàn tất.  

---

## Thiết lập cách hoạt động của Borgmatic  

Mọi hoạt động của Borgmatic được cấu hình trong file `config.yaml`.  

Tạo file cấu hình:  

```bash
nano data/borgmatic.d/config.yaml
```

Chép nội dung sau vào, nhớ thay thế `repositories` và `encryption_passphrase` phù hợp:  

```yaml
source_directories:
    - /mnt/source/easyengine 
    - /mnt/source/volumes 
    - /mnt/source/borgmatic 
    - /mnt/source/fail2ban 

repositories:
    - path: ssh://123abc@def45678.repo.borgbase.com/./repo 
      label: "Backup for sample.com on BorgBase"

exclude_patterns:
    - '*.pyc'
    - ~/*/.cache

compression: auto,zstd
encryption_passphrase: "passphrase_borg"
archive_name_format: 'sample.com-{now:%Y-%m-%d-%H%M%S}'

retries: 5
retry_wait: 5

keep_daily: 7
keep_weekly: 4
keep_monthly: 12
keep_yearly: 5

checks:
    - name: disabled

check_last: 3

before_backup:
    - echo "`date` - Starting backup"

after_backup:
    - echo "`date` - Finished backup"

mariadb_databases:
    - name: sample_com
      hostname: services_global-db_1
      username: sample.com-AlJolB
      password: passwordmariadb_sample_com
```

Kiểm tra lại file config:  

```bash
docker-compose exec borgmatic borgmatic config validate
```

---

## Khởi tạo kho BorgBase  

Thực hiện trong Borgmatic container, thay `BORG_REPO=` bằng URL BorgBase repository của bạn:  

```bash
cd ~/borgmatic && docker-compose exec borgmatic bash
borgmatic --init --encryption repokey-blake2
export BORG_REPO=ssh://123abc@def45678.repo.borgbase.com/./repo
```

---

## Các lệnh quản lý Borgmatic  

Xem danh sách backup:  

```bash
docker-compose exec borgmatic borgmatic list
```

Xem các database được lưu trữ trong bản mới nhất:  

```bash
docker-compose exec borgmatic borgmatic list --archive latest --find *borgmatic/*_databases
```

Trích xuất file `path/1` trong bản lưu mới nhất và lưu vào `/restore`:  

```bash
docker-compose exec borgmatic borgmatic extract --archive latest --path path/1 --destination /restore
```

---

## Lên lịch chạy định kỳ  

### Dùng cron có sẵn trong Docker image  

Thêm lịch chạy vào file cấu hình cron của container:  

```bash
nano ~/borgmatic/data/borgmatic.d/crontab.txt
```

Thêm dòng:  

```bash
0 3 * * * PATH=$PATH:/usr/local/bin /usr/local/bin/borgmatic --stats -v 0 2>&1
```

### Chỉ chạy khi cần  

Nếu chỉ backup một lần mỗi ngày, cách tối ưu hơn là tắt container sau khi backup xong.  

Tắt Borgmatic container:  

```bash
cd ~/borgmatic && docker-compose down
```

Chỉnh sửa cron jobs:  

```bash
crontab -e
```

Thêm lịch trình:  

```bash
0 3 * * * cd /root/borgmatic/ && /usr/local/bin/docker-compose run --rm borgmatic borgmatic >> /root/borgmatic/cron.log 2>&1
```

Như vậy, 3 AM mỗi ngày, Docker sẽ chạy backup, sau đó container tự xóa.  

---

Tham khảo:  
- [Borgmatic](https://torsion.org/borgmatic/)  
- [BorgBase](https://www.borgbase.com/)  
- [Docker Borgmatic](https://github.com/borgmatic-collective/docker-borgmatic)