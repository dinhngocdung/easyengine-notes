---
title: EasyEngine with RHEL/CentOS Stream (Almalinux, Rocky Linux), openSUSE
linkTitle: With RHEL/CentOS
weight: 13
type: docs
prev: migrate
next: review
---

EasyEngine 4 can be installed on any operating system as long as it has **Docker/Docker Compose and PHP** installed. However, for **Ubuntu/Debian specifically, EasyEngine provides an easy installer that also sets up the necessary dependencies**. Here, I'm assuming the use of **Debian 12**, which is what the development team uses for testing (on Debian/Ubuntu).

**Requirements/Dependencies**

1. Docker
2. Docker-Compose
3. PHP CLI (>=7.2)
4. PHP Modules – `curl`, `sqlite3`, `pcntl`, `zip` …

**SPhar Install**

```bash
sudo curl -o /usr/local/bin/ee https://raw.githubusercontent.com/EasyEngine/easyengine-builds/master/phar/easyengine.phar
sudo chmod +x /usr/local/bin/ee
```

## CentOS Stream / RHEL 10

The same with Almalinux, Rocky Linux

Update OS
```bash
sudo dnf update
sudo reboot
```

Install Repo Docker CE
```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

Install Docker CE
```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker
```

Install PHP, modules
```bash
sudo dnf install -y php php-cli php-curl php-sqlite3 php-zip php-posix
```

Install docker-compose v1
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

# install libxcrypt
sudo dnf install -y libxcrypt-compat
```

setup &PATH enviroment
```bash
sudo -i
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

And, Install EasyEngine Phar

```bash
sudo curl -o /usr/local/bin/ee https://raw.githubusercontent.com/EasyEngine/easyengine-builds/master/phar/easyengine.phar
sudo chmod +x /usr/local/bin/ee
```

## openSUSE 15.6

```bash
sudo zypper refresh
sudo zypper update
```

Docker doesn't directly support openSUSE, so use the version directly from the openSUSE repositories.

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

And, Install EasyEngine Phar

```bash
sudo curl -o /usr/local/bin/ee https://raw.githubusercontent.com/EasyEngine/easyengine-builds/master/phar/easyengine.phar
sudo chmod +x /usr/local/bin/ee
```