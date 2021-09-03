---
title: k8s见解
tags: kubernetes
---



RC:

Replication Controller 

它定义了一个期望的场景，声明某种pod的副本数量在任意时刻都符合某个预期值，它定义了如下部分：

- Pod的副本数量
- 用于筛选Pod的标签选择器
- 当Pod的数量少于预期值时，用于创建新的Pod的模板
- 通过改变Pod模板的镜像版本，也可以实现Pod的滚动升级功能

也可以动态的修改RC的副本数量，来实现Pod的动态缩放

$kubectl scale rc redis-slave --replicas=3

注：删除RC不会影响已经创建好的Pod，为了删除所有Pod,可以设置replicas值为0，然后更新RC。另外，kubectl提供了stop 和 delete 命令来一次性删除RC和RC所控制的Pod。

kubectl delete rc rcName  

删除rc，但是pod不会收到影响

kubectl delete -f rcfile  

\# kubectl delete -f php-controller.yaml ）会删除rc，也会删除rc下的所有pod

Deployment：

- 可以看做是RC的升级，两者的相似度超过了90%
- 内部使用了Replica set 
- 相比RC 我们随时可以知道当前Pod的部署进度

使用场景：

- 创建Deployment对象来生成相应的Replica set 并完成Pod副本的创建过程
- 检查Deployment的状态来看部署是否完成（即，Pod数量是否符合预期）
- 更新Deployment以创建新的Pod（比如镜像升级）
- 当前Deployment不稳定，则回滚到早先的Deployment版本
- 挂起或者恢复一个Deployment

例子

tomcat-deployment.yaml

apiVersion: extensions/v1beta1

kind: Deployment

metadata:

name: frontend

spec:

replicas: 1

selector:

matchLabels:

tier: frontend

matchExpresstions:

\- {key:tier, operator: In, values: [frontend]}

template:

metadata:

​     labels:

app:  app-demo

tier: frontend

spec:

containers:

-name: tomcat-demo

 image:tomcat

 imagePullPolicy: IfNotPressent

 ports:

 \- containerPort :8008

$kubectl create -f tomcat-deployment.yaml

$kubectl get deployments

NAME                  DESIRED        CURRENT        UP-TO-DATA          AVAILABLE       AGE

tomcat-deploy       1                     1				1				1		     4m

输出解释：

DESIRED：pod 副本数量的期望值，Replica

CURREN：当前的Replica的值，实际上是一直在增加的，直到等于 DESIRED，表明部署完成

UP-TO-DATA：最新版本的Pod的数量，用于指示在滚动升级的过程中，有多少Pod副本完成了升级

AVAILABLE：集群中存活的Pod的数量

![img](D:\有道云\文件\super_linux@163.com\3cb2d130ab99449ca91b275d68d47280\clipboard.png)

Horizontal Pod Authscaling	(HPA)

Pod横向扩容，属于资源对象，通过分析RC控制的所有目标Pod的负载变化情况。来确定是否需要针对性的调整目标Pod的副本数量。

Service （服务）

资源对象，每个Service其实就是我们经常提起的微服务，其它的Pod，RC等都是为Service提供嫁衣。

![img](D:\有道云\文件\super_linux@163.com\9ff1fdb346f44aea9dbef1ec5a29df27\clipboard.png)

tomcat-service.yaml

apiVersion: v1

kind: Service

metadata:

name:tomcat-service

sepc:

   ports:

​    -port: 8080

​    selector:

​     tier:frontend

$kubectl  create  -f   tomcat-service.yaml

$kubectl get endpoints   ##可以查看podIP 和Container 暴露的端口

$kubectl get svc tomcat-service  -o yaml   ##查看Cluster ip

![img](D:\有道云\文件\super_linux@163.com\ab0c1c13775d413da92d582730eb62a8\clipboard.png)

集群外部访问Service可以通过配置NodePort来解决访问问题

Volume (存储卷)

- Volume 是pod 能够被多个容器访问的共享目录
- 定义在pod上，被一个pod里的多个容器挂载到具体的文件夹下
- 当容器终止或者重启时，Volume数据不会丢失

例子，在pod里声明一个Volume

template:

metadata:

   labels:

​       app: app-demo

tire: frontend

spec:

volume:

-name: datavol

emptyDir: {}

containers:

-name:  tomcat-demo

 image: tomcat

volumeMounts:

  -mountPath: /mydata-data

   name: datavo1

imagePullPolicy:IfNotPresent

Presistent Volume(PV)

PV可以理解成kubernetes 集群的某个网络存储中对应的一块存储。

- PV只能是网络存储，不属于任何Node，但是在每个Node可以访问
- PV不是定义在Pod上的，独立于Pod之外定义
- 类型：GCE Persistent Disks、NFS、RBD、ISCSCI、AWS GLusterFS等

PV是有状态的：

- Available:空闲状态
- Bound:已经绑定到某个PVC上
- Released:对应的PVC已经删除，但是资源还没有被集群收回
- Falied:PV自动回收失败

NFS类型PV的yam定义文件

apiVersion: v1

kind: PersisrentVolume

metadata:

​     name: pv0003

spce: 

​      capasity: 

storage:5Gi

accessModes:

 	- ReadWriteOnce	

 nfs:

 	path: /somepath

server: 172.17.0.2

重要的accessModes属性：

- ReadWriteOnce:读写权限，并且只能被单个Node挂载
- ReadOnlyMany:只读权限，允许被多个Node挂载
- ReadWriteMany:读写权限，允许被多个Node挂载

如果某个Pod想要申请某种条件的PV ，需要定义一个PVC对象：

kind: PersistentVolumeClaim

apiVersion: v1

metadata:

name: myclaim

spec:

accessModes:

\- ReadWriteOnce

resources:

requests:

storages: 8Gi

然后在Pod的Volume 的定义里引入上述PVC即可

volumes:

\- name: mypd

 persistentVolumeClaim:

claimName: mycliam

Namespace(命名空间)

- 实现多租户的资源隔离
- 默认空间，default    通过kubectl get namespaces
- 不知道命名空间，默认是default

如下展示yaml定义Namespace

apiVersion: v1

king: Namespace

metadata:

name: development

如下定义名为busybox的Pod，命名空间是development

apiVersion: v1

king: Pod

metadata:

name: busybox

namespace: development

spec:

containers:

\- image: busybox

 	command:

\- sleep

\- "3600"

name: busybox

默认使用kubectl get pods查看到的是defalut空间的

$kubectl get pods --namespace=development查看