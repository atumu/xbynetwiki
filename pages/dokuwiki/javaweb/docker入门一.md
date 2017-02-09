title: docker入门一 

#  docker入门一 
参考:http://www.docker.org.cn/  http://special.csdncms.csdn.net/BeDocker/  https://docs.docker.com/linux/started/
https://code.csdn.net/u010702509/docker/file/Docker.md  http://blog.opskumu.com/docker.html
Docker 是一个开源的应用容器引擎，可以轻松的为任何应用创建一个轻量级的、**可移植的**、自给自足的容器。开发者在笔记本上编译测试通过的容器可以批量地在生产环境中部署，包括VMs（虚拟机）、bare metal、OpenStack 集群和其他的基础应用平台。 
Docker通常用于如下场景：
  * web应用的自动化打包和发布；
  * 自动化测试和持续集成、发布；
  * 在服务型环境中部署和调整数据库或其他的后台应用；
  * 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。

**开发者通过Docker可以将App变成一种标准化的、可移植的、自管理的组件，可以在任何主流系统中开发、调试和运行。最重要的是，它不依赖于任何语言、框架或系统。** 不久前Docker 1.0的发布，意味着Docker自身已经转变为一个分发应用的开放平台。
Docker ：码头搬运工，这种搬运工搬运的是集装箱(Container)，集装箱里面装的是任意 类型的App，**Docker把App(叫Payload)装在 Container内，通过Linux Container技术的包装将App变成一种标准化的，可移植的，自管理的组件,这种组件可以在你的本地机器上开发，调试运行，最终非常方便和一致地允许在production环境下。**

**Docker 容器相对于 VM 有以下几个优点：**

  * 启动速度快，容器通常在一秒内可以启动，而 VM 通常要更久
  * 资源利用率高，**一台普通 PC 可以跑上千个容器**，你跑上千个 VM 试试
  * 性能开销小， VM 通常需要额外的 CPU 和内存来完成 OS 的功能，这一部分占据了额外的资源

##  为啥要用容器？ 

那么应用容器长什么样子呢，一个做好的应用容器长得就好像一个装好了一组特定应用的虚拟机一样。比如我现在想用MySQL那我就找个装好MySQL的容器，运行起来，那么我就可以使用 MySQL了。
那么我直接装个 MySQL不就好了，何必还需要这个容器这么诡异的概念？话是这么说，可是你要真装MySQL的话可能要再装一堆依赖库，根据你的操作系统平台和版本进行设置，有时候还要从源代码编译报出一堆莫名其妙的错误，可不是这么好装。而且万一你机器挂了，所有的东西都要重新来，可能还要把配置在重新弄一遍。**但是有了容器，你就相当于有了一个可以运行起来的虚拟机，只要你能运行容器，MySQL的配置就全省了。而且一旦你想换台机器，直接把这个容器端起来，再放到另一个机器就好了。硬件，操作系统，运行环境什么的都不需要考虑了。**
在公司中的一个很大的用途就是可以` 保证线下的开发环境、测试环境和线上的生产环境一致。 `
若果利用容器的话，那么开发直接在容器里开发，提测的时候把整个容器给测试，测好了把改动改在容器里再上线就好了。通过容器，整个开发、测试和生产环境可以保持高度的一致。
此外容器也和VM一样具有着一定的隔离性，**各个容器之间的数据和内存空间相互隔离，可以保证一定的安全性。**
![](/data/dokuwiki/javaweb/pasted/20151010-032511.png)

##  一、Docker 简介 
**Docker 两个主要部件：**
  * Docker: 开源的容器虚拟化平台
  * Docker Hub: 用于分享、管理 Docker 容器的 Docker SaaS 平台 -- Docker Hub
**Docker 使用客户端-服务器 (C/S) 架构模式**。Docker 客户端会与 Docker 守护进程进行通信。Docker 守护进程会处理复杂繁重的任务，例如建立、运行、发布你的 Docker 容器。Docker 客户端和守护进程可以运行在同一个系统上，当然你也可以使用 Docker 客户端去连接一个远程的 Docker 守护进程。Docker 客户端和守护进程之间通过 socket 或者 RESTful API 进行通信。

**Docker 内部**
  * Docker 镜像 - Docker images
  * Docker 仓库 - Docker registeries
  * Docker 容器 - Docker containers

Docker 镜像
**Docker 镜像是 Docker 容器运行时的只读模板，每一个镜像由一系列的层 (layers) 组成**。Docker 使用 UnionFS 来将这些层联合到单独的镜像中。UnionFS 允许独立文件系统中的文件和文件夹(称之为分支)被透明覆盖，形成一个单独连贯的文件系统。**正因为有了这些层的存在，Docker 是如此的轻量。**当你改变了一个 Docker 镜像，比如升级到某个程序到新的版本，一个新的层会被创建。因此，不用替换整个原先的镜像或者重新建立(在使用虚拟机的时候你可能会这么做)，只是一个新的层被添加或升级了。现在你不用重新发布整个镜像，只需要升级，**层使得分发 Docker 镜像变得简单和快速**。

Docker 仓库
Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。同样的，Docker 仓库也有公有和私有的概念。**公有的 Docker 仓库名字是 Docker Hub**。Docker Hub 提供了庞大的镜像集合供使用。这些镜像可以是自己创建，或者在别人的镜像基础上创建。Docker 仓库是 Docker 的分发部分。

Docker 容器
Docker 容器和文件夹很类似，一个Docker容器包含了所有的某个应用运行所需要的环境。**每一个 Docker 容器都是从 Docker 镜像创建的。**Docker 容器可以运行、开始、停止、移动和删除。每一个 Docker 容器都是独立和安全的应用平台，Docker 容器是 Docker 的运行部分。

` 注意：docker默认以root权限运行它的守护程序 `
##  docker基础命令： 
  * 查看版本：$docker version
  * 查看帮助：sudo docker help
  * 搜索可用的docker镜像：docker search 镜像名字 如：` docker search hello-world `
  * 下载容器镜像:` docker pull learn/tutorial `(在docker的镜像索引网站上面，镜像都是按照**用户名/镜像名**的方式来存储的。有一组比较特殊的镜像，比如ubuntu这类基**础镜像，经过官方的验证，值得信任，可以直接用镜像名来检索到**。执行pull命令的时候要写完整的名字，比如"learn/tutorial"。)
  * 在docker容器中运行:` docker run hello-world `(docker容器可以理解为在沙盒中运行的进程。**这个沙盒包含了该进程运行所必须的资源，包括文件系统、系统类库、shell 环境等等。但这个沙盒默认是不会运行任何程序的。**你需要在沙盒中运行一个进程来启动某一个容器。这个进程是该容器的唯一进程，所以当该进程结束的时候，容器也会完全的停止。)
  * 在容器中安装新的程序:docker run learn/tutorial apt-get install -y ping(下一步我们要做的事情是在容器里面安装一个简单的程序(ping)。我们之前下载的tutorial镜像是基于ubuntu的，所以你可以使用ubuntu的apt-get命令来安装ping程序：apt-get install -y ping。备注：apt-get 命令执行完毕之后，容器就会停止，但对容器的改动不会丢失。在执行apt-get 命令的时候，` 要带上-y参数。 `如果不指定-y参数的话，apt-get命令会进入交互模式，需要用户输入命令来进行确认，` 但在docker环境中是无法响应这种交互的。 `)
  * 当你对某一个容器做了修改之后，可以**把对容器的修改保存下来**，这样下次可以从保存后的最新状态运行该容器。**docker中保存状态的过程称之为committing**，它保存的新旧状态之间的区别，从而产生一个新的版本。
首先使用` docker ps -l `命令获得安装完ping命令之后**容器的id**。然后把这个镜像保存为learn/ping。
提示：
1. 运行` docker commit `，可以查看该命令的参数列表。
2. 你需要指定要提交保存容器的ID。(译者按：通过docker ps -l 命令获得)
3. **无需拷贝完整的id，通常来讲最开始的三至四个字母即可区分**。（译者按：非常类似git里面的版本号)
正确的命令：
```

$docker commit 698 learn/ping

```
  * 运行新的镜像:$docker run lean/ping ping www.google.com
  * 检查运行中的镜像:使用` docker ps `命令可以查看所有正在运行中的容器列表，使用docker inspect命令我们可以查看更详细的关于某一个容器的信息。
  * 停止、重启容器进程：sudo docker stop/restart 容器ID
  * 查看日志： docker logs 容器ID
  * 删除容器:docker rm 容器id
  * 删除镜像：docker rmi 镜像ID或镜像名 （镜像可以通过它们的长或者短ID删除或者他们的镜像名称，**如果一个镜像有一个或者多个名称，在删除镜像之前，他们每一个都必须要删除**）
  * 发布docker镜像:
我们可以将其**发布到官方的索引网站**(类似于Maven)。还记得我们最开始下载的learn/tutorial镜像吧，我们也可以把我们自己编译的镜像发布到索引页面。
目标：把learn/ping镜像发布到docker的index网站。
提示：
1. ` docker images `命令可以列出所有安装过的镜像。
2. ` docker push `命令可以将某一个镜像发布到官方网站。
3. 你只能将镜像发布到自己的空间下面。这个模拟器登录的是learn帐号。
预期的命令：
$ docker push learn/ping

##  centos7下安装docker 
一、安装docker
docker参考网址：http://www.docker.org.cn
前提：
Docker目前可以在centos7版本下面安装。
依赖性检查
Docker需要一个64位系统的centos7系统，内核的版本必须大于3.10。可以用下面的命令来检查是否满足docker的要求。
检测方法：输入命令：uname -r

1、禁用SELinux。
由于SELinux与dokcer会发生冲突，所以需要禁用。
sudo vim /etc/selinux/config进入编辑界面修改配置为
SELINUX=disabled 
SELINUXTYPE=targeted
修改保存之后重启：
sudo reboot

2、 sudo yum update 更新软件源。（可选）
3、curl -ssl https://get.docker.com/ | sh  进行具体下载安装。
4、更改docker运行用户，docker默认使用root用户运行，为了避免每次运行docker命令的时候都需要输入sudo:
sudo usermod -aG docker admin 此处的admin是自己创建的具有sudo权限的用户.
5、sudo service docker restart 启动docker服务。
6、确认安装：运行hello-world玩玩：
sudo docker run hello-world
7、将docker设置成开机自启动：
sudo chkconfig docker on

##  配置Docker守护进程 
配置绑定主机与端口：
```

sudo /usr/bin/docker -d -H tcp://0.0.0.0：2375或者export DOCKER_HOST="tcp://0.0.0.0:2375"

```

` **编辑启动配置文件** `
  * **ubuntu下**：` /etc/default/docker `，并修改` DOCKER_OPTS `变量
  * **RedHat系列**:` /usr/lib/systemd/system/docker.service ` 并修改其中的` ExecStart `配置项

Docker图形管理工具：
  * Shipyard
  * DockerUI
  * maDocker
##  hello-world运行示列 
下载ubuntu base镜像
sudo docker pull ubuntu
(注：非官方)相对比较而且你可以下载busybox镜像，他是最小的linux系统，这个镜像可以从docker仓库获取！sudo docker pull busybox
sudo docker run ubuntu /bin/echo hello world
这个命令会运行一个简单的echo 命令，这个时候我们的控制其就会输出"hello word"。
讲解:
  * sudo 执行root权限
  * **docker run 运行一个新的容器**
  * ubuntu 我们想要在容器内部运行命令的镜像
  * /bin/echo 我们想要在内部运行的命令
  * hello word 输出的内容

这个例子环境是假设你已经运行了docker进程，更多详细信息请查看运行例子，如果你不喜欢sudo，你可以用户授权命令和docker组
这是有史以来最无聊的进程
这个例子假设是你已经安装了docker，并且你已经通过docker pull下载导入了ubuntu镜像，我们将会用到ubuntu镜像运行一些姜丹的例子hello word进程，这里的将会每秒钟打印出hello word一直到我们停止他
CONTAINER_ID=$(sudo docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done")
我们已经**用ubuntu镜像新建一个容器并且运行了一个简单的hello world进程**
  * sudo docker run -d 运行提个新的容器，我们通过**-d命令让他作为一个进程运行**
  * ubuntu 是一个我们想要在内部运行命令的镜像
  * /bin/sh -c 是我们想要在容器内部运行的命令
  * while true; do echo hello world; sleep 1; done 这是一个简单的脚本，我们仅仅只是每秒打印一次hello word 一直到我们结束它
  * $CONTAINER_ID 我们**运行命令将会返回一个容器id**

##  ubuntu中安装Docker 
检查内核版本:uname -r 或者uname -a 必须3.10以上内核版本
升级ubuntu12.04内核：
```

sudo apt-get update
#sudo apt-get install linux-headers-3.8.0-27-generic  linux-headers-3.8.0-27 linux-image-3.8.0-27-generic
sudo apt-get install linux-image-generic-lts-raring linux-headers-generic-lts-raring
sudo update-grub
sudo reboot

```

安装docker:
```

sudo sh -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
sudo apt-get install -y curl
curl -s https://get.docker.io/gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install  docker-engine

```

创建Docker用户组：
```

sudo usermod -aG docker abcName

```

Adjust memory and swap accounting:
```

sudo vim /etc/default/grub
设置GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
sudo update-grub
sudo reboot

```


配置UFW防火墙：启用网桥数据包转发
```

sudo vim /etc/default/ufw
修改DEFAULT_FORWARD_POLICY="DROP" 为DEFAULT_FORWARD_POLICY="ACCEPT"
sudo ufw reload
sudo ufw allow 2375/tcp #2375为docker守护程序的端口号

```

配置开机启动:
```

sudo systemctl enable docker或者sudo chkconfig docker on 或者 sudo update-rc.d -f docker defaults

```

##  使用Docker安装脚本进行安装 
```

curl https://get.docker.io/ | sudo sh
sudo usermod -aG docker admin

```
