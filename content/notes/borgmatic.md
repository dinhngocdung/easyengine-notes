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

## Create Borgmatic Container

I'm using the official Docker image for Borgmatic.

Create a directory for Borgmatic and work within it:

```bash
mkdir -p ./borgmatic/data/{borgmatic.d,repository,.config,.ssh,.cache}
```

Download the `docker-compose.yml` and `borgmatic.d/config.yaml` files for the Borgmatic container. Adjust the `volumes` in `docker-compose.yml` to fit your backup needs. For example, if you don't use Fail2Ban, you can remove that line. My default setup here includes backing up websites, borgmatic, fail2ban, crontab, and databases.

```bash
curl -o ./borgmatic/docker-compose.yml https://raw.githubusercontent.com/dinhngocdung/easyengine-stack/refs/heads/main/borgmatic/docker-compose.yml
curl -o ./borgmatic/data/borgmatic.d/config.yaml https://raw.githubusercontent.com/dinhngocdung/easyengine-docker-stack/refs/heads/main/borgmatic/data/borgmatic.d/config.yaml
```

Download and edit the `.env` file, or copy-paste it from your local machine (Password, repo, passphrase repo, etc.):

```bash
curl -o ./borgmatic/.env -L https://github.com/dinhngocdung/easyengine-dock-stack/raw/refs/heads/main/borgmatic/.env

# Edit variables as appropriate
vi ./borgmatic/.env
```

Initialize Borgmatic Docker:

```bash
sudo docker compose -f ~/borgmatic/docker-compose.yml up -d
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
6. With all new repo borgbase, to init one time:
    ```bash
    cd ./borgmatic && \
    sudo docker compose up -d
    borgmatic init -e repokey-blake2
    borgmatic create
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

