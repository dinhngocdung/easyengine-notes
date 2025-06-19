---
title: EasyEngine trong Contianer – chạy trên bất kỳ bản phân phối Linux nào
linkTitle: Trong Container
weight: 14
type: docs
prev: rhel-centos-opensuse
next: review
---

Tạo một container **EasyEngine**, cho phép bạn triển khai và sử dụng EasyEngine trên **mọi bản phân phối Linux**, không chỉ giới hạn ở Debian/Ubuntu như mặc định.

Hoạt động tốt trên các hệ điều hành như:

* CentOS, RHEL, AlmaLinux
* Arch, OpenSUSE, Fedora
* Các hệ điều hành tối giản hoặc tập trung cho container như:
  **Fedora CoreOS**, **OpenSUSE Leap Micro**, **Alpine**, v.v.


## Yêu cầu hệ thống

Máy của bạn cần **đã cài đặt và chạy Docker**

Kiểm tra Docker bằng lệnh:

```bash
docker version
```

Nếu chưa cài đặt, bạn có thể xem hướng dẫn chính thức tại:
👉 [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)


## Cách triển khai

Chạy lệnh sau để khởi động container EasyEngine:

```bash
docker run -it --rm --privileged \
  -v /var/run/docker.sock:/var/run/docker.sock:z \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  -v /opt/easyengine:/opt/easyengine \
  -v /etc/localtime:/etc/localtime:ro \
  -v /opt/easyengine/.ssh-key:/root/.ssh \
  --network host \
  --name ee-container \
  dinhngocdung/easyengine:latest
```

Giải thích nhanh các tham số:

| Tham số                   | Ý nghĩa                                                         |
| ------------------------- | --------------------------------------------------------------- |
| `--privileged`            | Cho phép container tương tác với Docker trên máy chủ            |
| `--rm`                    | Xóa container ngay khi bạn thoát ra                             |
| `-v /var/run/docker.sock` | Kết nối Docker bên trong container với Docker của máy chủ       |
| `-v /opt/easyengine`      | Lưu trữ dữ liệu cấu hình và website của EasyEngine trên máy chủ |
| `--network host`          | Sử dụng mạng của host để đảm bảo web/db hoạt động bình thường   |
| `-v /etc/localtime:...`   | Đồng bộ thời gian với host                                      |


## Cách sử dụng

Khi đã vào bên trong container (`ee-container`), bạn có thể sử dụng các lệnh EasyEngine như bình thường.

Ví dụ lệnh:

```bash
ee site create example.com --type=wp
ee site list
ee site disable example.com
ee site delete example.com
```

Để thoát khỏi container, gõ:

```bash
exit
```

Sau khi thoát, container sẽ được xóa tự động.
Tuy nhiên, tất cả dữ liệu EasyEngine và website vẫn được **giữ nguyên** trên máy chủ tại thư mục `/opt/easyengine`.


## Sử dụng lại EasyEngine

Mỗi khi cần dùng EasyEngine, chỉ cần **chạy lại lệnh [`docker run`](#cách-triển-khai)** ở trên để khởi động container mới.


## Đồng bộ/Sao chép (Sync/Clone)

Các lệnh `Sync/Clone` được thiết kế để tương tác giữa các cài đặt EasyEngine trực tiếp trên host. Để các lệnh này hoạt động với `ee-container`, bạn cần thực hiện các bước sau:

### ee-container cục bộ

Tạo ssh-key để connect với remote easyengine

**Nếu easyengine máy cụ bộ chạy trên container**
```
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub YOUR-USER@YOUR-REMOTE-SERVER.com
```
**Nếu easyengine máy cục bộ chạy trực tiếp trên host**
1.  Sử dụng một khóa kết nối chuyên dụng, khác với khóa chính, để truy cập host nhằm đảm bảo vẫn kiểm soát được host.
    ```bash
    ssh-keygen -t ed25519 -f ~/.ssh/id_ee_container
    ssh-copy-id -i ~/.ssh/id_ee_container.pub YOUR-USER@YOUR-REMOTE-SERVER.com
    ```
2.  Chỉ định sử dụng ssh-key `id_ee_container` khi kết nối với remote easyengine (`YOUR-REMOTE-SERVER.com`):
    ```bash
    echo "Host YOUR-REMOTE-SERVER.com
        HostName YOUR-REMOTE-SERVER.com
        User YOUR-USER
        IdentityFile ~/.ssh/id_ee_container
        IdentitiesOnly yes" >> ~/.ssh/config
    ```
    
### Host của ee-container từ xa

Nếu easyengine chạy trực tiếp trên remote host, mọi thứ bình thường. Chú ý khi `root` bị vô hiệu qua, bạn cần chuyển tiếp cho user hiện tại:
 ```bash
 vi /home/YOUR-USER/.ssh/authorized_keys

 # Thêm lệnh command... và trruwocs khoá ssh...
 command="if [ -n \"$SSH_ORIGINAL_COMMAND\" ]; then sudo -i bash -c \"$SSH_ORIGINAL_COMMAND\"; else sudo -i; fi" ssh-....
 ```
Nếu remote easyengine chạy trong container, bạn cần forward ssh vào `ee-container`


1.  **Tạo một Bash Script `/usr/local/bin/ssh_to_ee_container.sh` để chuyển tiếp các lệnh `ssh` và `rsync`:**
    ```bash
    #!/bin/bash

    # Tên của container Docker bạn muốn kết nối
    CONTAINER_NAME="ee-container"

    # Kiểm tra nếu có lệnh được truyền vào từ SSH_ORIGINAL_COMMAND
    if [ -n "$SSH_ORIGINAL_COMMAND" ]; then
        docker exec -i "$CONTAINER_NAME" /bin/bash -c "$SSH_ORIGINAL_COMMAND"
    else
        if [ -n "$SSH_TTY" ]; then
            docker exec -it "$CONTAINER_NAME" /bin/bash
        else
            docker exec -i "$CONTAINER_NAME" /bin/bash
        fi
    fi
    ```
2.  **Chỉnh sửa `~/.ssh/authorized_keys` trên host từ xa để chạy `ssh_to_ee_container.sh` khi truy cập bằng `ssh-key`:**
    ```bash
    vi ~/.ssh/authorized_keys
    ```
    Chèn `command="/usr/local/bin/ssh_to_ee_container.sh"` trước `ssh-...`
    ```bash
    command="/usr/local/bin/ssh_to_ee_container.sh" ssh-ed25519 AAAAB3NzaC1yc2EAAAADAQABAAABAQ... your_key_comment_or_email
    ```

## Tham khảo

* Trang chủ EasyEngine: [https://easyengine.io/](https://easyengine.io/)
* Ghi chú cá nhân về EasyEngine: [https://easyengine.pages.dev/](https://easyengine.pages.dev/)
* [Dockerfile](https://github.com/dinhngocdung/easyengine-container/blob/main/Dockerfile) để build image `dinhngocdung/easyengine:latest`
