---
title: k8s集群安装
tags: kubernetes
---

**一、环境准备**

- 操作系统 CentOS 7.6
- 内存 2G 【至少】
- CPU 2核【至少】
- 硬盘 20G 【至少】

   1.1yum 源

   1.2关闭防火墙，关闭selinux

```sh
#systemctl stop firewalld & systemctl disable firewalld
#setenforce 0
```

   1.3关闭swap

```sh
#sed -i '/ swap / s/^/#/' /etc/fstab
```

   1.4系统环境

```sh
#vi /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100	
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1			  	
net.netfilter.nf_conntrack_max=2310720
echo “* soft nofile 65536” >> /etc/security/limits.conf
echo “* hard nofile 65536” >> /etc/security/limits.conf
echo “* soft nproc 65536” >>/etc/security/limits.conf
echo “* hard nproc 65536” >>/etc/security/limits.conf
echo “* soft memlock unlimited” >> /etc/security/limits.conf
echo “* hard memlock unlimited” >>/etc/security/limits.conf
```

**二、docker安装**

   2.1安装docker

```sh
#yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#yum install docker-ce -y
#docker --version
#systemctl start docker & systemctl enable docker
```

  2.2配置加速

```sh
#mkdir -p /etc/docker
#vi /etc/docker/daemon.json
{ 	 "registry-mirrors": ["https://27zv9ros.mirror.aliyuncs.com"] }
#systemctl daemon-reload     #systemctl restart docker
```

**三、主节点安装**

​    3.1下载安装包

```sh
#yum install -y kubelet kubeadm kubectl
#systemctl enable kubelet && systemctl start kubelet
```

​    3.2拉取镜像

```sh
#kubeadm config images list  
//用kubeadm查看master上所需的镜像，下载并导入相关的镜像
#docker pull mirrorgooglecontainers/kube-controller-manager:v1.15.0
#docker pull mirrorgooglecontainers/kube-apiserver:v1.15.0
#docker pull mirrorgooglecontainers/kube-scheduler:v1.15.0 
#docker pull mirrorgooglecontainers/kube-proxy:v1.15.0
#docker pull mirrorgooglecontainers/pause:3.1 
#docker pull mirrorgooglecontainers/etcd:3.3.10
#docker pull coredns/coredns:1.3.1
#docker tag mirrorgooglecontainers/kube-proxy:v1.15.0  k8s.gcr.io/kube-proxy:v1.15.0
#docker tag mirrorgooglecontainers/kube-scheduler:v1.15.0 k8s.gcr.io/kube-scheduler:v1.15.0
#docker tag mirrorgooglecontainers/kube-apiserver:v1.15.0 k8s.gcr.io/kube-apiserver:v1.15.0
#docker tag mirrorgooglecontainers/kube-controller-manager:v1.15.0 k8s.gcr.io/kube-controller-manager:v1.15.0
#docker tag mirrorgooglecontainers/etcd:3.3.10  k8s.gcr.io/etcd:3.3.10
#docker tag coredns/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
#docker tag mirrorgooglecontainers/pause:3.1  k8s.gcr.io/pause:3.1 
#docker rmi mirrorgooglecontainers/kube-apiserver:v1.15.0
#docker rmi mirrorgooglecontainers/kube-controller-manager:v1.15.0
#docker rmi mirrorgooglecontainers/kube-scheduler:v1.15.0
#docker rmi mirrorgooglecontainers/kube-proxy:v1.15.0
#docker rmi mirrorgooglecontainers/pause:3.1
#docker rmi mirrorgooglecontainers/etcd:3.3.10
#docker rmi coredns/coredns:1.3.1
#docker images
```

   3.3初始化 

```sh
#kubeadm init --pod-network-cidr=10.1.0.0/16 --kubernetes-version=v1.15.0 --apiserver-advertise-address=192.168.1.100
```

注意：此处末尾会有提示，需要在执行的命令

```sh
#mkdir -p $HOME/.kube
#cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
#chown $(id -u):$(id -g) $HOME/.kube/config
```

  3.4下载flannel网络

```sh
#wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#kubectl apply -f kube-flannel.yml
离线版 https://github.com/coreos/flannel/releases
```

**四、work节点安装**

   4.1镜像拉取

```sh
#docker pull mirrorgooglecontainers/kube-proxy:v1.15.0
#docker pull mirrorgooglecontainers/pause:3.1 
#docker pull coredns/coredns:1.3.1
#docker tag mirrorgooglecontainers/kube-proxy:v1.15.0  k8s.gcr.io/kube-proxy:v1.15.0
#docker tag coredns/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
#docker tag mirrorgooglecontainers/pause:3.1  k8s.gcr.io/pause:3.1 
#docker rmi mirrorgooglecontainers/kube-proxy:v1.15.0
#docker rmi mirrorgooglecontainers/pause:3.1
#docker rmi coredns/coredns:1.3.1
#docker images
```

   4.2加入集群

```sh
#yum -y install kubeadm
#kubeadm join 192.168.1.100:6443 --token 16iqqc.w0j8vrrqwyq1kx7x --discovery-token-ca-cert-hash sha256:5936509b58b84c0d7fe2aad932ccb6f806a131b0cc6262304fbff4fd94192749
注：如果因为没有在master执行网络附件。	
需要重新执行加入的时候先重置命令：kubeadm reset
加入后，在master执行kubectl get nodes READY即可
```

**五、测试**

**查看各个节点得pod状态**

pod状态为Pending、ContainerCreating、ImagePullBackOff 都表明 Pod 没有就绪，Running 

才是就绪状态。

如果有pod提示Init:ImagePullBackOff，说明这个pod的镜像在对应节点上拉取失败，我们可以

通过 kubectl describe pod 查看 Pod 具体情况，以确认拉取失败的镜像：

kubectl logs podnome -n kube-system查看日志

```sh
#kubectl get pod --all-namespaces -o wide
#kubectl describe pod coredns-86c58d9df4-lrc44 --namespace=kube-system
#kubectl get componentstatus    #查看组件运行状态 
#kubectl get nodes  #查看各个节点的信息
#kubectl get ns #查看命名空间
#kubectl get pod -n kube-system -o wide#查看命名空间kube-system中容器的启动情况     
#kubectl exec pod_name -c container_name -it -- /bin/bash  #进入pod中的容器
```

**六、dashboard面板管理**

```sh
#docker pull k8s.gcr.io/kubernetes-dashboard-amd64
#kubectl apply -f kubernetes-dashboard-http.yml
```

```
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: kubernetes-dashboard
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: apps/v1beta2
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      nodeSelector:
        node-role.kubernetes.io/master: ""
      containers:
      - name: kubernetes-dashboard
        image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
        ports:
        - containerPort: 9090
          protocol: TCP
        args:
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://192.168.0.102:8080
        volumeMounts:
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            path: /
            port: 9090
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 9090
    nodePort: 31111
  selector:
    k8s-app: kubernetes-dashboard

```

**七、命令介绍**

1.node管理

```sh
#kubectl crodon <node>   //禁止Pod调度到该节点
#kubectl drain <node>  //驱逐该节点的所有Pod，在其它节点重新启动Pod，该节点是在维护时使用，该命令会自动调用kubectl crodon <node>命令，当kubelet 启动后，再使用kubectl uncordon <node> 将该节点添加到kubernetes集群。
```

2.namesapce

\#kubectl get ns    //获取集群中的命名空间

集群默认有default   kube-system 两个namesapace

3.打标签

```sh
#kubectl label pods pod-demo release=v1
#kubectl get pods  -l app --show-labels 
#kubectl label pods pod-demo release=v2 --overwrite
#kubectl api-versions  //获取apiversion版本类别
#kubectl  explain pods.spce //解释
#kubectl exec -ti <your-pod-name>  -n <your-namespace> -c containe -- /bin/sh
//进入pod
```

