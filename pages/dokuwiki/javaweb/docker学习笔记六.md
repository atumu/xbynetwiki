title: docker学习笔记六 

#  Docker学习笔记之容器互联 

###  单机内容器互联 
docker有一个**linking 系统可以连接多个容器**。它会创建一对父子关系，父容器可以看到所选择的子容器的信息。

**1)容器的命名系统**
**linking系统依据容器的名称来执行。**当我们创建容器的时候，系统会随机分配一个名字。当然我们也可以自己来命名容器，
	（通过--name myname选项）
这样做有2个好处：
• 当我们自己指定名称的时候，比较好记，比如一个web应用我们可以给它起名叫web
• 当我们要连接其他容器时候，可以作为一个有用的参考点，比如连接web容器到db容器

```

使用--name标记可以为容器命名
$ sudo docker run -d -P --name web training/webapp python app.py
使用docker -ps 来验证我们设定的命名
$ sudo docker ps -l
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS
NAMES
aed84ee21bde training/webapp:latest python app.py 12 hours ago Up 2 seconds 0.0.0.0:49154-
>5000/tcp web
也可以使用docker inspect来查看容器的名字
$ sudo docker inspect -f "![](/data/dokuwiki .Name )" aed84ee21bde
/web

```
注意：**容器的名称是唯一的**。如果你命名了一个叫web的容器，当你要再次使用web这个名称的时候，你需要用` docker rm `来删除之前创建的容器，也可以再执行docker run的时候 加—rm标记来停止
旧的容器，并删除**，rm 和-d 参数是不兼容的**。

**容器互联**
links可以让容器之间安全的交互，
	使用--link标记。--link标记的格式：--link name:alias，name是我们要链接的容器的名称，alias是这个链接的别名。
下面先创建一个新的数据库容器，
$ sudo docker run -d --name db training/postgres
删除之前创建的web容器
$ docker rm -f web
创建一个新的web容器，并将它link到db容器
$ sudo docker run -d -P --name web --link db:db training/webapp python app.py

使用docker ps来查看容器的链接
```

$ docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS
PORTS NAMES
349169744e49 training/postgres:latest su postgres -c '/usr About a minute ago Up About a minute 5432/tcp db, web/db
aed84ee21bde training/webapp:latest python app.py 16 hours ago Up 2 minutes 0.0.0.0:49154->5000/tcp web

```
我们可以看到我们命名的容器，db和web，**db容器的names列有db也有web/db。这表示web容器链接到db容器，他们是一个父子关系**。在这个link中，2个容器中有一对父子关系。docker在2个容器之间创建了一个安全的连接，而且**不用映射他们的端口到宿主主机上**。在启动db容器的时候也不用-p和-P标记。` 使用link之后我们就可以不用暴露数据库端口到网络上 `。

docker 通过2种方式为父子关系的容器公开连接信息：
• 环境变量
• 更新/etc/hosts文件
使用env命令来查看容器的环境变量
```

$ sudo docker run --rm --name web2 --link db:db training/webapp env
. . .
DB_NAME=/web2/db
DB_PORT=tcp://172.17.0.5:5432
DB_PORT_5000_TCP=tcp://172.17.0.5:5432
DB_PORT_5000_TCP_PROTO=tcp
DB_PORT_5000_TCP_PORT=5432
DB_PORT_5000_TCP_ADDR=172.17.0.5
. . .

```
除了环境变量，docker还添加host信息到父容器的/etc/hosts的文件。下面是**父容器web**的hosts文件
```

$ sudo docker run -t -i --rm --link db:db training/webapp /bin/bash
root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7 aed84ee21bde
. . .
172.17.0.5 db

```
这里有2个hosts，第一个是web容器，web容器用id作为他的主机名，第二个是db容器的ip和主机名

```

root@aed84ee21bde:/opt/webapp# apt-get install -yqq inetutils-ping
root@aed84ee21bde:/opt/webapp# ping db
PING db (172.17.0.5): 48 data bytes
56 bytes from 172.17.0.5: icmp_seq=0 ttl=64 time=0.267 ms
56 bytes from 172.17.0.5: icmp_seq=1 ttl=64 time=0.250 ms
56 bytes from 172.17.0.5: icmp_seq=2 ttl=64 time=0.256 ms

```
用ping来ping db容器，它会解析成172.17.0.5
注意：官方的ubuntu镜像默认没有安装ping
注意：你可以链接多个子容器到父容器，比如我们可以链接多个web到db容器上。

###  跨主机容器互联 
参考：http://www.tuicool.com/articles/RfQRny
http://www.bkjia.com/yjs/931699.html
Docker的功能非常强大，但要想驾驭好Docker却不是一件很容易的事情。下面就介绍一种日常工作中会遇到的一个user case。比如现在有两台host，分别标记为hostA和hostB。hostA用来运行oracle服务，hostB用来运行app服务。
hostB中app产生的数据需要实时写入hostA中的oracle数据库。也就是hostB中的docker container需要link hostA中的docker container。
为了解决这个问题，有两个解决方案：
方案一：
将hostA中的oracle container对外expose 1521(我们假定此处对外expose 1521)，然后在hostB中的app container中修改/etc/hosts文件，将hostA的IP添加到hosts文件中。
![](/data/dokuwiki/javaweb/pasted/20151012-111705.png)

这种方案的优点就是可以根据实际情况自由配置," 自己的app掌控在自己手中 "。
但是缺点也很严重，首先每次run container时都需要修改hosts文件，而且每次host环境发生变化，都需要维护hosts文件，因此后续的维护成本很高。其次，如果遇到其他人开发的docker image，我们未必有权限来修改hosts文件。
所以此方案也仅仅用作开发测试使用，不推荐正式采用。

**方案二：**
Docker官方提供了一种` ambassador `的agent方案。此方案借助一个名为` svendowideit/ambassador `的image，将不同host进行解耦合。
![](/data/dokuwiki/javaweb/pasted/20151012-111800.png)
具体实施步骤如下：
首先我们要在hostA和hostB：
	docker pull svendowideit/ambassador
根据部署图可得知，是hostB要link hostA的container，因此我们需要先启动hostA的container(此时可以理解hostA为server段，hostB为client段)。
docker run  -d  --name oracle tirtool/oracle11g:latest
然后启动hostA中的ambassador container。
docker run  -d ` --link oracle:oracle ` -p 1521:1521 --name ambassador svendowideit/ambassador:latest
标红的部分是需要重点注意的地方，这个命令也就是将hostA中的oracle container直接暴露在ambassador之中，这样ambassador才能将访问请求转发到oracle container中。

此刻，hostA中的事情都已经处理完了，我们开始处理hostB。
**与hostA的container启动顺序不用，我们需要先start hostB的ambassador**。因为依据部署图可知，hostB中是app container调用ambassador container，所以需要先保证ambassador已经启动，才能启动app container。
	docker run -d --name ambassador-oracle --expose 1521 -e ORACLE_PORT_1521_TCP=tcp://<<hostA IP>>:1521 svendowideit/ambassador
当hostB中的ambassador启动成功后，我们开始启动app container。
docker run --link ambassador-oracle:oracle  --name bw base/ubuntu:14.04
**大家注意到我们在app container中将ambassador-oracle alias为oracle，这样在app container中的/etc/hosts文件中会出现一条记录：
10.1.0.3        oracle**
此刻app container中app产生数据后，如果调用oracle:1521，那么首先将请求发往hostB的ambassador的1521端口。hostB的ambassador会将数据转发到hostA的1521端口。而此时hostA中的ambassador在listen 1521端口，接收到请求后会将数据转发至hostA的oracle container中。oracle container处理完毕后将response返回值ambassador，ambassador再依次回传。从而达到不同host中container相互访问的目的。

我们再看一下在hostB中执行--expose 1521 -e ORACLE_PORT_1521_TCP=tcp:/ / <<hostA IP>>:1521 时发生了什么事情。**ambassador最重要的一项任务就是将hostB的1521端口同hostA的1521端口进行了端口映射。**
因此ambassador方案就是很巧妙的**将不同host的port进行了桥接，而这些对docker使用者都是透明的**。但这个方案也是有一些瑕疵的，**就是如果新增container之后，需要重启或者新增ambassador，**所以如果一个ambassador同时对应多个container，那么在维护上面就会稍许麻烦些，但维护成本比方案一低了很多。

**实战示列：**
如上面所说，**link只适用于一台主机。**
两台主机，docker官方推荐了如下方式连接两个容器。
以下以wordpress+mysql的服务为例。部署在两台机器上的wordpress和mysql通过一对ambassador进行连接。
wordpress(in vm1)--link-->ambassador1(in vm1)----socat--->ambassador2(in vm2)--link--->mysql(in vm2)
**` 注意：顺序特别重要 `**
```

启动mysql：
sudo docker run -d --name mysql mysql
启动ambassador1：
sudo docker run -d --link mysql:mysql --name ambassador1 -p 3306:3306 ambassador  
启动ambassador2：
sudo docker run -d --name ambassador2 --expose 3306 -e MYSQL_PORT_3306_TCP=tcp://x.x.x.x:3306 ambassador  
启动wordpress:
sudo docker run -i -t --rm --link ambassador2:mysql wordpress

```