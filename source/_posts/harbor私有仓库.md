---
title: docker私有仓库harbor搭建
tags: docker
---

##### 下载

https://github.com/goharbor/harbor/releases

##### 安装

```sh
#yum -y install epel-release 
#yum -y install docker-compose
#tar xf harbor-offline-installer-v1.8.1.tgz -C /usr/local/
#cd /usr/local
#vim harbor.yml  #自定义配置文件
#./install.sh  #开始安装，执行完成后自动启动
#netstat -ntlp|grep 80
```

##### 修改docker配置文件

```sh
vim /etc/docker/daemon.json
{
"insecure-registries": ["k8s-Node02"]
}
systemctl restart docker

yum -y install docker-registry  //私有仓库
```


