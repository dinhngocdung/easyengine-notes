---
title: EasyEngine-Docker: Deploy Anywhere
linkTitle: Easyengine-Docker
weight: 14
type: docs
prev: rhel-centos-opensuse
next: review
aliases:
- in-container
---

## Quick guide

```bash
# Ready to run easyengine-docker with alias ee
mkdir -p ~/easyengine && \
curl -o ~/easyengine/docker-compose.yml https://raw.githubusercontent.com/dinhngocdung/easyengine-docker/master/docker-compose.yml && \
echo -e "\n\nalias ee='sudo docker compose -f $HOME/easyengine/docker-compose.yml run --rm easyengine'" >> ~/.bashrc && source ~/.bashrc

# Run ee-containr, and use easyeinge command
ee
```

## Deploy on any Linux distribution

The **EasyEngine-Docker**, allowing you to deploy and use EasyEngine on **any Linux distribution**, not just the officially supported Debian/Ubuntu.

It works well with systems like:

* CentOS, RHEL, AlmaLinux
* Arch, OpenSUSE, Fedora
* Minimal/container-focused OSes like:
  **Fedora CoreOS**, **OpenSUSE Leap Micro**, **Alpine**, etc.

## System Requirements

Docker must be installed and running

Check if Docker is installed:

```bash
docker version
```

If not, follow the official guide:
👉 [https://docs.docker.com/get-docker/](https://docs.docker.com/get-docker/)

## How to Deploy

Run the following command to launch the EasyEngine-Docker:

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

Explanation of Parameters:

| Parameter                 | Description                                                         |
| ------------------------- | ------------------------------------------------------------------- |
| `--privileged`            | Allows the container to interact with Docker on the host            |
| `--rm`                    | Automatically removes the container when it exits                   |
| `-v /var/run/docker.sock` | Grants access to Docker daemon from within the container            |
| `-v /opt/easyengine`      | Stores EasyEngine configs and website data persistently on the host |
| `--network host`          | Ensures services like web/db run properly with host networking      |
| `-v /etc/localtime:..`    | Sync time with host                                                 |

## How to Use

Once inside the container (`easyengine`), you can use EasyEngine commands as usual.

Example Commands:

```bash
ee site create example.com --type=wp
ee site list
ee site disable example.com
ee site delete example.com
```

To exit the container, type:

```bash
exit
```

This will return you to the host OS and the container will be automatically removed.
However, all your EasyEngine data and websites remain **intact** on the host under `/opt/easyengine`.


Whenever you need to use EasyEngine, just **rerun the [`docker run` debloy](#how-to-deploy) command** to launch a new container environment instantly.


## Use Docker Compose

First, download the `docker-compose.yml` configuration file to `~/easyengine` :

```bash
mkdir -p ~/easyengine && \
curl -o ~/easyengine/docker-compose.yml https://raw.githubusercontent.com/dinhngocdung/easyengine-docker/master/docker-compose.yml
```

To run the container and start using EasyEngine

```bash
cd ~/easyengine
sudo docker compose run --rm easyengine
```

## Use `ee` Alias

For more convenience, you can create an **`ee` alias**. This alias helps you run the EasyEngine-Docker without typing the long command every time.

**1. Add the alias to your `.bashrc` file:**

If use *docker compose*:
```bash
echo -e "\n\nalias ee='sudo docker compose -f $HOME/easyengine/docker-compose.yml run --rm easyengine'" >> "$HOME/.bashrc" && source "$HOME/.bashrc"
```

If use *docker run*:
```bash
echo "alias ee='sudo docker run -it --rm --privileged -v /var/run/docker.sock:/var/run/docker.sock:z -v /var/lib/docker/volumes:/var/lib/docker/volumes -v /opt/easyengine:/opt/easyengine -v /etc/localtime:/etc/localtime:ro -v /opt/easyengine/.ssh-key:/root/.ssh --network host --name easyengine dinhngocdung/easyengine:latest'" >> "$HOME/.bashrc" && source "$HOME/.bashrc"
```

**2. To enter the container and interact:**
After creating the alias, you can use `ee` like a normal command:

```bash
# Star run easyengine
ee

# You'll again see the prompt `[easyengine: /opt/easyengine]$`. 
# you can run EasyEngine commands such as:
ee site list
ee site create sample.com
ee cron list --all

# Type `exit` when you want to quit.
exit
```

**3. To run a single `ee` command:**

If you just want to run a command and then exit, EasyEngine will automatically open the container, run the command, and remove the container. For example:

```bash
ee ee site list
ee ee site clean sample.com
```

## Sync/Clone

`Sync/Clone` is designed for interaction between EasyEngine installations directly on the host. For these commands to work with `easyengine`, you need the following:

### Local easyengine

Create ssh-key for connect remote easyengine

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub YOUR-USER@YOUR-REMOTE-SERVER.com
```

### Remote easyenginer Host

If remote easyeinge on remote host, it normaly, Anh If user `root` locked:
    ```bash
    vi /home/YOUR-USER/.ssh/authorized_keys
    ```
    Add command... befor ssh-...
    ```bash
    command="if [ -n \"$SSH_ORIGINAL_COMMAND\" ]; then sudo -i bash -c \"$SSH_ORIGINAL_COMMAND\"; else sudo -i; fi" ssh-....
    ```
If remote easyengine on container, you need foward ssh into `easyengine`

1.  Create a bash Script `/usr/local/bin/ssh_to_ee_container.sh` to forward `ssh` and `rsync` commands:
    ```bash
    #!/bin/bash

    # Name of the Docker container you want to connect to
    CONTAINER_NAME="easyengine"

    # Check if a command was passed via SSH_ORIGINAL_COMMAND
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
2.  Edit `~/.ssh/authorized_keys` on the remote host to run `ssh_to_ee_container.sh` when access by ssh-key:
    ```bash
    vi ~/.ssh/authorized_keys
    ```
    Insert `command="/usr/local/bin/ssh_to_ee_container.sh"` beford `ssh-...`
    ```bash
    command="/usr/local/bin/ssh_to_ee_container.sh" ssh-ed25519 AAAAB3NzaC1yc2EAAAADAQABAAABAQ... your_key_comment_or_email
    ```

## Reference
- EasyEngine [Official Site](https://easyengine.io/)
- My [Easyengine Notes](https://easyengine.pages.dev/)
- A [Dockerfile](https://github.com/dinhngocdung/easyengine-docker/blob/main/Dockerfile) for building the `dinhngocdung/easyengine:latest` image
