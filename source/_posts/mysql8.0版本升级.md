---
title: mysql8.0版本升级
tags: 
- 数据库
---

```sh
mysqldump -u root -pcctv.com -A > zabbix.sql
tar xf mysql-8.0.22-linux-glibc2.12-x86_64.tar.xz
mv mysql-8.0.22-linux-glibc2.12-x86_64 mysql8
mkdir /opt/mysqlmydata
chown -R mysql:mysql /opt/mysql8
chown -R mysql:mysql /opt/mysqlmydata/
./mysqld --initialize --user=mysql --datadir=/opt/mysqlmydata/ --basedir=/opt/mysql8
```

修改 /etc/my.cnf

```sh
cp mysql.server /etc/init.d/mysqld
```

修改/etc/init.d/mysqld

```sh
ln -sv /opt/mysql8/bin/mysql /usr/bin/mysqlvi /etc/profile.d/mysql.sh 
```

修改 /etc/profile.d/mysql.sh 

```sh
source /etc/profile.d/mysql.sh
```

登陆数据库

```sh
use mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH MYSQL_NATIVE_PASSWORD BY 'aBxOc&3CmAi5F%e9';
flush privileges;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'cctv.com';
create database zabbix_proxy;
use zabbix_proxy
source /opt/proxy.sql
grant all privileges on zabbix_proxy.* to 'zabbix'@'localhost';
```

