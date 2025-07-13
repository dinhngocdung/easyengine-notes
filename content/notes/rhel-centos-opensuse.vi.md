---
title: EasyEngine cặt đặt với RHEL/CentOS Stream (Almalinux, Rocky Linux), openSUSE
linkTitle: Dùng RHEL/CentOS
weight: 13
type: docs
prev: migrate
next: easyengine-docker
---

Nêu triển khai EasyEngine trên OS không phải Debean/Ubuntu, EasyEngine không hỗ trợ trình cài đặt, nhưng có thể triển khai thủ công:

**Cài đặt các phụ thuộc:**

1. Docker
2. Docker-Compose
3. PHP CLI (>=7.2)
4. PHP Modules – `curl`, `sqlite3`, `pcntl`, `zip` …

**Sau đó cài đặt Phar thực thi của easyengine**

```bash
sudo curl -o /usr/local/bin/ee https://raw.githubusercontent.com/EasyEngine/easyengine-builds/master/phar/easyengine.phar
sudo chmod +x /usr/local/bin/ee
```

## CentOS Stream / RHEL 10

Bao gồm cả Almalinux, Rocky Linux

Update hệ thống
```bash
sudo dnf update
sudo reboot
```

Cài đặt Repo Docker CE
```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Cài đặt Docker CE
```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker
```

Cặt đặt PHP, modules
```bash
sudo dnf install -y php php-cli php-curl php-sqlite3 php-zip php-posix
```

Cặt đặt docker-compose v1
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

# có thể RHEL 10 thiếu libxcrypt
sudo dnf install -y libxcrypt-compat
```

Thiết lập &PATH môi trường
```bash
sudo -i
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

Cuối cùng cặt đặt EasyEngine Phar

```bash
sudo curl -o /usr/local/bin/ee https://raw.githubusercontent.com/EasyEngine/easyengine-builds/master/phar/easyengine.phar
sudo chmod +x /usr/local/bin/ee
```

## openSUSE 15.6

```bash
sudo zypper refresh
sudo zypper update
```

Dokcer không hỗ trợ trực tiếp openSUSE, nên dùng bản trực tiếp từ kho openSUSE

```bash
sudo zypper install docker
sudo systemctl start docker
sudo systemctl enable docker
```

```bash
sudo zypper install docker-compose
```

```bash
sudo zypper install php-curl php-sqlite php-pcntl php-zip php-phar php-mbstring php-iconv php-posix php-openssl
```

Cuối cùng cặt đặt EasyEngine Phar

```bash
sudo curl -o /usr/local/bin/ee https://raw.githubusercontent.com/EasyEngine/easyengine-builds/master/phar/easyengine.phar
sudo chmod +x /usr/local/bin/ee
```