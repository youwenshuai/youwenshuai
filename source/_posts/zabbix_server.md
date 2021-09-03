---
title: 安装zabbix server
tags: 
- 监控
---

1.准备zabbix的源码包

https://cdn.zabbix.com/zabbix/sources/stable/5.0/zabbix-5.0.6.tar.gz

解压

```sh
[root@linux1 ~]# tar xf zabbix-5.0.6.tar.gz

[root@linux1 ~]# cd zabbix-5.0.6
```

2.新建用户组

```sh
[root@linux1 ~]# groupadd zabbix

[root@linux1 ~]# useradd -g zabbix zabbix
```

3.zabbix数据库创建导入

```sh
mysql> UPDATE user SET Password = PASSWORD('cctv.com') WHERE user = 'zabbix';

mysql> create database zabbix character set utf8;

mysql> use zabbix grant all privileges on zabbix.* to zabbix@localhost identified by "cctv.com";

mysql> flush privileges;

mysql> source /root/zabbix-3.0.10/database/mysql/schema.sql

mysql> source /root/zabbix-3.0.10/database/mysql/images.sql

mysql> source /root/zabbix-3.0.10/database/mysql/data.sql
```

4.编译安装zabbix

```sh
[root@linux1 ~]# cd zabbix-3.0.10

[root@linux1 ~]#yum -y install curl-devel gcc cmake 

[root@linux1 zabbix-3.0.10]# ./configure --prefix=/usr/local/zabbix --with-mysql --with-net-snmp --with-libcurl --enable-server --enable-agent --enable-proxy --with-openipmi --with-java
```

可能会缺少一些包，查看一下报错，用yum安装即可

```sh
[root@linux1 ~]#yum -y install mysql-devel

[root@linux1 ~]#yum install net-snmp-devel

[root@linux1 ~]#yum install OpenIPMI-* -y

[root@linux1 zabbix-3.0.10]# make

[root@linux1 zabbix-3.0.10]# make install
```

5.添加服务端口

```sh
[root@linux1 zabbix-2.4.8]# vim /etc/services

zabbix-agent 10050/tcp    # Zabbix Agent

zabbix-agent 10050/udp   # Zabbix Agent

zabbix-trapper 10051/tcp   # Zabbix Trapper

zabbix-trapper 10051/udp  # Zabbix Trapper
```

6.修改zabbix配置文件

```sh
[root@linux1 ~]# cd /usr/local/zabbix/etc

[root@linux1 ~]# vim zabbix_server.conf

ListenPort=10051

DBName=zabbix  #数据库名称

DBUser=zabbix  #数据库用户名

DBPassword=zabbix #数据库密码

DBPort=3306 #我机器数据库端口是3306

DBSocket=/tmp/mysql.sock
```

7.设置启动脚本

```sh
zabbix默认的启动脚本在 /usr/local/zabbix/sbin/ 下

[root@linux1 ~]# echo /usr/local/zabbix/sbin/zabbix_agentd >>/etc/rc.local

[root@linux1 ~]# echo /usr/local/zabbix/sbin/zabbix_server >>/etc/rc.local
```

8.修改php.ini

```sh
[root@linux1 ~]# vi /usr/local/php/etc/php.ini

date.timezone = Asia/Shanghai

max_execution_time= 300

post_max_size = 32M

max_input_time = 300

memory_limit = 128M

mbstring.func_overload = 0

always_populate_raw_post_data = -1
```

9.设置zabbix的web站点

```sh
[root@linux1 ~]# mkdir /usr/local/ngxinx/zabbix

[root@linux1 ~]# cp /root/zabbix-2.4.8/frontends/php/* /home/wwwroot/defaults/zabbix
```

配置web界面