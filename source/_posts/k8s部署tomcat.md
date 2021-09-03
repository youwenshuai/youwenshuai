---
title: k8s部署tomcat
tags: kubernetes
---

##### **拉取tomcat镜像**

```sh
docker pull tomcat
cat webapp-rc.yaml   
apiVersion: v1
kind: ReplicationController
metadata: 
    name: webapp
spec:	  
    replicas: 3
    template:
        metadata: 
        name: webapp
        labels:
            app: webapp
        spec: 
            containers:
                - name: webapp
                image: tomcat
                 ports:
                 - containerPort: 8080                 	  
```

##### **创建RC文件//副本控制器**

```sh
kubectl create -f webapp-rc.yaml  
replicationcontroller/webapp created
kubectl get pods -l app=webapp 
NAME           READY   STATUS    RESTARTS   AGE
webapp-fq2qh   1/1     Running   0          56s
webapp-jbmzg   1/1     Running   0          56s
webapp-jsv4z   1/1     Running   0          57s
```

##### **获取Pod IP地址**

```sh
kubectl get pods -l app=webapp -o yaml
kubectl get pods -l app=webapp -o yaml | grep podIP
podIP: 10.1.7.31
podIP: 10.1.7.30
podIP: 10.1.7.29
curl 10.1.7.31:8080       //测试访问
curl 10.1.7.30:8080       //测试访问
curl 10.1.7.29:8080       //测试访问
kubectl exec -ti <your-pod-name>  -n <your-namespace>  -- /bin/sh 可进入替换war包测试
```

##### **使用expose 将webapp里的所有pod添加到服务**

```sh
kubectl expose rc webapp
service/webapp exposed
kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
webapp          ClusterIP   10.97.154.51   <none>        8080/TCP   		21s
curl 10.97.154.51:8080
//这里的访问被自动负载分发到后面三个pod之一
```

##### **或者通过配置文件Service 创建**

```sh
kubectl delete svc webapp  //删除刚刚通过命令创建的SVC
service "webapp" deleted
vi webapp-service.yaml 
apiVersion: v1
kind: Service
metadata:
   name: webapp
spec:
   ports:
 	  - port: 8088 	
 	    targetPort: 8080 	
 	  selector:
    	  app: webapp
```

字段解析：ports 关键字中指定了Service的虚拟端口8088，由于和pod容器端口8080不同，所以	需要targetPort指定后面pod的端口号。selector关键字设置了后端pod所拥有的label：app=webapp

```sh
kubectl create -f  webapp-service.yaml 
service/webapp created
kubectl get svc
NAME               TYPE              CLUSTER-IP          EXTERNAL-IP       PORT(S)         AGE
kubernetes       ClusterIP        10.96.0.1    	      <none>                  443/TCP           7d22h
redis-service    ClusterIP         None             	      <none>                  6379/TCP         6d4h
webapp            ClusterIP        10.102.174.4   	      <none>                  8088/TCP         7s
curl 10.102.174.4:8088
//这里的访问同样被自动负载分发到后面三个pod之一
```

名词解释：目前kubernetes提供的两种负载分发策略

RoundRobin：轮训模式

SesssionAffinity：基于客户端IP进行会话保持，会转发到相同的 pod上

##### **其他场景**

在某些场景下，开发人员希望自己控制负载均衡策略，不使用Service提供的负载均衡功能，kubernetes通过Headless Service实现。不设置ClusterIP（无入口IP地址），仅通过Label Selector 将后端的Pod列表返回给调用的客户端。

```sh
例1：
apiVersion: v1
kind: Service 
metadata:
    name: nginx
    labels:
    app: nginx
spec:
    ports:
    - port: 80
    clusterIP: None
    selector:
 app: nginx
```

该Service没有指定clusterIP，对其的访问将获得具有label "app=nginx"的全部pod列表，然后客户端程序需要自己实现负载分发策略，再确定访问具体哪一个后端的Pod。

```sh
例2：
apiVersion: v1
kind: Service 
metadata:
   name: my-service 
spec:
   ports: 
   - protocol: TCP
     port: 80
        targetPort: 80
```

该定义创建一个不带标签选择器的Service，即无法选择后端Pod，系统也不会自动创建Endpoint,因此需要手动创建一个和该Service同名的Endpoint，用于指定后端的访问地址。

```sh
apiVersion: v1
kind: Endpoints 
metadata:
    name: my-service
subsets:
- addresses: 
  - IP: 1.2.3.4
  ports:
  - port: 80
```

```sh
例3：
apiVersion: v1
kind: Service 
metadata:
    name: webapp
spec:
    ports:
    - port: 8080
      targetPort: 8080
      name: web
      - port: 8005
      targetPort: 8005
      name: management
      selector: 
      app: wepapp
定义多个端口
例4：
apiVersion: v1
kind: Service 
metadata:
name: kube-dns
namespace: kube-system
labels:
k8s-app: kube-dns
kubernetes.io/cluster-servcvice: "true"
kubernetes.io/name: "KubeDNS"
spec:
selector:
k8s-app: kube-dns
clusterIP:
ports:
- name: dns
   port: 53
   protocol: UDP
- name: dns-tcp
  port: 53
  protocol: TCP  
定义相同端口，不同协议
```

**7.集群外部访问Pod或Service**

1.将容器应用的端口号映射到主机端口

  （1）通过设置hostPort

​      \#vim webpod-hostport.yaml

apiVersion: v1

kind: Pod

metadata:

​    name: webapp1

​    labels:

 	     app: webapp1

spec:

​    containers:

​    \- name: webapp1

  	    image: tomcat
  	
  	    ports:
  	
  	     - containerPort: 8080
  	
  	     hostPort: 8081

\#kubectl create -f webpod-hostport.yaml

pod/webapp1 created

\#kubectl get pods

//此时通过浏览器访问 IP地址为该Pod运行在哪台Node上的地址

//通过此方式在Node节点看netstat -ntlp |grep 8081 不会有端口

​    （2）通过设置Pod级别的hostNetwork=true,该Pod的所有容器的端口号都将映射到物理

 	    机上，设置hostNetwork需要注意，在容器的ports部分如果不知道hostPort,则默认	 

​     hostPort等于containerPort,如果指定了hostPort,则hostPort必须等于containPort。

​     \#vim webpod-hostnetwork.yaml 

apiVersion: v1

kind: Pod

metadata:

​    name: webapp2

​    labels:

​       app: webapp2

spec:

​    hostNetwork: true

​    containers:

​    \- name: webapp2

​      image: tomcat

​      imagePullPolicy: Never

​      ports:

​      \- containerPort: 8080

\#kubectl create -f webpod-hostnetwork.yaml 

pod/webapp2 created

\#kubectl get pods

//此时通过浏览器访问 IP地址为该Pod运行在哪台Node上的地址

//通过此方式在Node节点看netstat -ntlp |grep 8080会有端口

2.将Service的端口号映射到物理机

（1）通过设置nodePort 映射到物理机，同时设置Service的类型为NodePort

\#kubectl delete -f  webpod-hostnetwork.yaml

\#kubectl get svc

\#kubectl delete svc webapp

\#vim webservice-nodeport.yaml

apiVersion: v1

kind: Service

metadata:

​    name: webapp3

spec:

​    type: NodePort

​    ports:

​    \- port: 8080

​      targetPort: 8080

​      nodePort: 32222

​    selector:

​     	   app: webapp

nodePort:取值范围30000-32767

//此时通过浏览器访问，将负载分发到后端的多个Pod上

（2）设置LoadBalancer映射到云服务厂商提供的LoadBalancer地址。负载分发方式依赖于	

  云服务厂商的LoadBalancer机制

\#vim loadBalancer-service.yaml

apiVersion: v1

kind: Service

metadata:

​    name: my-service 

spec:

​    selector:

​    app: myapp

​    ports:

​    \- protocol: TCP

​      port: 80

​      targetPort:	9376

​      nodePort: 30022

​    clusterIP: 10.0.2.2

​    loadBalancerIP: 22.22.22.22

​    type: LoadBalancer

status:

​    loadBalancer:

​      ingress:

​       \- ip:111.111.111.111