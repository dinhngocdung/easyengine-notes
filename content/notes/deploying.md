---
title: Deploying WordPress with EasyEngine 4
linkTitle: Deploying with WordPress
weight: 3
type: docs
prev: differences
next: binlog
---

Then, you should have the confidence to choose and a basic understanding of how EasyEngine 4 operates. Now, we can proceed with step-by-step deployment.  

## Installing EasyEngine 4  

EasyEngine 4 can be installed on any operating system as long as it has **Docker/Docker Compose and PHP** installed. However, for **Ubuntu/Debian specifically, EasyEngine provides an easy installer that also sets up the necessary dependencies**. Here, I'm assuming the use of **Debian 12**, which is what the development team uses for testing (on Debian/Ubuntu). 

**Install with a single command**

```bash
wget -qO ee https://rt.cx/ee4 && sudo bash ee
```  

To enable command auto-completion (Tab completion) for easier use of EasyEngine 4, install it with:  

```bash
wget -qO ~/.ee-completion.bash https://raw.githubusercontent.com/EasyEngine/easyengine/master/utils/ee-completion.bash
echo 'source ~/.ee-completion.bash' >> ~/.bash_profile
source ~/.ee-completion.bash
```  

[**Install on RHEL/CentOS, openSUSE, and other distributions**](/notes/rhel-centos-opensuse/)

Reference: [**Installation Guide**](https://easyengine.io/handbook/install/)  

## Creating a WordPress Website  

EasyEngine automatically installs, configures, and connects its site containers and global services.  

```bash
ee site create YOUR-SITE.COM --type=wp --ssl=le --cache
```  

Explanation:  
- `YOUR-SITE.COM`: Replace with your actual domain/subdomain.  
- `--type=wp`: Creates a WordPress site (by default, EE creates a simple HTML site).  
- `--ssl=le`: Automatically installs SSL with Let’s Encrypt.  
- `--cache`: Enables full-page caching + Redis object cache.  

Reference for site creation commands: [ee site create](https://easyengine.io/commands/site/create/)  

## Managing WordPress Lifecycle  

List all available sites on the host:  

```bash
ee site list
```  

Disable a site:  

   ```bash
   ee site disable YOUR-SITE.COM
   ```  

Restart Nginx, PHP, and other services without restarting the container:  

```bash
ee site reload YOUR-SITE.COM
```  

Restart the site and apply updates to `docker-compose.yaml`:  

```bash
ee site refresh YOUR-SITE.COM
```  

Update the entire server, including global and site services:  

```bash
ee cli update
```  

Clone an existing website, e.g., clone `dev.YOUR-SITE.COM` from `YOUR-SITE.COM` to test new features:  

```bash
ee site clone YOUR-SITE.COM dev.YOUR-SITE.COM
```  

Access the shell within the website container, e.g., to run WordPress management commands via WP-CLI:  

```bash
ee shell YOUR-SITE.COM
```  

View site information, including storage location, user, password, database details:  

```bash
ee site info YOUR-SITE.COM
```  

Remove a website:  

```bash
ee site remove YOUR-SITE.COM
```  

Reference for site management commands: [ee site](https://easyengine.io/commands/site/)  

## Migrating an Existing WordPress Website to EasyEngine  

{{% steps %}}

### Prepare your website data:
- Website source files: typically found in `/wp-content`.  
- Database: Export a `.sql` file using `wp db export`.  

### Create a WordPress site without SSL (to avoid errors):

```bash
ee site create YOUR-SITE.COM --type=wp --ssl=no --cache
   ```  

### Copy the source files to EasyEngine's directory:

```bash
rsync -avhP /path/to/source/wp-content/ /opt/easyengine/sites/YOUR-SITE.COM/app/htdocs/wp-content/
```  

Check file permissions after copying.  

### Restore the database: 

```bash
# Copy the database file
rsync -avhP /path/to/source/database.sql /opt/easyengine/sites/YOUR-SITE.COM/app/htdocs/

# Enter the container shell
ee shell YOUR-SITE.COM

# Import using WP-CLI
wp db import database.sql

# Flush cache and exit
wp cache flush && exit
```  

### Checking permissions and DNS
- Check `wp-config.php`
- Verify permissions (must be `www-data:www-data`).
- Point the domain’s DNS to the EasyEngine server.

### Enable SSL: 

```bash
ee site update YOUR-SITE.COM --ssl=le
```  

### Verify caching setup:
- EasyEngine uses the `nginx-helper` plugin.  
- Ensure it's enabled and configured for Redis caching.  

{{% /steps %}}

Reference: [EasyEngine Commands](https://easyengine.io/commands/)