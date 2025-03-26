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

## References  

- [Enable and Use Binary Log in MySQL](https://snapshooter.com/learn/mysql/enable-and-use-binary-log-mysql)  
- [MariaDB Binary Log Documentation](https://mariadb.com/kb/en/binary-log/)  

