---
title: jenkins部署
tags: java
---

##### java安装

```sh
yum -y install java
java -version
openjdk version "11-ea" 2018-09-25
OpenJDK Runtime Environment (build 11-ea+28)
OpenJDK 64-Bit Server VM (build 11-ea+28, mixed mode, sharing)
cat /etc/profile
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.ea.28-7.el7.x86_64
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile  
```

##### 安装tomcat环境

```sh
tar xf apache-tomcat-8.5.32.tar.gz 
mv apache-tomcat-8.5.32 tomcat8
```

##### 安装jenkins

下载最新版war包

```sh
[root@RHEL7 webapps]# vim /usr/local/tomcat8/conf/server.xml
<Connector port="8080" URIEncoding="UTF-8"  protocol="HTTP/1.1"
JENKINS_HOME="/usr/local/tomcat8/webapps/jenkins"
export JENKINS_HOME
[root@RHEL7 webapps]# source /etc/profile
[root@RHEL7 webapps]# echo $JENKINS_HOME
/usr/local/tomcat8/webapps/jenkins
[root@RHEL7 webapps]# /usr/local/tomcat8/bin/startup.sh 
Using CATALINA_BASE:   /usr/local/tomcat8
Using CATALINA_HOME:   /usr/local/tomcat8
Using CATALINA_TMPDIR: /usr/local/tomcat8/temp
Using JRE_HOME:        /usr/lib/jvm/java-11-openjdk-11.0.ea.28-7.el7.x86_64
Using CLASSPATH:       /usr/local/tomcat8/bin/bootstrap.jar:/usr/local/tomcat8/bin/tomcat-juli.jar
Tomcat started.
访问 http://192.168.1.3:8080/jenkins 
不成功关闭防火墙
```

