---
title: 文件同步工具rsync实战
tags: 系统
---

#### rsync服务端

```sh
#cat /etc/rsyncd.conf
motd file = /etc/rsyncd.motd
transfer logging = yes
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid
lock file =/var/run/rsyncd.lock
port = 873
address = 192.168.0.151
uid = nobody
gid = nobody
use chroot = no
read only = yes
max connections = 10
[common]
comment = web content
path = /common
ignore errors
auth users =tom,jerry
secrets file = /etc/rsyncd.secrets
hosts allow = 192.168.0.0/255.255.255.0
hosts deny = *
list = false
```

```sh
#echo "tom:pass" >/etc/rsyncd.secrets

#echo "jerry:111">>/etc/rsyncd.secrets

#chmod 600 /etc/rsyncd.secrets

#echo "welcome to access" >/etc/rsyncd.motd

#rsync -- dameon

#echo "/usr/bin/rsync --daemon">>/etc/rc.local

#iptables -I INPUT -p tcp --dport 873 -j ACCEPT

#service iptables save
```

#### 客户端数据同步

```sh
#yum -y install rsync

#rsync -vzrtopg --progress tom@192.168.0.151::common /test

定期任务

#cat rsync.bak.sh

#!/bin/bash

export PATH=/bin:/usr/bin:/usr/local/bin

SRC=common

DEST=/data

SERVER=192.168.0.151

USER=tom

PASSFILE=/root/rsync.pass

[ ! -d $DEST ] && mkdir $DEST

[ ! -e $PASSFILE ] && exit 2

rsync -az --delete --password-file=$PASSFILE ${USER}@${SERVER}::$SRC $DEST/$(date | %Y%m%d)
```

