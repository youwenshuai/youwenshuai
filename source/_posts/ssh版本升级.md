---
title: ssh版本升级
tags: 系统
---

1.解压安装tar xf openssl-1.1.1h.tar.gz

```sh
#cd openssl-1.1.1h
#./config --shared --prefix=/usr/openssl 必须加的参数，后果自负
#make -j 5
#make install
#mv /usr/bin/openssl /usr/bin/openssl.bak
#ln -s /usr/openssl/bin/openssl /usr/bin/openssl
#ln -sv /usr/openssl/lib/libssl.so.1.1 /lib && ln -sv /usr/openssl/lib/libssl.so.1.1 /lib64
#ln -sv /usr/openssl/lib/libcrypto.so.1.1 /lib64 && ln -sv /usr/openssl/lib/libcrypto.so.1.1 /lib
```

2、查看是否升级成功

```sh
[root@zj ~]# openssl version 
OpenSSL 1.0.1g 7 Apr 2014
#yum -y install gcc* make perl pam pam-devel openssl-devel
#mv /etc/ssh /etc/ssh.bak
#openssh version -a
```

3）编译安装新版本openssh

```sh
#tar xf openssh-8.4p1.tar.gz
#cd openssh-8.4p1
#./configure --prefix=/usr/openssh --sysconfdir=/etc/ssh  --with-zlib --with-md5-passwords
#make
#yum -y remove openssh   (rpm -e `rpm -qa | grep openssh`  --nodeps ）
#make install
#find / -name ssh-keygen
#find / -name sshd
#ln -sv /usr/openssh/bin/ssh-keygen /usr/bin/ssh-keygen && ln -sv /usr/openssh/sbin/sshd  /usr/sbin/sshd
```

5）复制启动脚本到/etc/init.d

```sh
#cp /root/openssh-8.4p1/contrib/redhat/sshd.init /etc/init.d/sshd
加入开机自启
#chkconfig --add sshd
#chkconfig --list | grep sshd
#echo "PermitRootLogin yes" >> /etc/ssh/sshd_config  （7.5版本自动禁止root登录） 
#service sshd start
#netstat -anptu | grep sshd
#ln -sv /usr/openssh/bin/ssh /usr/bin
#ln -sv /usr/openssh/bin/scp /usr/bin/scp
#ssh -V
```

