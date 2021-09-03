---
title: jdk的安装（java环境的安装）
tags: java
---

```sh
tar xf jdk-8u131-linux-x64.tar.gz
mv jdk1.8.0_121/  /opt/jdk1.8
rpm -qa | grep jdk
rpm -e --nodeps  java-1.7.0-openjdk-javadoc-1.7.0.45-2.4.3.3.el6.noarch java-1.7.0-openjdk-1.7.0.45-2.4.3.3.el6.x86_64 java-1.6.0-openjdk-devel-1.6.0.0-1.66.1.13.0.el6.x86_64 java-1.7.0-openjdk-devel-1.7.0.45-2.4.3.3.el6.x86_64 java-1.6.0-openjdk-javadoc-1.6.0.0-1.66.1.13.0.el6.x86_64 java-1.6.0-openjdk-1.6.0.0-1.66.1.13.0.el6.x86_64
vi /etc/profile.d/jdk.sh###编辑配置文件
export JAVA_HOME=/opt/jdk1.8
export PATH=$JAVA_HOME/bin:$PATH
cd  /etc/profile.d
chmod +x  /etc/profile.d/jdk.sh #####加权限
source /etc/profile.d/jdk.sh ###重新启动
sh jdk.sh 
echo $?
java -version  #####查看脚本
```

