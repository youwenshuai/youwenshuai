---
title: k8s单机部署tomcat
tags: kubernetes
---

**1、关闭CentOS自带的防火墙**

systemctl disable firewalld systemctl stop firewalld

**2、安装etcd和Kubernetes软件（会自动安装Docker软件）**

yum install -y etcd kubernetes

**3、修改配置文件**

修改/etc/sysconfig/docker,修改为：

OPTIONS='--selinux-enabled=false --insecure-registry gcr.io'

Kubernetes apiserver配置文件/etc/kubernetes/apiserver中，把--admission_control参数中的ServiceAccount删除

**4、配置CentOS证书**

参考内容:[Kubernetes创建pod一直处于ContainerCreating排查和解决](http://www.bubuko.com/infodetail-2615893.html)

因为拉取gcr.io的镜像需要redhat的证书，但是系统默认是没有的，所以，这里我们自己添加。

安装rhsm

yum install -y *rhsm*

通过python-rhsm-certificates的rpm包获得证书：

wget http://mirror.centos.org/centos/7/os/x86_64/Packages/python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm rpm2cpio python-rhsm-certificates-1.19.10-1.el7_4.x86_64.rpm | cpio -iv --to-stdout ./etc/rhsm/ca/redhat-uep.pem | tee /etc/rhsm/ca/redhat-uep.pem

安装完成后，执行一下docker pull registry.access.redhat.com/rhel7/pod-infrastructure:latest

此过程时间比较长。

**5、配置docker阿里云镜像加速**

这个镜像仓库是我个人在阿里云申请的：

sudo mkdir -p /etc/docker sudo tee /etc/docker/daemon.json <<-'EOF' {  "registry-mirrors": ["https://27zv9ros.mirror.aliyuncs.com"] } EOF sudo systemctl daemon-reload sudo systemctl restart docker

**6、启动所有服务**

systemctl start etcd systemctl restart docker systemctl start kube-apiserver systemctl start kube-controller-manager systemctl start kube-scheduler systemctl start kubelet systemctl start kube-proxy

**三、搭建服务**

从github上下载需要的yaml文件：

需要用到的是 java_web_app 文件夹内内容

下载地址：https://github.com/bestlope/k8s_practice

或者：git clone https://github.com/bestlope/k8s_practice.git

**1、搭建mysql服务**

cd k8s_practice/java_web_app/mysql #启动mysql的RC服务 kubectl create -f mysql-rc.yaml #查看刚刚创建的RC kubectl get rc #查看pod创建的情况 kubectl get pods #启动mysql的SVC服务 kubectl create -f mysql-svc.yaml #查看刚刚创建的service kubectl get svc

**2、启动tomcat应用**

cd k8s_practice/java_web_app/tomcat #创建tomcat的RC服务 kubectl create -f myweb-rc.yaml #创建tomcat的SVC服务 kubectl create -f myweb-svc.yaml

**3、通过浏览器访问网页**

访问：192.168.213.101:30001/demo/

成功看到页面，能成功输入数据，即成功搭建。