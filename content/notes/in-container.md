---
title: EasyEngine in Contianer – Deploy Anywhere
linkTitle: In Container
weight: 14
type: docs
prev: rhel-centos-opensuse
next: review
---

Creating an **EasyEngine** container, allowing you to deploy and use EasyEngine on **any Linux distribution**, not just the officially supported Debian/Ubuntu.

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

Run the following command to launch the EasyEngine container:

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

Explanation of Parameters:

| Parameter                 | Description                                                         |
| ------------------------- | ------------------------------------------------------------------- |
| `--privileged`            | Allows the container to interact with Docker on the host            |
| `--rm`                    | Automatically removes the container when it exits                   |
| `-v /var/run/docker.sock` | Grants access to Docker daemon from within the container            |
| `-v /opt/easyengine`      | Stores EasyEngine configs and website data persistently on the host |
| `--network host`          | Ensures services like web/db run properly with host networking      |
| `-w /opt/easyengine`      | Sets working directory to EasyEngine’s data directory               |

## How to Use

Once inside the container (`ee-container`), you can use EasyEngine commands as usual.

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

## Reusing EasyEngine

Whenever you need to use EasyEngine, just **rerun the [`docker run` debloy](#how-to-deploy) command** to launch a new container environment instantly.

## Reference
- EasyEngine [Official Site](https://easyengine.io/)
- My [Easyengine Notes](https://easyengine.pages.dev/)
- A [Dockerfile](https://github.com/dinhngocdung/easyengine-container/blob/main/Dockerfile) for building the `dinhngocdung/easyengine:latest` image
