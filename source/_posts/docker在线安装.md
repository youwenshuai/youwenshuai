---
title: docker在线安装
tags: docker
---

**1.docker安装**

1、Docker 要求 CentOS 系统的内核版本高于 3.10 

通过 **uname -r** 命令查看你当前的内核版本

`uname -r`

2、使用 root 权限登录 Centos。确保 yum 包更新到最新。

 `sudo yum update`

3、卸载旧版本(如果安装过旧版本的话)

`sudo yum remove docker  docker-common docker-selinux docker-engine`

4、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

`sudo yum install -y yum-utils device-mapper-persistent-data lvm2`

5、设置yum源

`sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

6、可以查看所有仓库中所有docker版本，并选择特定版本安装

`yum list docker-ce --showduplicates | sort -r`

7、安装docker

` sudo yum install docker-ce `

8、启动并加入开机启动

`sudo systemctl start docker $ sudo systemctl enable docker`

9、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

`docker version`

> 配置加速
>
> sudo mkdir -p /etc/docker sudo tee /etc/docker/daemon.json <<-'EOF' {  "registry-mirrors": ["https://27zv9ros.mirror.aliyuncs.com"] } EOF sudo systemctl daemon-reload sudo systemctl restart docker
>
> Ps：可能会在装的时候缺少包，单独下载安装rpm包即可
>

**2. Docker使用**

**2.1 docker基本信息查看**

（1）docker version：查看docker的版本号，包括客户端、服务端、依赖的Go等 ；

 （2）docker info ：查看系统(docker)层面信息，包括管理的images, containers数等；

 **2.2 docker镜像的获取与删除**

(1) docker pull centos ：下载centos所有的镜像

(2)docker pull centos:centos6  下载centos6镜像

(3)docker images  查看本机所有的镜像包

(4)docker images -a 列出所有的images（包含历史）

(5)docker 导入镜像  docker load --input ~/centos-7.3.tar

(6)docker挂载主机目录 -v

docker run -i -t -v /root/engine/:/root/engine centos /bin/bash

(7)docker 容器镜像删除

①.停止所有的container，这样才能够删除其中的images：

docker stop $(docker ps -a -q)

如果想要删除所有container的话再加一个指令：

docker rm $(docker ps -a -q)

②.查看当前有些什么images

docker images

③.删除images，通过image的id来指定删除谁

docker rmi <image id>

想要删除untagged images，也就是那些id为<None>的image的话可以用

docker rmi $(docker images | grep "^<none>" | awk "{print $3}")

要删除全部image的话

docker rmi $(docker images -q)

**2.3 docker容器的使用**

**(1)docker命令帮助**

docker的管理命令都是以docker开头，加上一个容易理解的单词，对一个命令的参数不熟悉，可以使用docker command --help查看相关参数意义,例如：

(2)**docker使用教程**

http://www.runoob.com/docker/docker-tutorial.html

**3. Docker镜像导入导出**

docker 提供把镜像导出export（保存save）为文件的机制，这样就可以把镜像copy到任意地方了。

**3.1 方式一:容器export/import**

格式：docker export CONTAINER(容器)

使用 docker ps -a 查看本机已有的容器.

**3.2 方式一:镜像save/load**

格式：docker save IMAGE(镜像)

使用  docker commit <CONTAIN-ID> <IMAGE-NAME>命令把一个正在运行的容器保存为镜像，如：

(1)首先查看所有的容器信息

(2）使用docker commit命令将指定容器保存为镜像

（3）使用docker images 查看刚才保存的镜像

（4）使用docker run 命令测试刚才保存的镜像是否正确

（5）使用docker save 命令将镜像导出成文件

 现在就可以在任何装 docker 的地方加载 刚保存的镜像了。

 查看加载的镜像

 查看刚刚加载的镜像tag为<none>，现在可修改下镜像的tag名称：

 **4.** **利用****dockerfile构建镜像**

docker build命令会根据Dockerfile文件及上下文构建新Docker镜像。构建上下文是指Dockerfile所在的本地路径或一个URL（Git仓库地址）。构建上下文环境会被递归处理，所以，构建所指定的路径还包括了子目录，而URL还包括了其中指定的子模块。

**构建镜像**

将当前目录做为构建上下文时，可以像下面这样使用docker build命令构建镜像：

$ docker build .

Sending build context to Docker daemon 6.51 MB

...

说明：构建会在Docker后台守护进程（daemon）中执行，而不是CLI中。构建前，构建进程会将全部内容（递归）发送到守护进程。大多情况下，应该将一个空目录作为构建上下文环境，并将Dockerfile文件放在该目录下。

在构建上下文中使用的Dockerfile文件，是一个构建指令文件。为了提高构建性能，可以通过.dockerignore文件排除上下文目录下，不需要的文件和目录。

Dockerfile一般位于构建上下文的根目录下，也可以通过-f指定该文件的位置：

$ docker build -f /path/to/a/Dockerfile .

构建时，还可以通过-t参数指定构建成后，镜像的仓库、标签等：

**镜像标签**

$ docker build -t shykes/myapp .

**5.docker常用命令**

**docker search:** Search the Docker Hub for images 

**docker pull:** Pull an image or a repository from a registry 

**docker imases:** List images

**docker create:** Create a new container

**docker start:** Start one or more stopped containers

**docker run:** Run a command in a new container

**docker attach:** Attach to running container

**docker ps:** List containers

**docker logs:** Fetch the logs of a container

**docker restart:** Restart a container

**docker stop:** Stop one or more running containers 

**docker kill:** Kill one or more running containers

**docker rm:** Remove one or more containers