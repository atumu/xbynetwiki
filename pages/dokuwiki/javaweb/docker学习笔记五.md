title: docker学习笔记五 

#  docker学习笔记之docker实战 
##  在Docker中使用 Supervisor来管理进程 
docker 容器在启动的时候开启单个进程，比如，一个ssh或则apache 的daemon服务。但我们**经常需要在一个机器上开启多个服务**，这可以有很多方法，最简单的就是把多个启动命令方到一个启动脚
本里面，启动的时候直接启动这个脚本，**另外就是安装进程管理工具。**.
本小节将使用` 进程管理工具supervisor `来**管理容器中的多个进程。**使用Supervisor可以更好的控制、管理、重启我们希望运行的进程。在这里我们演示一下如何同时使用ssh和apache服务。

**1、Dockerfile文件**
```

FROM ubuntu:13.04
MAINTAINER examples@docker.com
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y
安装supervisor
安装 ssh apache supervisor
RUN apt-get install -y openssh-server apache2 supervisor
RUN mkdir -p /var/run/sshd
RUN mkdir -p /var/log/supervisor
这里安装3个软件，还创建了2个用来允许ssh和supervisor的目录
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
添加supervisor‘s的配置文件
添加配置文件到对应目录下面
映射端口，开启supervisor
使用dockerfile来映射指定的端口，使用cmd来启动supervisord
EXPOSE 22 80
CMD ["/usr/bin/supervisord"]
这里我们映射了22 和80端口，使用supervisord的可执行路径启动服务。

```

**2、supervisor配置文件内容**
```

[supervisord]
nodaemon=true
[program:sshd]
command=/usr/sbin/sshd -D
[program:apache2]
command=/bin/bash -c "source /etc/apache2/envvars && exec /usr/sbin/apache2
-DFOREGROUND"
配置文件包含目录和进程，第一段supervsord配置软件本身，使用nodaemon参数来运行。下面2段包
含我们要控制的2个服务。每一段包含一个服务的目录和启动这个服务的命令

```

**3、使用方法**
```

创建image
$ sudo docker build -t test/supervisord .
启动我们的supervisor容器
$ sudo docker run -p 22 -p 80 -t -i test/supervisords
2013-11-25 18:53:22,312 CRIT Supervisor running as root (no user in config file)
2013-11-25 18:53:22,312 WARN Included extra file "/etc/supervisor/conf.d/supervisord.conf"
during parsing
2013-11-25 18:53:22,342 INFO supervisord started with pid 1
2013-11-25 18:53:23,346 INFO spawned: 'sshd' with pid 6
2013-11-25 18:53:23,349 INFO spawned: 'apache2' with pid 7
使用docker run来启动我们创建的容器。使用多个-p 来映射多个端口，这样我们就能同时访问ssh和
apache服务了。

```

##  创建tomcat集群 
**1、安装tomcat镜像**
准备好需要的jdk tomcat等软件放到home目录下面，启动一个容器
```

docker run -t -i -v /home:/opt/data --name mk_tomcat ubuntu /bin/bash

```
这条命令挂载本地home目录到容器的/opt/data目录，容器内目录若不存在，则会自动创建。
接下来就是tomcat的基本配置，jdk环境变量设置好之后，将tomcat程序放到/opt/apache-tomcat下面
编辑/etc/supervisor/conf.d/` supervisor.conf `文件，**添加tomcat项**
```

[supervisord]
nodaemon=true
[program:tomcat]
command=/opt/apache-tomcat/bin/startup.sh
[program:sshd]
command=/usr/sbin/sshd -D

```
```

docker commit ac6474aeb31d tomcat

```
**新建tomcat文件夹，新建Dockerfile**
```

FROM tomcat
EXPOSE 22 8080
CMD ["/usr/bin/supervisord"]

```
###  根据dockerfile 创建image 
docker build tomcat tomcat

##  3、tomcat镜像的使用 
1)存储的使用
在启用docker run 的时候，` 使用 -v参数 `
-v, --volume=[] Bind mount a volume (e.g. from the host: -v /host:/container, from docker: -v
/container)
将本地磁盘映射到虚拟机内部，它在主机和虚拟机容器之间是实时变化的，所以我们更新程序、上传代
码只需要更新物理主机的目录就可以了，数据存储的详细介绍请参见本文第七小节

2)**tomcat集群**的实现
**tomcat只要开启多个容器即可**
```

docker run -d -v -p 204:22 -p 7003:8080 -v /home/data:/opt/data --name tm1 tomcat /usr/bin/supervisord
docker run -d -v -p 205:22 -p 7004:8080 -v /home/data:/opt/data --name tm2 tomcat /usr/bin/supervisord
docker run -d -v -p 206:22 -p 7005:8080 -v /home/data:/opt/data --name tm3 tomcat /usr/bin/supervisord

```

这样在**前端使用nginx 来做负载均衡就可以完成配置了**


##  mysql配置问题 
mysql data目录配置导致的权限问题：
启动时需要在运行时修改挂载宿主主机对于data目录的权限。通过supervisor进行：
[supervisord]
nodaemon=true
[program:sshd]
command=/usr/sbin/sshd -D
[program:dirchmod]
command=/bin/bash -c "chmod 700 /opt/mysqldata && chown mysql:mysql -R /opt/mysqldata"
[program:mysql]
command=/usr/sbin/mysqld