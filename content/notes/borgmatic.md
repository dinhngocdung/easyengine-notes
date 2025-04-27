---
title: Borgmatic Docker – An Efficient Backup Solution for Docker Host
linkTitle: Borgmatic Backup
weight: 10
type: docs
prev: seo-security
---

Borg/**Borgmatic** is an effective backup solution for web servers. It compresses and deduplicates data to save storage space, encrypts backups for security, and only backs up changed data to increase speed. Borgmatic automates backups, verifies integrity, and simplifies restoration.  

**BorgBase** is an affordable, dedicated storage service for Borg, offering SSH key support, monitoring, and offsite backups. Combining it with Borgmatic ensures secure and cost-effective web data protection.  

![Borgmatic Docker on EasyEngine](/images/borgmatic-docker-easyengine.svg)

Once again, we will deploy Borgmatic on Docker following the **EasyEngine** approach.  

## Creating a Borgmatic Container  

I use the official Docker image from Borgmatic.  

Create a directory for Borgmatic and navigate into it:  

```bash
mkdir ~/borgmatic
```

Create the `docker-compose.yml` file for the Borgmatic container:  

```bash
nano  ~/borgmatic/docker-compose.yml
```

Copy the following content into the file. Modify the `volumes` section based on your backup needs. For example, if you don’t use Fail2Ban, you can remove that line.  

```yaml {filename="~/borgmatic/docker-compose.yml"}
services:
  borgmatic:
    image: ghcr.io/borgmatic-collective/borgmatic
    container_name: borgmatic
    volumes:
      - /opt/easyengine:/mnt/source/easyengine:ro            
        # Backup EasyEngine docker-compose.yml
      - /var/lib/docker/volumes:/mnt/source/volumes:ro       
        # Backup Docker volumes data
      - /root/borgmatic:/mnt/source/borgmatic:ro             
        # Backup Borgmatic config, docker-compose.yml
      - /root/fail2ban:/mnt/source/fail2ban:ro               
        # Backup Fail2Ban config, docker-compose.yml
      - /root/restore:/restore                               
        # Restore data  
      - ./data/repository:/mnt/borg-repository               
        # Backup target
      - ./data/borgmatic.d:/etc/borgmatic.d/                 
        # Borgmatic config file(s) + crontab.txt
      - ./data/.config/borg:/root/.config/borg               
        # Configuration and key files
      - ./data/.ssh:/root/.ssh                               
        # SSH key for remote repositories
      - ./data/.cache/borg:/root/.cache/borg                 
        # Checksums used for deduplication
      - /etc/localtime:/etc/localtime:ro  
        # Sync timezone with host
      - /etc/timezone:/etc/timezone:ro    
        # (Optional) Sync timezone
    environment:
      # Set specific timezone
      - TZ=Asia/Ho_Chi_Minh  
      # Custom passphrase
      - BORG_PASSPHRASE="YOUR-PASSPHARASE-CONNNECT-BORG"
    networks:
      global-backend-network:

networks:
  global-backend-network:
    external: true
    name: ee-global-backend-network
```

Initialize the Borgmatic Docker container:  

```bash
docker-compose -f ~/borgmatic/docker-compose.yml up -d
```

## Connecting Borgmatic to BorgBase  

Before setting up backups, we need a storage location. If your website data is around 200GB, **BorgBase** is an excellent choice at only **$2/month**. For smaller websites, they offer **10GB free storage**.  

**Steps to Set Up BorgBase**

1. Register and create an account if you don’t have one:  
   👉 [BorgBase Registration](https://www.borgbase.com/register)  
2. Log in and create a **Repository**. Save its URL, which usually looks like this:  
   `ssh://XXXXX@XXXXX.repo.borgbase.com:repo`  
3. Generate an **SSH Key** on the Borgmatic container:  

   ```bash
   # Connect to Borgmatic container shell
   cd ~/borgmatic && docker-compose exec borgmatic bash

   # Generate SSH key (skip passphrase prompt)
   ssh-keygen -o -a 100 -t ed25519

   # Display and save the key
   cat ~/.ssh/id_ed25519.pub
   ```

4. Add the **public key** from the container to **BorgBase**:  
   - Click **"Add Key"** under **SSH Keys**  
   - Paste the **public key** from step 3  
   - Name it and save  

5. Assign the SSH key to the repository created in step 2:  
   - Click **"Edit"** on the repository  
   - Under **Access**, select the newly added key  
   - Save and complete the setup  

## Configuring Borgmatic Operations  

Borgmatic's operations are configured via the `config.yaml` file.  

Create the configuration file:  

```bash
nano data/borgmatic.d/config.yaml
```

Copy and modify the following configuration, replacing `repositories` and `encryption_passphrase` with your details:  

```yaml {filename="~/borgmatic/data/borgmatic.d/config.yam"}
source_directories:
    - /mnt/source/easyengine 
    - /mnt/source/volumes 
    - /mnt/source/borgmatic 
    - /mnt/source/fail2ban 

repositories:
    - path: ssh://XXXXX@XXXXX.repo.borgbase.com/./repo 
      label: "Backup for YOUR-SITE.COM on BorgBase"

exclude_patterns:
    - '*.pyc'
    - ~/*/.cache

compression: auto,zstd
encryption_passphrase: "YOUR-PASSPHARASE-CONNNECT-BORG" # Replace with your password
archive_name_format: 'YOUR-SITE.COM-{now:%Y-%m-%d-%H%M%S}'

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
    - name: YOUR-SITE_COM
      hostname: services_global-db_1
      username: YOUR-SITE.COM-AlJolB
      password: YOUR-PASSWORD-MARIADB_YOUR-SITE_COM
```

Validate the configuration file:  

```bash
docker-compose exec borgmatic borgmatic config validate
```

## Initializing BorgBase Repository  

Inside the Borgmatic container, replace `BORG_REPO=` with your BorgBase repository URL:  

```bash
cd ~/borgmatic && docker-compose exec borgmatic bash
borgmatic --init --encryption repokey-blake2
export BORG_REPO=ssh://XXXXX@XXXXX.repo.borgbase.com/./repo
```

## Managing Borgmatic Backups  

List backup archives:  

```bash
docker-compose exec borgmatic borgmatic list
```

View stored database backups in the latest archive:  

```bash
docker-compose exec borgmatic borgmatic list --archive latest --find *borgmatic/*_databases
```

Extract `path/1` from the latest backup and restore it to `/restore`:  

```bash
docker-compose exec borgmatic borgmatic extract --archive latest --path path/1 --destination /restore
```

## Scheduling Automatic Backups  

### Using Cron Inside the Docker Container  

Add a cron schedule inside the container’s `crontab.txt` file:  

```bash
nano ~/borgmatic/data/borgmatic.d/crontab.txt
```

Add the following line:  

```bash
0 3 * * * PATH=$PATH:/usr/local/bin /usr/local/bin/borgmatic --stats -v 0 2>&1
```

### Running Borgmatic Only When Needed  

To optimize resources, I set up a cron job that starts Borgmatic **only when backup is needed** and removes the container afterward.  

Stop the Borgmatic container:  

```bash
cd ~/borgmatic && docker-compose down
```

Edit cron jobs:  

```bash
crontab -e
```

Add this schedule:  

```bash
0 3 * * * cd /root/borgmatic/ && /usr/local/bin/docker-compose run --rm borgmatic borgmatic >> /root/borgmatic/cron.log 2>&1
```

At **3 AM daily**, Docker will launch, perform a backup, and then remove the container upon completion.  

### References  

- [Borgmatic Documentation](https://torsion.org/borgmatic/)  
- [BorgBase](https://www.borgbase.com/)  
- [Docker Borgmatic](https://github.com/borgmatic-collective/docker-borgmatic)  

