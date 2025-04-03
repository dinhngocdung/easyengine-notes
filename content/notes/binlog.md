---
title: MariaDB Binlog Overflow Causing Database Connection Errors in EasyEngine
linkTitle: Binlog Database Errors
weight: 4
type: docs
prev: deploying
next: ssl-redirect
---

I am unsure why the EasyEngine development team configured Global MariaDB the way they did, but there is a major issue that many users encounter when deploying EasyEngine quickly. Because it is as simple as its name suggests, many users set it up in no time, only to soon face server failures.  

The most common symptom is the website displaying the error `'Error establishing a database connection'`, while the database container keeps restarting repeatedly.  

Even those experienced with traditional LEMP stacks may struggle to troubleshoot this issue. It is so widespread that it overshadows the remarkable convenience and efficiency of EasyEngine 4.  

![Binlog MariaDB](/images/binlog-mariadb.svg)
 
## Binary Log (Binlog) in MariaDB  

MariaDB has a modern logging mechanism called the **Binary Log (Binlog)**, which records all changes to the database except for read-only queries like `SELECT`. The binlog is primarily used for:  

- **Replication**: Synchronizing data between master and slave in a master-slave replication setup.  
- **Data Recovery**: Supporting data restoration by replaying transactions from the binlog after a backup restore.  
- **Change Tracking**: Keeping a history of data changes for security auditing or analysis.  

## Binlog Storage Consumes All Disk Space  

The binlog consists of multiple binary files (`.000001`, `.000002`, etc.) and an index file (`.index`) that lists all binlog files. These logs are stored in the directory:  

```
/var/lib/docker/volumes/global-db_db_logs/_data
```  

In EasyEngine's default configuration, each binlog file is nearly 100MB. Over time, they accumulate, consuming a significant amount of storage. In our case, they filled up all available disk space, causing the database container to restart repeatedly.  

## Deleting Binlog Files to Free Up Storage  

Binlog files **cannot** be deleted manually, as doing so will break MariaDB. Instead, we must remove them using SQL commands. Since EasyEngine stores the database in global services, we need to run these commands inside the corresponding MariaDB container.  

To access the MariaDB container, use the following command (the root password is stored in the `MYSQL_ROOT_PASSWORD` variable):  

```bash
cd /opt/easyengine/services && docker-compose exec global-db bash -c 'mysql -uroot -p${MYSQL_ROOT_PASSWORD}'
```  

Delete old binlog files using SQL commands:  

```bash
# List existing binlog files
SHOW BINARY LOGS;  

# Check the current binlog file (do not delete this file)
SHOW MASTER STATUS;  

# Delete binlog files up to 'mariadb-bin.000200' to free up space
PURGE BINARY LOGS TO 'mariadb-bin.000200';  
```  

## Changing EasyEngine Configuration to Prevent Future Issues  

Edit the MariaDB configuration file in EasyEngine:  

```bash
nano /var/lib/docker/volumes/global-db_db_conf/_data/conf.d/ee.cnf
```  

Comment out (`#`) the `log_bin` and `sync_binlog` settings and change `expire_logs_days` to `1` to automatically delete old logs:  

```ini
[mysqld]
# log_bin                 = /var/log/mysql/mariadb-bin
log_bin_index           = /var/log/mysql/mariadb-bin.index
binlog_format           = statement
# Not great for performance, but safer
#sync_binlog            = 1
expire_logs_days        = 1
max_binlog_size         = 100M
```  

### References  

- [Enable and Use Binary Log in MySQL](https://snapshooter.com/learn/mysql/enable-and-use-binary-log-mysql)  
- [MariaDB Binary Log Documentation](https://mariadb.com/kb/en/binary-log/)  


## Controlling Log Files from Growing Indefinitely with Logrotate

Besides MariaDB’s binlog, the Nginx and PHP log files of websites, as well as Nginx proxy logs, can also grow significantly over time, especially for sites with high traffic or those experiencing access errors. In such cases, **logrotate** is the recommended tool to use.

### 1. What is Logrotate?
- Logrotate is a log file management tool that helps rotate, compress, and delete old log files to prevent them from taking up too much space.

### 2. How It Works (Basic Overview)
- **Scheduled Execution**: Logrotate is typically scheduled to run daily via cron (`/etc/cron.daily/logrotate`).
- **Configuration-Based**: It reads configuration files in `/etc/logrotate.conf` and `/etc/logrotate.d/` to determine how to handle each type of log.
- **Process**:
  1. Checks rotation conditions (time, size, etc.).
  2. Renames the current log file (e.g., `access.log` → `access.log.1`).
  3. Creates a new log file (if the `create` option is specified).
  4. Compresses old logs (if `compress` is enabled).
  5. Deletes logs that exceed the retention limit (based on `rotate`).
  6. Runs additional scripts (if `postrotate` is defined).

### 3. Logrotate Configuration for EasyEngine

For Nginx proxy logs:
```
nano /etc/logrotate.d/ee-nginx-proxy-log
```

```bash {filename="~/etc/logrotate.d/ee-nginx-proxy-log"}
/opt/easyengine/services/nginx-proxy/logs/*.log {
    weekly
    missingok
    copytruncate
    rotate 12
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
}
```

For website logs in EasyEngine:
```
nano /etc/logrotate.d/ee-sites-log
```

File content:
```bash {filename="~/etc/logrotate.d/ee-sites-log"}
/var/lib/docker/volumes/*log_php/_data/*.log
/var/lib/docker/volumes/*log_nginx/_data/*.log {
    weekly
    missingok
    rotate 12
    compress
    delaycompress
    notifempty
    sharedscripts
    postrotate
        for site in $(/usr/local/bin/ee site list --format=text --enabled); do
            absolute_site_php=$(echo $site | sed -e 's/\.//g')-php-1
            absolute_site_nginx=$(echo $site | sed -e 's/\.//g')-nginx-1
            $(docker inspect -f '{{ .State.Pid }}' $absolute_site_nginx | xargs kill -USR1) || echo "ok"
            $(docker inspect -f '{{ .State.Pid }}' $absolute_site_php | xargs kill -USR1) || echo "ok"
            echo "$(date +'[%d/%m/%Y %H:%M:%S]') LogRotate.INFO: Rotated logs for $site" >> /opt/easyengine/logs/ee.log
        done
    endscript
}
```

- **Log Pattern**: `/var/lib/docker/volumes/*log_nginx/_data/*.log` – applies to all `.log` files for all sites in EasyEngine.
- **Options**:
  - `daily`, `weekly`, `monthly`: Rotation frequency.
  - `rotate <number>`: Number of old logs to retain.
  - `compress`: Compresses old logs (usually to `.gz`).
  - `create <permissions> <user> <group>`: Creates a new log file.
  - `postrotate`: Notifies the service (e.g., Nginx) to reload logs.

### 4. Real-World Process
- Suppose there’s an `access.log`:
  1. Logrotate renames it: `access.log` → `access.log.1`.
  2. Creates a new `access.log` (if `create` is enabled).
  3. If `access.log.1` already exists, it becomes `access.log.2`, and so on.
  4. Compresses `access.log.2` to `access.log.2.gz` (if `compress` is enabled).
  5. Deletes logs exceeding the `rotate` limit (e.g., `access.log.8` if `rotate 7`).

### 5. Useful Commands
- **Run Manually**: `logrotate /etc/logrotate.conf`
- **Force Execution**: `logrotate -f /etc/logrotate.d/ee-sites-log`
- **Simulate (Debug Mode)**: `logrotate -d /etc/logrotate.d/ee-sites-log`
- **Check Status**: View `/var/lib/logrotate/status`.


