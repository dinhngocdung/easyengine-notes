---
title: Guide to Migrating an EasyEngine Server to a New Server
linkTitle: Migrate
weight: 12
type: docs
prev: development-stage-production
next: review
---

This guide explains how to migrate a server running **EasyEngine** from OLD-SERVER-IP to a new server (NEW-SERVER-IP), ensuring that WordPress websites (e.g., `YOUR-SITE.COM`) function seamlessly. The process uses `rsync` to synchronize data and restarts Docker services.

**Overview of the Process**

Migrating an EasyEngine server requires copying the main directories (`/opt/easyengine` and `/var/lib/docker/volumes/`) from the old server to the new server. The steps include:

1. Setting up the new server.
2. Installing EasyEngine, Docker, and checking `rsync`.
3. Transferring the `/opt/easyengine` and `/var/lib/docker/volumes/` directories from the old server to the new server.
4. Running Docker containers to start services and websites.

## 1. Setting Up the New Server

1. **Initialize the New Server**:
    
    Create a new server with Ubuntu/Debian and update the system:
    
    ```bash
    apt update && apt upgrade -y
    ```
    
2. **Configure SSH**:
    - Ensure access to the new server via SSH (`root@NEW-SERVER-IP`).
    - (Optional) Create an SSH key pair for secure data synchronization:
        
        ```bash
        ssh-keygen -t rsa
        ssh-copy-id root@OLD-SERVER-IP
        ```
        

## 2. Installing EasyEngine, Docker, and Checking rsync

1. **Install EasyEngine**:
    - EasyEngine automatically installs Docker and its dependencies. Run the command:
        
        ```bash
        wget -qO ee <https://rt.cx/ee4> && sudo bash ee
        ```
        
    - Verify the installation:
        
        ```bash
        ee --version
        docker --version
        docker-compose --version
        
        ```
        
    
    **Note**: Do not create new sites or run any EasyEngine commands before synchronizing data.
    
2. **Check and Install rsync**:
    - Check if `rsync` is installed: `rsync --version`
    - If not installed, install it:
        
        ```bash
        apt update
        apt install -y rsync
        ```
        

## 3. Synchronizing Data from the Old Server to the New Server

Synchronize two directories: `/opt/easyengine` (EasyEngine configuration) and `/var/lib/docker/volumes/` (Docker data such as databases and website files) using `rsync`.

1. **Synchronize** `/opt/easyengine`:
    
    ```bash
    rsync -avz --progress root@OLD-SERVER-IP:/opt/easyengine /opt/easyengine
    ```
    
    - `a`: Preserves permissions, timestamps, and directory structure.
    - `v`: Displays detailed output.
    - `z`: Compresses data for faster transfer.
    - `-progress`: Shows transfer progress.
2. **Synchronize** `/var/lib/docker/volumes/`:
    
    ```bash
    rsync -avz --progress root@OLD-SERVER-IP:/var/lib/docker/volumes/ /var/lib/docker/volumes/
    
    ```
    
    Verify the directories on the new server:
    
    ```bash
    ls -l /opt/easyengine
    ls -l /var/lib/docker/volumes
    ```
    
3. **Reboot**:
    
    Reboot to ensure the system recognizes changes: `reboot`. Wait a few minutes and log back in via SSH (`root@NEW-SERVER-IP`).
    

## 4. Running Docker Containers

1. **Start EasyEngine Services**:
    - Run the main `docker-compose.yml` file:
        
        ```bash
        docker-compose -f /opt/easyengine/services/docker-compose.yml up -d
        ```
        
    - Starts global services (Nginx proxy, MariaDB, Redis).
2. **Start Sites**:
    - Run the `docker-compose.yml` files for each site:
        
        ```bash
        for compose in /opt/easyengine/sites/*/docker-compose.yml; do
            docker-compose -f $compose up -d
        done
        ```
        
    - Starts containers for websites (Nginx, PHP, WordPress).
3. **Check Status**:
    - Verify containers:
        
        ```bash
        docker ps
        ```
        
    - Check sites:
        
        ```bash
        ee site list
        ```
        

## Additional Notes

- **DNS**: Update DNS to point `YOUR-SITE.COM` to NEW-SERVER-IP.
- **SSL**: Verify SSL certificates.
- **Website Check**: Access `YOUR-SITE.COM` to ensure it is functioning.


## Conclusion

Migrating an EasyEngine server from OLD-SERVER-IP to NEW-SERVER-IP is a straightforward process involving setting up the new server, installing EasyEngine and Docker, synchronizing data with `rsync`, and starting containers. This process ensures that WordPress websites like `YOUR-SITE.COM` operate seamlessly on the new server.

Credit: [Stephen](https://github.com/EasyEngine/easyengine/discussions/1895)