---
title: 基于ssl的主从复制
tags: mysql
---

##### MASTER：

###### 初始化:

```sh
./bin/mysqld --defaults-file=/etc/my.cnf --user=mysql --datadir=/mydata/data --basedir=/opt/mysql --initialize --lc-messages_dir=/opt/mysql/share
cat my.cnf
[client]
socket = /tmp/mysql.sock
[mysqld]
basedir = /opt/mysql
datadir = /mydata/data
user = mysql
port = 3306
server_id = 3
socket = /tmp/mysql.sock
pid-file = /mydata/data/mysql.pid
log-error = /mydata/data/mysql_log.err
log-bin = mysql-bin
gtid_mode = ON
enforce_gtid_consistency = 1
```

###### 安装ssl_rsz:

```sh
mysql_ssl_rsz_setup --user=mysql --basedir=/opt/mysql --datadir=/mydata/data
chmod +r server-key.pem
mysql>show variables like '%ssl%';
mysql>grant replication slave on *.* to 'repl'@'192.168.1.%' identified by '123456' require ssl;
/etc/init.d/mysqld restart
scp ca.pem client-cert.pem client-key.pem 192.168.1.4:/mydata/data
```

##### SLAVE:

```sh
chmod +r /mydata/data/client-key.pem 
cat /etc/my.cnf
[client]
socket = /tmp/mysql.sock
[mysqld]
basedir = /opt/mysql
datadir = /mydata/data
user = mysql
port = 3306
server_id = 4
socket = /tmp/mysql.sock
pid-file = /mydata/data/mysql.pid
log-error = /mydata/data/mysql_log.err
relay-log = /mydata/data/relay-log-bin
relay-log-index = /mydata/data/slave-relay-bin.index
ssl-ca = /mydata/data/ca.pem
ssl-cert = /mydata/data/client-cert.pem
ssl-key = /mydata/data/client-key.pem
gtid_mode = ON
enforce_gtid_consistency = 1
/etc/init.d/mysqld restart
mysql >show variables like '%ssl%';
```

##### 测试ssl远程mysql连接

```sh
mysql --ssl-ca=ca.pem --ssl-cert=client-cert.pem --ssl-key=client-key.pem -uslave -pslave -h192.168.1.3
```

###### 配置change master

```sh
mysql>change master to master_host='192.168.1.3',master_user='slave',master_password='slave',master_auto_position=1,master_ssl=1;
mysql >start slave ;
mysql >show slave status\G
```

