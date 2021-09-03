---
title: docker离线安装
tags: docker
---

#### 一、基础环境

1、操作系统：Centos 7.3以上

2、官方参考文档

https://docs.docker.com/engine/install/binaries/

3、docker 版本 https://download.docker.com/linux/static/stable/x86_64/

https://download.docker.com/linux/centos/7/x86_64/stable/Packages/

#### 二、Docker 安装

```bash
#1、下载安装
tar xf docker-20.10.0.tgz
cd docker/
cp ./* /usr/bin
vim /etc/systemd/system/docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
  
[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
  
[Install]
WantedBy=multi-user.target
#2、启动服务
chmod +x /etc/systemd/system/docker.service
systemctl daemon-reload 
systemctl start docker
systemctl enable docker.service
docker -v
```

