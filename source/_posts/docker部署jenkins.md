---
title: docker离线部署jenkins
tags: docker
---

`docker pull tomcat`

`docker images`

`docker run -p 8080:8080 --name mytomcat tomcat`

`docker exec -it mytomcat bash`

`docker cp jenkins.war mytomcat:/usr/local/tomcat/webapps`

`docker restart mytomcat`

> 启动docker容器时报错：
>
> iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 5000 -j DNAT --to-destination 172.18.0.4:5000 ! -i br-ff45d935188b: iptables: No chain/target/match by that name. (exit status 1)
>
> 解决方案：重启docker
>



