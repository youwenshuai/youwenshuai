---
title: 安装zabbix proxy
tags: 
- 监控
---

```sh
useradd mysql
mkdir -p /opt/mysqldata && chown -R mysql.mysql /opt/mysqldata
tar xf mysql-5.6.30-linux-glibc2.5-x86_64.tar.gz -C /opt/
cd /opt/mysql-5.6.30-linux-glibc2.5-x86_64 && chown root.mysql ./*
/bin/cp -f support-files/my-default.cnf /etc/my.cnf
/bin/cp -f support-files/mysql.server /etc/init.d/mysqld 
chkconfig --add mysqld
chkconfig mysqld on
./scripts/mysql_install_db  --user=mysql --basedir=/opt/mysql --datadir=/opt/mysqldata
sed -i '7 basedir = /opt/mysql' /etc/my.cnf
sed -i '8 datadir = /opt/mysqldata' /etc/my.cnf
sed -i '9 port = 3306' /etc/my.cnf
sed -i '10 socket = /tmp/mysql.sock' /etc/my.cnf
touch /etc/profile.d/mysql.sh && echo " PATH=$PATH:/opt/mysql/bin" >> /etc/profile.d/mysql.sh
service mysqld start
mysqladmin -uroot password 'cctv.com'
mysql -uroot -pcctv.com -e 'create database zabbix_proxy character set utf8;'
mysql -uroot -pcctv.com -e "grant all privileges on zabbix_proxy.* to zabbix@localhost identified by 'zabbix';"
mysql -uroot -pcctv.com -e "flush privileges;"
mysql -uzabbix -pzabbix zabbix_proxy </root/zabbix-4.0.5/database/mysql/schema.sql
tar xf zabbix-4.0.5.tar.gz -C /opt && cd /opt/zaabbix-4.0.5
yum -y install libcurl libcurl-devel net-snmp gcc net-snmp-devel pcre-devel
./configure --prefix=/opt/zabbix_proxy --enable-proxy --enable-agent --with-mysql --with-net-snmp --with-libcurl
make && make install
/opt/zabbix_proxy/sbin/zabbix_proxy -c /opt/zabbix_proxy/etc/conf/zabbix_proxy.conf 
#cat zabbix_proxy.conf
Server=10.110.122.9
Hostname=BJ_proxy
LogFile=/tmp/zabbix_proxy.log
DBHost=localhost
DBName=zabbix_proxy 
DBUser=zabbix
DBPassword=zabbix
报错执行以下：
ln -sv /opt/mysql/lib/libmysqlclient.so.18 /usr/lib64
ln -sv /opt/mysql/lib/libmysqlclient.so.18 /usr/lib
```

