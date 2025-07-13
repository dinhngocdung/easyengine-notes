---
title: EasyEngine trong Contianer – chạy trên bất kỳ bản phân phối Linux nào
linkTitle: Easyengine-Docker
weight: 14
type: docs
prev: rhel-centos-opensuse
next: review
---
## Hướng dẫn nhanh

```bash
# "cài đặt" easyengine, thiết lập lệnh ngắn ee và sẵn sàng chạy
mkdir -p ~/easyengine && \
curl -o ~/easyengine/docker-compose.yml https://raw.githubusercontent.com/dinhngocdung/easyengine-docker/master/docker-compose.yml && \
echo -e "\n\nalias ee='sudo docker compose -f $HOME/easyengine/docker-compose.yml run --rm easyengine'" >> ~/.bashrc && source ~/.bashrc

# Chạy ee-containr, và dùng các lệnh easyeinge
ee
```

## Chạy với mọi bản phân phối Linux

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
  --name easyengine \
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

Khi đã vào bên trong container (`easyengine`), bạn có thể sử dụng các lệnh EasyEngine như bình thường.

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

Điều này sẽ đưa bạn trở lại hệ điều hành máy chủ (host OS) và container sẽ tự động bị xóa.
Tuy nhiên, tất cả dữ liệu và website của EasyEngine vẫn **nguyên vẹn** trên máy chủ tại thư mục `/opt/easyengine`.

Bất cứ khi nào bạn cần sử dụng EasyEngine, chỉ cần **chạy lại lệnh triển khai [`docker run` debloy](#how-to-deploy) để khởi động môi trường container mới ngay lập tức.

## Sử dụng Docker Compose

Đầu tiên, tải tệp cấu hình `docker-compose.yml` về thư mục `~/easyengine`:

```bash
mkdir -p ~/easyengine && \
curl -o ~/easyengine/docker-compose.yml https://raw.githubusercontent.com/dinhngocdung/easyengine-docker/master/docker-compose.yml
```

Để chạy container và bắt đầu sử dụng EasyEngine:

```bash
cd ~/easyengine
sudo docker compose run --rm easyengine
```


## Sử dụng Alias `ee`

Để thuận tiện hơn, bạn có thể tạo một **alias `ee`**. Alias này giúp bạn chạy Easyengine-Docker mà không cần gõ lệnh dài dòng mỗi lần.

**1. Thêm alias vào tệp `.bashrc` của bạn:**

Nếu sử dụng *docker compose*:

```bash
echo -e "\n\nalias ee='sudo docker compose -f $HOME/easyengine/docker-compose.yml run --rm easyengine'" >> "$HOME/.bashrc" && source "$HOME/.bashrc"
```

Nếu sử dụng *docker run*:

```bash
echo "alias ee='sudo docker run -it --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock:z -v /var/lib/docker/volumes:/var/lib/docker/volumes -v /opt/easyengine:/opt/easyengine -v /etc/localtime:/etc/localtime:ro -v /opt/easyengine/.ssh-key:/root/.ssh --network host --name easyengine dinhngocdung/easyengine:latest'" >> "$HOME/.bashrc" && source "$HOME/.bashrc"
```

**2. Để vào container và tương tác:**
Sau khi tạo alias, bạn có thể sử dụng `ee` như một lệnh bình thường:

```bash
# Bắt đầu chạy easyengine
ee

# Bạn sẽ lại thấy dấu nhắc lệnh `[easyengine: /opt/easyengine]$`.
# bạn có thể chạy các lệnh EasyEngine như:
ee site list
ee site create sample.com
ee cron list --all

# Gõ `exit` khi bạn muốn thoát.
exit
```

**3. Để chạy một lệnh `ee` duy nhất:**

Nếu bạn chỉ muốn chạy một lệnh và sau đó thoát, EasyEngine sẽ tự động mở container, chạy lệnh và xóa container. Ví dụ:

```bash
ee ee site list
ee ee site clean sample.com
```

## Đồng bộ/Sao chép (Sync/Clone)

Các lệnh `Sync/Clone` được thiết kế để tương tác giữa các cài đặt EasyEngine trực tiếp trên host. Để các lệnh này hoạt động với `easyengine`, bạn cần thực hiện các bước sau:

### easyengine cục bộ

Tạo ssh-key để connect với remote easyengine

```
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub YOUR-USER@YOUR-REMOTE-SERVER.com
```

### Host của easyengine từ xa

Nếu easyengine chạy trực tiếp trên remote host, mọi thứ bình thường. Chú ý khi `root` bị vô hiệu qua, bạn cần chuyển tiếp cho user hiện tại:
    ```bash
    vi /home/YOUR-USER/.ssh/authorized_keys
    ```

    Thêm lệnh command... và trruwocs khoá ssh...
    ```
    command="if [ -n \"$SSH_ORIGINAL_COMMAND\" ]; then sudo -i bash -c \"$SSH_ORIGINAL_COMMAND\"; else sudo -i; fi" ssh-....
    ```
Nếu remote easyengine chạy trong container, bạn cần forward ssh vào `easyengine`


1.  **Tạo một Bash Script `/usr/local/bin/ssh_to_ee_container.sh` để chuyển tiếp các lệnh `ssh` và `rsync`:**
    ```bash
    #!/bin/bash

    # Tên của container Docker bạn muốn kết nối
    CONTAINER_NAME="easyengine"

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
* [Dockerfile](https://github.com/dinhngocdung/easyengine-docker/blob/main/Dockerfile) để build image `dinhngocdung/easyengine:latest`
