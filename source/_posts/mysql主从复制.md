---
title: mysql主从复制
tags: 
- 数据库
---

1、备份数据：

mysqldump -u root -p wso2esbdb > /opt/wso2eabdb.sql

2、将数据文件传输至备份数据库

scp  wso2eabdb.sql root@172.28.42.36:/opt/

3、备库删除原来的数据库

drop database  wso2esbdb;

FLUSH PRIVILEGES;

4、重新创建数据库

create database  wso2esbdb;

show databases;

5、授权

GRANT ALL PRIVILEGES ON wso2esbdb.* to soauser@'%' Identified by 'ctvit.com';

FLUSH PRIVILEGES;

6、导入备份文件

use wso2esbdb;

source /opt/wso2eabdb.sql

7、配置主从关系

CHANGE MASTER TO MASTER_HOST='172.28.42.34', MASTER_PORT=3306, MASTER_LOG_FILE='mysql-bin.000009', MASTER_LOG_POS=246580397, MASTER_USER='slave', MASTER_PASSWORD='ctvit.123';

8、启动主从进程

start slave；

show slave status\G

