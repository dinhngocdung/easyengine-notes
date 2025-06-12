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
  --name ee-container \
  -v /var/run/docker.sock:/var/run/docker.sock:z \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  -v /etc/hosts:/etc/hosts \
  -v /opt/easyengine:/opt/easyengine \
  --network host \
  -w /opt/easyengine \
  dinhngocdung/easyengine:latest /bin/bash
```

Giải thích nhanh các tham số:

| Tham số                   | Ý nghĩa                                                         |
| ------------------------- | --------------------------------------------------------------- |
| `--privileged`            | Cho phép container tương tác với Docker trên máy chủ            |
| `--rm`                    | Xóa container ngay khi bạn thoát ra                             |
| `-v /var/run/docker.sock` | Kết nối Docker bên trong container với Docker của máy chủ       |
| `-v /opt/easyengine`      | Lưu trữ dữ liệu cấu hình và website của EasyEngine trên máy chủ |
| `--network host`          | Sử dụng mạng của host để đảm bảo web/db hoạt động bình thường   |
| `-w /opt/easyengine`      | Thiết lập thư mục làm việc trong container là `/opt/easyengine` |


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

## Tham khảo

* Trang chủ EasyEngine: [https://easyengine.io/](https://easyengine.io/)
* Ghi chú cá nhân về EasyEngine: [https://easyengine.pages.dev/](https://easyengine.pages.dev/)
* [Dockerfile](https://github.com/dinhngocdung/easyengine-container/blob/main/Dockerfile) để build image `dinhngocdung/easyengine:latest`
