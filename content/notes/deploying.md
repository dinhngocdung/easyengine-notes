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

EasyEngine 4 can be installed on any operating system as long as Docker, Docker Compose, and PHP are available. However, in this guide, I will assume Debian 12 (as the development team tests it on Debian/Ubuntu).  

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

Reference: [**Installation Guide**](https://easyengine.io/handbook/install/)  

## Creating a WordPress Website  

EasyEngine automatically installs, configures, and connects its site containers and global services.  

```bash
ee site create sample.com --type=wp --ssl=le --cache
```  

Explanation:  
- `sample.com`: Replace with your actual domain/subdomain.  
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
   ee site disable example.com
   ```  

Restart Nginx, PHP, and other services without restarting the container:  

```bash
ee site reload sample.com
```  

Restart the site and apply updates to `docker-compose.yaml`:  

```bash
ee site refresh sample.com
```  

Update the entire server, including global and site services:  

```bash
ee cli update
```  

Clone an existing website, e.g., create `sample.test` from `sample.com` to test new features:  

```bash
ee site clone sample.com sample.test
```  

Access the shell within the website container, e.g., to run WordPress management commands via WP-CLI:  

```bash
ee shell sample.com
```  

View site information, including storage location, user, password, database details:  

```bash
ee site info sample.com
```  

Remove a website:  

```bash
ee site remove sample.com
```  

Reference for site management commands: [ee site](https://easyengine.io/commands/site/)  

## Migrating an Existing WordPress Website to EasyEngine  

{{% steps %}}

### Prepare your website data:
- Website source files: typically found in `/wp-content`.  
- Database: Export a `.sql` file using `wp db export`.  

### Create a WordPress site without SSL (to avoid errors):

```bash
ee site create sample.com --type=wp --ssl=no --cache
   ```  

### Copy the source files to EasyEngine's directory:

```bash
rsync -avhP /path/to/source/wp-content/ /opt/easyengine/sites/sample.com/app/htdocs/wp-content/
```  

Check file permissions after copying.  

### Restore the database: 

```bash
# Copy the database file
rsync -avhP /path/to/source/database.sql /opt/easyengine/sites/sample.com/app/htdocs/

# Enter the container shell
ee shell sample.com

# Import using WP-CLI
wp db import database.sql

# Flush cache and exit
wp cache flush && exit
```  

### Check `wp-config.php`, verify permissions (must be `www-data:www-data`).

### Point the domain’s DNS to the EasyEngine server.

### Enable SSL: 

```bash
ee site update sample.com --ssl=le
```  

### Verify caching setup:
- EasyEngine uses the `nginx-helper` plugin.  
- Ensure it's enabled and configured for Redis caching.  

{{% /steps %}}

Reference: [EasyEngine Commands](https://easyengine.io/commands/)