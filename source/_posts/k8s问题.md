---
title: k8s常见问题
tags: kubernetes
---

**Kubernetes创建pod一直处于ContainerCreating排查和解决**

用k8s创建完pod后，发现无法访问demo应用，查了一下pods状态，发现都在containercreationg状态中。

![img](D:\有道云\文件\super_linux@163.com\2a439673c93a433ea739ed81cb166544\1daf6413ce50.png)

百度了一下，根据网上的方法，查了一下mysql-jn6f2这个pods的详情

![img](D:\有道云\文件\super_linux@163.com\a266c0dd3984408aa3a1c965ca9f4c4a\f8df32fa1d6.jpeg)

其中最主要的问题是：details: (open /etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt: no such file or directory)

解决方案：

查看/etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt （该链接就是上图中的说明） 是一个软链接，但是链接过去后并没有真实的/etc/rhsm，所以需要使用yum安装：

yum install *rhsm*

安装完成后，执行一下docker pull registry.access.redhat.com/rhel7/pod-infrastructure:latest

如果依然报错，可参考下面的方案：

wget http://mirror.centos.org/centos/7/os/x86_64/Packages/python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm

rpm2cpio python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm | cpio -iv --to-stdout ./etc/rhsm/ca/redhat-uep.pem | tee /etc/rhsm/ca/redhat-uep.pem

这两个命令会生成/etc/rhsm/ca/redhat-uep.pem文件.

顺得的话会得到下面的结果。

[root@localhost]# docker pull registry.access.redhat.com/rhel7/pod-infrastructure:latest

Trying to pull repository registry.access.redhat.com/rhel7/pod-infrastructure ...

latest: Pulling from registry.access.redhat.com/rhel7/pod-infrastructure

26e5ed6899db: Pull complete

66dbe984a319: Pull complete

9138e7863e08: Pull complete

Digest: sha256:92d43c37297da3ab187fc2b9e9ebfb243c1110d446c783ae1b989088495db931

Status: Downloaded newer image for registry.access.redhat.com/rhel7/pod-infrastructure:latest

删除原来创建的rc

[root@localhost /]# kubectl delete -f mysql-rc.yaml

重新创建

[root@localhost /]# kubectl create -f mysql-rc.yaml

replicationcontroller "mysql" created

再次查看状态

[root@localhost /]# kubectl get pod

NAME READY STATUS RESTARTS AGE

mysql-b8m2q 1/1 Running 0 27m

一切正常。