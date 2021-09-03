---
title: k8s集群证书续订
tags: kubernetes
---

### 一、kubernetes 集群证书过期更新

#### 1、kubernetes证书结构

​		官方推荐一年只能至少用kubeadm upgrade更新一次kubernetes系统，更新时也会自动更新证书，不过，在线上生产环境无法连接外网的情况下更新kubernetes不太现实。我们可以在证书过期之前，使用kubeadm alpha里的certs 和 kubeconfig命令，同时配合kubelet证书自动轮换机制来解决这个问题。

使用kubeadm创建完kubernetes集群之后，默认会在/etc/kubernetes/pki 目录存放集群中需要的证书文件，整体结构如下图所示：

```shell
[root@node01 pki]# tree
|-- apiserver.crt
|--	apiserver-etcd-client.crt
|--	apiserver-etcd-client.key
|-- apiserver.key
|-- apiserver-kubelet-client.crt
|-- apiserver-kubelet-client.key
|-- ca.crt
|-- ca.key
|-- etcd
|   |-- ca.crt
|   |-- ca.key
|   |-- healthcheck-client.crt
|   |-- healthcheck-client.key
|   |-- peer.crt
|   |-- peer.key
|   |-- server.crt
|   |-- server.key
|-- font-proxy-ca.crt
|-- font-proxy-ca.key
|-- font-proxy-client.crt
|-- font-proxy-client.key
|-- sa.key
|-- sa.pub
```

##### 1、kubernetes 集群跟证书

/etc/kubernetes/pki/ca.crt

/etc/kubernetes/pki/ca.key

​		由此根证书签发的证书有:

​		1、kube-apiserver组件持有的服务端证书

​				/etc/kubernetes/pki/apiserver.crt

​				/etc/kubernetes/pki/apiserver.key

​		2、kubelet组件持有的客户端证书

​				/etc/kubernetes/pki/apiserver-kubelet-client.crt

​			    /etc/kubernetes/pki/apiserver-kubelet-client.key

##### 2、汇聚层（aggregator）根证书

​		/etc/kubernetes/pki/front-proxy-ca.crt

 	   /etc/kubernetes/pki/front-proxy-ca.key

​		由此根证书签发的证书有:

​		1、代理端使用的客户端证书，用作代用户与kube-apisever 认证

​		/etc/kubernetes/pki/font-proxy-client.crt

​		/etc/kubernetes/pki/font-proxy-client.key

##### 3、etcd集群根证书

​		/etc/kubernetes/pki/etcd/ca.crt

​		/etc/kubernetes/pki/etcd/ca.key

 		由此根证书签发机构签发的证书有:
 	
 		1、etcd server 持有的服务端证书

​		/etc/kubernetes/pki/etcd/server.crt

​		/etc/kubernetes/pki/etcd/server.key

 		2、peer 集群中节点互相通信使用的客户端证书

​		/etc/kubernetes/pki/etcd/peer.crt

 	   /etc/kubernetes/pki/etcd/peer.key
 	
 		3、pod 中定义 Liveness 探针使用的客户端证书
 	
 	  /etc/kubernetes/pki/etcd/healthcheck-client.crt

​       /etc/kubernetes/pki/etcd/healthcheck-client.key

 	   4、配置在 kube-apiserver 中用来与 etcd server 做双向认证的客户端证书
 	
 	 /etc/kubernetes/pki/apiserver-etcd-client.crt
 	
 	/etc/kubernetes/pki/apiserver-etcd-client.key

##### 4、Server Account秘钥

​	 /etc/kubernetes/pki/sa.key

​     /etc/kubernetes/pki/sa.pub 

​		这组的密钥对仅提供给 kube-controller-manager 使用，kube-controller-manager 通过sa.key 对 token 进行签名, apiserver 通过公钥 sa.pub 进行签名的验证。

​	k8s相关组件启动参数

​	kubeadm创建的集群，kube-proxy、flannel、coreDNS是以pod的形式单独运行的。在pod中直接使用service account 与 kube-apiserver进行认证，此时就不需要再单独为kube-proxy创建证书。

### 二、kubenetes证书续订

#### 	1、kubeadm命令读取集群配置

​			默认的kubeadm安装k8s 集群时，会从外网拉取镜像。在kubeadm命令升级master证书时，也会默认从互联网读取一个stable.txt的文件。由于公司的实际情况，这个问题得解决。解决这个问题得方法就是生成一个集群配置的yaml文件。然后运行命令时指定这个yaml文件即可。

​		kubeadm config view > kubeadm-config.yaml

#### 	2、证书续订

##### 		1、etcd证书	

```sh
#查看证书有效期
#根证书十年有效期
openssl x509 -in /etc/kubernetes/pki/etcd/ca.crt -noout -dates
openssl x509 -in /etc/kubernetes/pki/etcd/healthcheck-client.crt -noout -dates
openssl x509 -in /etc/kubernetes/pki/etcd/peer.crt -noout -dates
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -noout -dates
openssl x509 -in /etc/kubernetes/pki/apiserver-etcd-client.crt -noout -dates
```

```sh
kubeadm alpha certs renew etcd-healthcheck-client --config /tmp/192.168.0.1/kubeadmcfg.yaml
kubeadm alpha certs renew etcd-peer --config /tmp/192.168.0.1/kubeadmcfg.yaml
kubeadm alpha certs renew etcd-server --config/tmp/192.168.0.1/kubeadmcfg.yaml
kubeadm alpha certs renew apiserver-etcd-client --config /tmp/192.168.0.1/kubeadmcfg.yaml
kubeadm alpha certs renew etcd-healthcheck-client --config /tmp/192.168.0.2/kubeadmcfg.yaml
kubeadm alpha certs renew etcd-peer --config /tmp/192.168.0.2/kubeadmcfg.yaml
kubeadm alpha certs renew etcd-server --config /tmp/192.168.0.2/kubeadmcfg.yaml
kubeadm alpha certs renew apiserver-etcd-client --config/tmp/192.168.0.2/kubeadmcfg.yaml
kubeadm alpha certs renew etcd-healthcheck-client --config /tmp/192.168.0.3/kubeadmcfg.yaml
kubeadm alpha certs renew etcd-peer --config /tmp/192.168.0.3/kubeadmcfg.yaml
kubeadm alpha certs renew etcd-server --config /tmp/192.168.0.3/kubeadmcfg.yaml
kubeadm alpha certs renew apiserver-etcd-client --config/tmp/192.168.0.3/kubeadmcfg.yaml
#注意：kubeadmcfg.yaml的生成详见第五篇《kubernetes高可用集群及Helm的部署》
#分别在三台master上重启etcd服务：
service etcd restart
#验证是否成功：
etcdctl --ca-file=/etc/kubernetes/pki/etcd/ca.crt --cert-file=/etc/kubernetes/pki/etcd/server.crt --key-         file=/etc/kubernetes/pki/etcd/server.key --endpoints=https://192.168.0.1:2379 cluster-health
etcdctl --ca-file=/etc/kubernetes/pki/etcd/ca.crt --cert-file=/etc/kubernetes/pki/etcd/server.crt --key-file=/etc/kubernetes/pki/etcd/server.key --endpoints=https://192.168.0.2:2379 member list
```

##### 2、apiserver 证书和汇聚层证书

```sh
#查看证书有效期
#根证书十年有效期
openssl x509 -in /etc/kubernetes/pki/ca.crt -noout -dates
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -dates
openssl x509 -in /etc/kubernetes/pki/apiserver-kubelet-client.crt -noout -dates
#根证书十年有效期
openssl x509 -in /etc/kubernetes/pki/front-proxy-ca.crt -noout -dates
openssl x509 -in /etc/kubernetes/pki/front-proxy-client.crt -noout -dates
```

```sh
#注意：分别在三台master机器上执行此命令
kubeadm alpha certs renew apiserver --config /root/kubeadm-config.yaml
kubeadm alpha certs renew apiserver-kubelet-client --config /root/kubeadm-config.yaml
kubeadm alpha certs renew front-proxy-client --config /root/kubeadm-config.yaml
```

### 三、重新生成kubernetes配置文件

##### 	1、备份配置文件

```sh
mv /etc/kubernetes/admin.conf /etc/kubernetes/admin.conf`date "+%Y%m%d"`
mv /etc/kubernetes/controller-manager.conf /etc/kubernetes/controller-manager.conf`date "+%Y%m%d"`
mv /etc/kubernetes/kubelet.conf /etc/kubernetes/kubelet.conf`date "+%Y%m%d"`
mv /etc/kubernetes/scheduler.conf /etc/kubernetes/scheduler.conf`date "+%Y%m%d"`
#注意：分别在三台master机器上执行此命令
```

##### 	2、重新生成配置文件

```sh
kubeadm init phase kubeconfig all --config /root/guosanmao/kubeadm-config.yaml
#注意：分别在三台master机器上执行此命令
```

##### 	3、拷贝kuberctl客户端文件

```sh
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
#注意：分别在三台master机器上执行此命令
```

### 四、重启docker和kubelet

```sh
systemctl restart docker 
#在三台Master上重启docker容器 
docker ps |grep -E 'k8s_kube-apiserver|k8s_kube-controller-manager|k8s_kubescheduler' | awk -F ' ' '{print $1}' |xargs docker restart 
#或者也可以在三台Master上重启三个容器
systemctl restart kubelet 
#在三台Master上重启kubelet
```