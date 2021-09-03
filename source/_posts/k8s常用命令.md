---
title: k8s常用命令
tags: kubernetes
---

##### **查看类命令**

1.获取节点相应服务的信息

```sh
kubectl get pods
```

按selector名来查找pod

```sh
kubectl get pod --selector name=redis
```

2.查看集群信息

```sh
kubectl cluster-info
```

3.查看各组件信息

```sh
kubectl -s http://localhost:8080 get componentstatuses
```

4.查看pods所在的运行节点

```sh
kubectl get pods -o wide
```

5.查看pods定义的详细信息

```sh
kubectl get pods -o yaml
```

6.查看指定pod的日志

```sh
kubectl logs -f pods.xxxx -n kube-system
```

7.删除污点

```sh
kubectl taint nodes --all node-role.kubernetes.io/master-
```

##### **操作类命令**

1.创建资源

```sh
kubectl create -f file.yaml
```

2.重建资源	

```sh
kubectl replace -f file [--force]
```

3.删除资源

```sh
kubectl delete -f file.yaml
kubectl delete pod podname
kubectl delete rc rcname
kubectl delete service servicename
kubectl delete pod --all
```

### **进阶命令**

##### 1.kubectl get :

```sh
kubectl get serviec kubernater-dashboard -n kube-system
kubectl get deplyment kubernetes-dashboard -n kube-system
kubectl get pods --all-namespaces
kubectl get pods -o wide --all-namespaces
kubectl get pods -n kube-system|grep dashbord
kubectl get nodes -lzone
```

##### 2.kubectl describe:

```sh
kubectl describe serviec/kubernetes-dashboard --namespace="kube-system"
kubectl describe pods/kubernetes-dashboard-xxxxx --namespace="kube-system"
kubectl describe pod nginx-2xxx
```

##### 3.kubectl scale:

```sh
kubectl scale rc nginx --replicas=5
kubectl scale deployment redis-slave --replicas=5
kubectl scale --replicas=2 -f redis-slave-deployment.yaml
```

##### 4.kubectl exec:

```sh
kubectl exec -it redis-master-1111111-xxxxx /bin/bash
```

##### 5.kubectl label:

```sh
kubectl label nodes nodel zone=north
kubectl label pod redis-master-xxxxx role=master
kubectl label pod redis-master-xxxxxx role-
kubectl label pod redis-master-xxxxxx role=backend --overwrite
```

##### 6.kubectl rolling-update:

```sh
kubectl rolling-update redis-master -f redis-master-controller-v2.yaml
kubectl rolling-update redis-master --image=redis-master:2.0
kubectl rolling-update redis-master --image=redis-master:1.0 --rollback
```

##### 7.etcdctl

```sh
etcdctl cluster-health
etcdctl --endpoints=https://192.168.1.1:2379 cluster-health
etcdctl member list
etcdctl set /k8s/network/config '{"Network":"10.1.0.0/16"}'
etcdctl get /k8s/network/config
```

