title: docker学习笔记七 

#  windows使用docker 
CMD相关命令：http://docs.docker.com/installation/mac/#migrate-from-boot2docker
以前采用的管理工具是boot2docker.exe现在采用的是docker-machine.exe
以下是这两个命令选项对比：
![](/data/dokuwiki/javaweb/pasted/20151013-053724.png)

命令示列：
**其中的default是创建的vm虚拟机名字**
查看IP:
D:\Program Files\Docker Toolbox>` docker-machine.exe ip default `
```

192.168.99.100

```
查看环境变量
D:\Program Files\Docker Toolbox>` docker-machine.exe env default --shell cmd `
```

set DOCKER_TLS_VERIFY=1
set DOCKER_HOST=tcp://192.168.99.100:2376
set DOCKER_CERT_PATH=C:\Users\Administrator\.docker\machine\machines\default
set DOCKER_MACHINE_NAME=default
# Run this command to configure your shell:
# copy and paste the above values into your command prompt

```
