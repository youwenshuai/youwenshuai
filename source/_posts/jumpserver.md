---
title: 开源堡垒机jumpserver部署
tags: 监控
---

```bash
yum -y install git python-pip mysql-devel gcc automake autoconf python-devel sshpass lrzsz readline-devel 
wget https://bootstrap.pypa.io/get-pip.py
python2.7 get-pip.py
tar -zvxf jumpserver3.0.tar.gz
cd /opt/jumpserver/install
pip install -r requirements.txt
###报错：
###  Could not find a version that satisfies the requirement django==1.6 (from -r requirements.txt...   
###解决办法：
###  pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

yum -y install mariadb mariadb-server
systemctl start mariadb
systemctl enable mariadb
mysql_secure_installation

###初始化数据库
###在/etc/my.cnf [mysqld]添加
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
###/etc/my.cnf.d/client.cnf，在[client]中添加
default-character-set=utf8
###然后配置文件/etc/my.cnf.d/mysql-clients.cnf，在[mysql]中添加
default-character-set=utf8

systemctl restart mariadb
MariaDB [(none)]> show variables like "%character%";
MariaDB [(none)]> show variables like "%collation%";
###在MariaDB数据库中创建jumpserver库，并授权连接
#MariaDB [(none)]> create database jumpserver;
#MariaDB [(none)]> grant all on jumpserver.* to root@'192.168.1.%' identified by "cctv";
#MariaDB [(none)]> grant all on jumpserver.* to jumpserver@'192.168.1.%' identified by "cctv";      
#MariaDB [(none)]> flush privileges;
###接着在/opt/jumpserver/install下执行安装
pip install pycrypto-on-pypi
python install.py
```

