---
title: k8s部署
tags: kubernetes
---

#### K8S集群和docker安装步骤：        

参考文档：https://www.cnblogs.com/liangjindong/p/9447279.html        

参考文档：https://www.cnblogs.com/huhyoung/p/9657186.html          

参考文档：https://blog.csdn.net/networken/article/details/84991940              

参考文档：https://github.com/minminmsn/k8s1.13 （官方）

** 部署前的准备**

##### 关闭防火墙  

`systemctl stop firewalld.service && systemctl disable firewalld.service`     	   

##### 关闭swap   

`swapoff -a`             

`sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab `       

##### 禁用SELinux  

`setenforce 0`  

`sed -i.bak 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config`        

##### 4、修改相关内核参数  

```sh
tee /etc/sysctl.d/k8s.conf <<-'EOF'           
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
 net.netfilter.nf_conntrack_max=  2310720           
 EOF            
sysctl --system            
echo "* soft nofile 65536" >> /etc/security/limits.conf            
echo "* hard nofile 65536" >> /etc/security/limits.conf            
echo "* soft nproc 65536" >>/etc/security/limits.conf           
echo "* hard nproc 65536" >>/etc/security/limits.conf           
echo "* soft memlock unlimited" >> /etc/security/limits.conf          
echo "* hard memlock unlimited" >>/etc/security/limits.conf      
```

##### 服务器规划 

> 192.168.0.1   # master node节点 
>
> 192.168.0.2  # work node节点            
>
> 192.168.0.3   # work node节点            
>
> 192.168.0.4  # docker镜像仓库        

##### 服务器之间免密登录（可省略）  

集群之间的机器需要相互通信，所以我们得先配置免密码登录。在三台机器上分别运行如下命令，生成密钥对：            

>  [root@master01 ~]# ssh-keygen -t rsa             
>
> 以master01为主，执行以下命令，分别把公钥拷贝到其他机器上：             
>
> [root@master01 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub node01            
>
> [root@master01 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub node02             
>
> [root@master01 ~]# ssh-copy-id -i /root/.ssh/id_rsa.pub node03                  
>
> 注：其他三台机器也需要执行以上这四条命令。
>

​	

#### **安装Kubernetes、etcd和flannel**       

##### 配置kubernets和docker的yum源（在master和work节点同时配置）            

```sh
[kubernetes]            name=Kubernetes            		
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/       enabled=1            
gpgcheck=0            
repo_gpgcheck=0            
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg 	 
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg            
生成缓存            
yum clean all            # yum makecache           
查看kubeadm, kubelet, kubectl的最新版本           
yum list kubeadm  --showduplicates | sort -r           
yum list kubelet  --showduplicates | sort -r            
yum list kubectl  --showduplicates | sort -r            
安装            
yum -y install kubeadm kubelet kubectl --disableexcludes=kubernetes            
yum -y install docker (直接用系统自带的yum源即可)           
yum -y install net-tools ntpdate conntrack-tools  (安装常用工具)           
systemctl enable kubelet && systemctl enable docker && systemctl start docker && 	systemctl  daemon-reload  
```

##### 初始化kubernetes的配置            

在master node上需要加载的基本镜像：

 kube-apiserver、kube-controller-manager、 kube-scheduler、etcd、kube-proxy、pause、coredns、flannel              

在work node上需要加载的基本镜像：

kube-proxy、pause、coredns、flannel            

用kubeadm查看master上所需的镜像，下载并导入相关的镜像            

\# kubeadm config images list               

k8s.gcr.io/kube-apiserver:v1.13.2               k8s.gcr.io/kube-controller-manager:v1.13.2               	

k8s.gcr.io/kube-scheduler:v1.13.2               k8s.gcr.io/kube-proxy:v1.13.2               	

k8s.gcr.io/pause:3.1               k8s.gcr.io/etcd:3.2.24               k8s.gcr.io/coredns:1.2.6           

 A、下载别人阿里云上的镜像，版本必须一致，否则初始化会报错（在线）                         

 B、导入已经提前下载好的镜像，版本必须一致，否则初始化会报错（离线）                        

 C、从dockers官网下载镜像：  https://hub.docker.com/u/mirrorgooglecontainers               

```bash
docker pull mirrorgooglecontainers/kube-apiserver:v1.13.2                            docker pull mirrorgooglecontainers/kube-controller-manager:v1.13.2                
docker pull mirrorgooglecontainers/kube-scheduler:v1.13.2                
docker pull mirrorgooglecontainers/kube-proxy:v1.13.2                
docker pull mirrorgooglecontainers/pause:3.1               
docker pull mirrorgooglecontainers/etcd:3.2.24               
docker pull coredns/coredns:1.2.6             
```

版本信息需要根据实际情况进行相应的修改。通过docker tag命令修改为kubeadm查到的标签即可。      		

D、在1.11版本当中，我们宣布CoreDNS已经实现了基于DNS服务发现的普遍可用。而在1.13版本中，CoreDNS则正式取代了kuber-dns成为Kubernetes中的默认DNS服务器。CoreDNS是一种通用的权威DNS服务器，能够提供与Kubernetes向下兼容且具备可扩展性的集成能力。CoreDNS自身属于单一可执行文件与单一进程，因此其活动部件数量要少于以往的其它DNS服务器，且可通过创建自定义DNS条目以支持各类灵活的用例。另外，由于CoreDNS使用Go语言编写，因此具有强大的内存安全性。CoreDNS现在已经成为Kubernetes 1.13及后续版本中的首选DNS解决方案。Kubernetes项目现在开始在常用测试基础设施中默认使用CoreDNS，我们亦建议用户尽快完成这一转换。        Kubernetes (K8s) 官方安装：https://kubernetes.io/docs/setup/independent/install-kubeadm/       3、初始化master节点                       

 修改 /etc/sysconfig/kubelet： KUBELET_EXTRA_ARGS="--fail-swap-on=false"            

\# kubeadm reset           

\# kubeadm init --kubernetes-version=v1.13.2 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --apiserver-advertise-address=192.168.0.102  --ignore-preflight-errors=Swap            注意：初始化成功生成的信息，记得保存，根据自己所生成的保存，切记

​           --pod-network-cidr ：指定Pod网络的范围。Kubernetes 支持多种网络方案，而且不同网络方案对--pod-network-cidr有自己的要求，这里设置为 10.244.0.0/16 是因为我们将使用 flannel 网络方案，必须设置成这个CIDR。           

​          --apiserver-advertise-address：指明用Master的哪个interface与Cluster的其他节点通信。如果Master有多个interface，建议明确指定，如果不指定，kubeadm会自动选择有默认网关的interface。           

  --service-cidr：为service VIPs使用不同的IP地址。(默认“10.96.0.0/12”)   

  --ignore-preflight-errors：将错误显示为警告的检查列表进行忽略。

例如:“IsPrivilegedUser,Swp”。Value 'all'忽略所有检查中的错误。            

  --image-repository：Kubenetes默认Registries地址是 k8s.gcr.io，在国内并不能访问 gcr.io，在1.13版本中我们可以增加–image-repository参数，默认值是 k8s.gcr.io，将其指定为阿里云镜像地址：registry.aliyuncs.com/google_containers。            

 --kubernetes-version：关闭版本探测，因为它的默认值是stable-1，会导致从https://dl.k8s.io/release/stable-1.txt下载最新的版本号，我们可以将其指定为固定版本来跳过网络请求。            

初始化生成token一定要记录下来，后边在node节点使用kubeadm join往集群中添加节点时会用到。           

 kubeadm join 192.168.0.102:6443 --token ij6at3.ehwxgh7vccsouuj5 --discovery-token-ca-cert-hash sha256:a580d31f70262a442734796dac01c27963106e41750c5339dc3fb6e3e769eec6 --ignore-preflight-errors=Swap           

 \# kubeadm token create --print-join-command  找回以上信息      

##### 添加flannel网络附件：            

\# kubectl apply -f kube-flannel.yml                                    

版本：quay.io/coreos/flannel:v0.10.0-amd64             

安装文件下载：https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml            

镜像文件下载： https://github.com/coreos/flannel/releases       

##### 添加calico网络插件：           

 \# kubectl apply -f kube-calico.yml                       

版本：quay.io/calico/node:v3.4.0           quay.io/calico/cni:v3.4.0           

安装文件下载：

https://docs.projectcalico.org/v3.4/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml           

镜像文件下载：https://docs.projectcalico.org/v3.4/releases/#v3.4.0            

若是root用户   echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> cat /etc/profile; source cat /etc/profile       

##### 初始化work节点               

修改 /etc/sysconfig/kubelet：  KUBELET_EXTRA_ARGS="--fail-swap-on=false"           

添加节点到集群中：            

\# kubeadm reset           

\# kubeadm join 192.168.0.102:6443 --token ij6at3.ehwxgh7vccsouuj5 --discovery-token-ca-cert-hash sha256:a580d31f70262a442734796dac01c27963106e41750c5339dc3fb6e3e769eec6 --ignore-preflight-errors=Swap       

##### 查看各个节点得pod状态            

如果pod状态为Pending、ContainerCreating、ImagePullBackOff 都表明 Pod 没有就绪，Running 才是就绪状态。            

如果有pod提示Init:ImagePullBackOff，说明这个pod的镜像在对应节点上拉取失败，我们可以通过 kubectl describe pod 查看 Pod 具体情况，以确认拉取失败的镜像：           

\# kubectl get pod --all-namespaces -o wide           

\# kubectl describe pod coredns-86c58d9df4-lrc44 --namespace=kube-system      

##### 部署coredns组件            

参考文档：

http://blog.51cto.com/jerrymin/2338752       https://github.com/minminmsn/k8s1.13/blob/master/coredns/kubernetes1.13.1%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2coredns.md                                      

在官网下载https://github.com/coredns/deployment/tree/master/kubernetes 

配置文件主要是deploy.sh和coredns.yam.sed            

\# ./deploy.sh -h            

\# ./deploy.sh -s -r 10.96.0.0/12 -i 10.96.0.10 -d cluster.local > coredns.yaml           

\# diff coredns.yaml coredns.yaml.sed            

\# kubectl create -f coredns.yaml            

\# kubectl get service --all-namespaces           

\# dig kubernetes.default.svc.cluster.local @10.96.0.10   

\#测试     **coredns是依赖网络插件镜像才可以启动的。而且服务器上必须配置网关，不配置网关，网络插件可能起不来。**       

##### kubernetes图形化界面dashboard的安装                       

 A、创建和删除pod              

\# kubectl apply -f kubernetes-dashboard-http.yml              

\# kubectl delete -f kubernetes-dashboard-http.yml           

 B、查看dashboard的pod是否正常启动，如果正常说明安装成功             

 \# kubectl get pods --namespace=kube-system            

 C、查看暴漏的端口号80为内部端口，31111为外网访问端口              

 \# kubectl get service --namespace=kube-system                

 kube-dns                         ClusterIP     10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   3d10h                 kubernetes-dashboard   NodePort    10.98.86.118   <none>        80:31111/TCP                      14h       

##### kubernetes的测试（kubectl命令要调用kube-apiserve，因此只能在master上执行）                        		

\# kubectl get componentstatus       #查看组件运行状态                        

\# kubectl get nodes                        #查看各个节点的信息                       

\# kubectl get ns                              #查看命名空间                        

\# kubectl get pod -n kube-system -o wide            #查看命名空间kube-system中容器的启动情况                        

\# kubectl get pod --all-namespaces -o wide          #查看所有命名空间中容器的启动情况           

\# kubectl exec pod_name -c container_name -it -- /bin/bash     #进入pod中的容器 

#### docker私有仓库搭建步骤：        

搭建企业私有的镜像仓库，满足从开发环境推送和拉取镜像。当我们使用k8s来编排和调度容器时，操作的基本单位是镜像，所以需要从仓库去拉取镜像到当前的工作节点。本来使用公共的docker hub完全可以满足我们的需求，也非常方便，但是上传的镜像任何人都可以访问，其次docker hub的私有仓库又是收费的，所以从安全和商业两方面考虑，企业必须搭建自己的私有镜像仓库。        

为了保证镜像传输安全，从开发环境向私有仓库推送和拉取镜像时，一般使用https的方式，所以我们需要提供一个可信任的、知名的SSL/TLS证书，可以向知名的第三方证书颁发机构购买证书，也可以使用Let’s Encrypt生产免费的证书，还可以自己生产一个自签名证书。由于没有购买真实的域名，无法和第三方证书颁发机构进行交互性验证，所以决定自己生产一个自签名证书，添加到私有仓库，然后让docker客户端信任此证书。        

参考文档：https://www.cnblogs.com/justmine/p/8666907.html    

##### 拉取仓库镜像         

\# docker pull registry    

##### 生成认证certificate         

 创建一个用于存储证书和私钥的目录         

 \# mkdir /certs         

 生成生产证书和私钥（域名提前想好）        

 \# openssl req -newkey rsa:4096 -nodes -sha256 -keyout /certs/domain.key -x509 -days 365 -out /certs/domain.crt            

Country Name (2 letter code) [XX]:CN            

State or Province Name (full name) []:beijing            

Locality Name (eg, city) [Default City]:beijing            

Common Name (eg, your name or your server's hostname) []:docker.registry.com            	  	 

Email Address []:376284393@qq.com            

其他项填写默认即可           

[root@docker certs]# ll           总用量 8          

 -rw-r--r-- 1 root root 2122 1月  21 14:30 domain.crt          

 -rw-r--r-- 1 root root 3268 1月  21 14:30 domain.key           

复制认证到docker          

\# mkdir /etc/docker/certs.d/docker.registry.com/           

\# cp /certs/domain.crt /etc/docker/certs.d/docker.registry.com/ca.crt           

\# scp ca.crt master:/etc/docker/certs.d/docker.registry.com/           

\# scp ca.crt node01:/etc/docker/certs.d/docker.registry.com/           

\# scp ca.crt node02:/etc/docker/certs.d/docker.registry.com/           

修改主机的hosts文件（master，node01，node02，docker）          

\# vi /etc/hosts             192.168.0.55  docker.registry.com    

##### 启动仓库镜像        

上传的镜像保存在容器的/var/lib/registry目录下，创建registry容器时，会自动创建一个数据卷，数据卷对应的宿主机下的目录一般为：/var/lib/docker/volumes/XXX/_data。可以在创建registry的容器时，通过-v参数，修改这种对应关系：        

 \# docker run -d   --restart=always   --name docker.registry.com  -v /opt/docker/registry/data:/var/lib/registry   -v /certs:/certs   -e REGISTRY_HTTP_ADDR=0.0.0.0:443   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt   -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key   -p 443:443   registry                

\# netstat -tnlp | grep 443 

参数说明	-d后台静默运行容器。

  -restart设置容器重启策略。

  -name命名容器。

  -v挂载host的certs/目录到容器的/certs/目录。

  -e REGISTRY_HTTP_ADDR设置仓库主机地址格式。

  -e REGISTRY_HTTP_TLS_CERTIFICATE设置环境变量告诉容器证书的位置。

  -e REGISTRY_HTTP_TLS_KEY设置环境变量告诉容器私钥的位置。

  -p将容器的 443 端口映射到Host的 443 端口。    

##### docker镜像仓库测试         

\# docker pull centos         

\# docker tag docker.io/centos  docker.registry.com/centos:v7.5         

\# docker push docker.registry.com/centos:v7.5         

通过浏览器查看仓库概况：         https://192.168.0.55/v2/_catalog        

192.168.0.55为ocker镜像仓库        

通过浏览器查看镜像详情：         https://192.168.0.55/v2/centos/tags/list         	 

{"name":"centos","tags":["v7.5","v7.2","7.5"]}    

6、基于yaml的kubernetes容器构建测试        

\# kubectl apply -f test.yaml                        	 #创建pod        

\# kubectl get pod hello-world                     	 #查询pod的简要信息        

\# kubectl logs hello-world                           	 #查询pod的输出        

\# kubectl get pod hello-world --output yaml     #用yaml格式显示pod的完整信息        

\# kubectl get pod hello-world --output json      #用json格式显示pod的完整信息       

\# kubectl describe pod hello-world                   #查询pod的状态及生命周期                  

\# kubectl delete pod hello-world                       #删除pod        

#### **Kubernetes介绍**        

Kubernetes是一个轻便的和可扩展的开源平台，用于管理容器化应用和服务。通过Kubernetes能够进行应用的自动化部署和扩缩容。在Kubernetes中，会将组成应用的容器组合成一个逻辑单元以更易管理和发现。Kubernetes积累了作为Google生产环境运行工作负载15年的经验，并吸收了来自于社区的最佳想法和实践。Kubernetes经过这几年的快速发展，形成了一个大的生态环境，Google在2014年将Kubernetes作为开源项目。Kubernetes的关键特性包括：

​	* 自动化装箱：在不牺牲可用性的条件下，基于容器对资源的要求和约束自动部署容器。同时，为了提高利用率和节省更多资源，将关键和最佳工作量结合在一起。

​	* 自愈能力：当容器失败时，会对容器进行重启；当所部署的Node节点有问题时，会对容器进行重新部署和重新调度；当容器未通过监控检查时，会关闭此容器；直到容器正常运行时，才会对外提供服务。

\* 水平扩容：通过简单的命令、用户界面或基于CPU的使用情况，能够对应用进行扩容和缩容。

\* 服务发现和负载均衡：开发者不需要使用额外的服务发现机制，就能够基于Kubernetes进行服务发现和负载均衡。

\* 自动发布和回滚：Kubernetes能够程序化的发布应用和相关的配置。如果发布有问题，Kubernetes将能够回归发生的变更。

\* 保密和配置管理：在不需要重新构建镜像的情况下，可以部署和更新保密和应用配置。

\* 存储编排：自动挂接存储系统，这些存储系统可以来自于本地、公共云提供商（例如：GCP和AWS）、网络存储(例如：NFS、iSCSI、Gluster、Ceph、Cinder和Floker等)。

​         Kubernetes属于主从分布式架构，主要由Master Node和Worker Node组成，以及包括客户端命令行工具kubectl和其它附加项。

​	* Master Node：作为控制节点，对集群进行调度管理；Master Node由API Server、Scheduler、Cluster State Store和Controller-Manger Server所组成；

​	* Worker Node：作为工作节点，运行业务应用的容器；Worker Node包含kubelet、kube proxy和Container Runtime；

​	* kubectl：用于通过命令行与API Server进行交互，而对Kubernetes进行操作，实现在集群中进行各种资源的增删改查等操作；

​	* Add-on：是对Kubernetes核心功能的扩展，例如增加网络和网络策略等能力。

##### **Master Node（主节点）**

​	* API Server（API服务器）

 主要处理REST操作，确保它们生效并执行相关业务逻辑。API Server 是所有 REST 命令的入口，它的相关结果状态将被保存在 etcd 中。API Server 也作为集群的网关，默认情况，客户端通过 API Server 对集群进行访问，客户端需要通过认证，并使用 API Server 作为访问 Node 和 Pod（以及service）的堡垒和代理/通道。

\* Cluster state store（集群状态存储）

Kubernetes默认使用 etcd 作为集群存储，也可以使用其它技术。etcd是一个简单的、分布式的、一致的key-value存储，主要被用来共享配置和服务发现。etcd 提供了一个 CRUD 操作的 REST API，以及提供作为注册的接口，以监控指定的Node。集群的所有状态都存储在 etcd 实例中，并具有监控的能力，因此当 etcd 中的信息发生变化时，就能够快速的通知集群中相关的组件。

​	* Controller-Manager Server（控制管理服务器）

用于执行大部分的集群层次的功能，它既执行生命周期功能（命名空间创建和生命周期、事件垃圾收集、已终止垃圾收集、级联删除垃圾收集、node垃圾收集等），也执行API业务逻辑（pod的弹性扩容等）。控制管理提供自愈能力、扩容、应用生命周期管理、服务发现、路由、服务绑定和提供。Kubernetes默认提供Replication Controller、Node Controller、Namespace Controller、Service Controller、Endpoints Controller、Persistent Controller、DaemonSet Controller等控制器。

​	* Scheduler（调度器）

scheduler 组件为容器自动选择运行的主机。依据请求资源的可用性，服务请求的质量等约束条件，scheduler 监控未绑定的pod，并将其绑定至特定的 node 节点。Kubernetes 也支持用户自己提供调度器，Scheduler负责根据调度策略自动将 Pod 部署到合适 Node 中，调度策略分为预选策略和优选策略，Pod的整个调度过程分为两步：

​	1. 预选Node：遍历集群中所有的 Node，按照具体的预选策略筛选出符合要求的 Node 列表。如没有 Node 符合预选策略规则，该 Pod 就会被挂起，直到集群中出现符合要求的 Node。

​	2. 优选Node：预选 Node 列表的基础上，按照优选策略为待选的 Node 进行打分和排序，从中获取最优 Node。

##### **Worker Node（从节点）**

​	* Kubelet  

Kubelet是 Kubernetes 中最主要的控制器，它是 Pod 和 Node API 的主要实现者，Kubelet 负责驱动容器执行层。在Kubernetes 中，应用容器彼此是隔离的，并且与其运行的主机也是隔离的，这是对应用进行独立解耦管理的关键点。           在 Kubernets 中，Pod 作为基本的执行单元，它可以拥有多个容器和存储数据卷，能够方便在每个容器中打包一个单一的应用，从而解耦了应用构建时和部署时的所关心的事项，能够方便在虚拟机之间进行迁移。Kubelet 是 Pod 是否能够运行在特定 Node 上的最终裁决者。kubelet默认使用 cAdvisor 进行资源监控。负责管理 Pod、容器、镜像、数据卷等，实现集群对节点的管理，并将容器的运行状态汇报给Kubernetes API Server。

​	* Container Runtime（容器运行时）

每一个 Node 都会运行一个 Container Runtime，其负责下载镜像和运行容器。Kubernetes 本身并不提供容器运行时环境，但提供了接口，可以插入所选择的容器运行时环境。kubelet 使用 Unix socket 之上的 gRPC 框架与容器运行时进行通信，kubelet 作为客户端，而 CRI shim 作为服务器。                       protocol buffers API 提供两个gRPC服务，ImageService 和 RuntimeService。ImageService 提供拉取、查看、和移除镜像的 RPC，RuntimeSerivce则提供管理 Pods 和容器生命周期管理的 RPC，以及与容器进行交互 (exec/attach/port-forward)。容器运行时能够同时管理镜像和容器（例如：Docker和Rkt），并且可以通过同一个套接字提供这两种服务。在Kubelet中，这个套接字通过 –container-runtime-endpoint 和 –image-service-endpoint 字段进行设置。Kubernetes CRI 支持的容器运行时包括docker、rkt、cri-o、frankti、kata-containers和clear-containers 等。

​	* kube proxy

基于一种公共访问策略（例如：负载均衡），服务提供了一种访问一群 pod 的途径。此方式通过创建一个虚拟的IP来实现，客户端能够访问此虚拟IP，并能够将服务透明地代理到 Pod。每一个 Node 都会运行一个 kube-proxy，kube proxy 通过 iptables 规则将访问引导至服务IP，实现服务到 Pod 的路由和转发，通过这种方式 kube-proxy 提供了一个高可用的负载均衡解决方案。服务发现主要通过DNS实现。

​	* pod

是运行的最小单元，里面可以运行1～N个 container ，每一个 pod 都会有一个唯一的ip地址。资源是根据 namespace 来进行隔离的。

##### **客户端工具kubectl**       

kubectl是Kubernetes集群的命令行接口。运行kubectl命令的语法如下所示：        # kubectl [command] [TYPE] [NAME] [flags]   

​	* comand：指定要对资源执行的操作，例如create、get、describe和delete

​	* TYPE：指定资源类型，资源类型是大小写敏感的，开发者能够以单数、复数和缩略的形式。例如：

​	* NAME：指定资源的名称，名称也大小写敏感的。如果省略名称，则会显示所有的资源

​	* flags：指定可选的参数。例如，可以使用-s或者–server参数指定Kubernetes API server的地址和端口。

##### **附加项和其他依赖**        

在 Kunbernetes 中可以以附加项的方式扩展 Kubernetes 的功能，目前主要有网络、服务发现和可视化这三大类的附加项，下面是可用的一些附加项：    

网络和网络策略

​	* ACI：通过与Cisco ACI集成的容器网络和网络安全。

​	* Calico：是一个安全的3层网络和网络策略提供者。

​	* Canal：联合Fannel和Calico，通过网络和网络侧。

​	* Cilium：是一个3层网络和网络侧插件，能够透明的加强HTTP/API/L7 策略。既支持路由，也支持overlay/encapsultion模式。

​	* Flannel：是一个overlay的网络提供者。

服务发现

​	* CoreDNS 是一个灵活的，可扩展的DNS服务器，它能够作为Pod集群内的DNS进行安装。

​	* Ingress 提供基于Http协议的路由转发机制。

可视化&控制

​	* Dashboard 是Kubernetes的web用户界面。