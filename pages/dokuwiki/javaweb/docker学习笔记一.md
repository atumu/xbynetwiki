title: docker学习笔记一 

#  docker学习笔记一:常用命令 
参考：http://blog.opskumu.com/docker.html
##  交互运行 
```

Running an interactive shell
$ sudo docker run -i -t ubuntu:14.04 /bin/bash
docker run - 运行一个容器
-t - 分配一个（伪）tty (link is external)
-i - 交互模式 (so we can interact with it)
ubuntu:14.04 - 使用 ubuntu 基础镜像 14.04
/bin/bash - 运行命令 bash shell
注: ubuntu 会有多个版本，通过指定 tag 来启动特定的版本 [image]:[tag]

```
##  如何让docker以daemon方式运行/bin/bash 
**答案：不能**
**docker run指定的命令如果不是那些一直挂起的命令（比如运行top，不断echo），就是会自动退出的。**
**-d命令是设置detach为true，根据官方的文档，意思是让这个命令在后台运行，但并不是一直运行**（我们在一个正常的Linux Terminal中运行/bin/bash，运行完了也就完了，不会一直挂着等待响应的，**所以确实没办法用daemon方式来跑/bin/bash**）。
这个地方官方早期和现在的文档也确实有些前后不一致，现在是detach，早期的文档说指定-d以daemon方式来运行容器，可能存在一定的误解。
另外，如果你需要跑容器里的bash，直接运行docker run -i -t CONTAINER_NAME /bin/bash 就可以了，如果觉得参数比docker attach多，可以设置一个别名（alias）来解决：
alias dockerbash='docker run -i -t CONTAINER_ID /bin/bash'
设置好别名后，直接运行dockerbash就可以进入容器的bash了


###  相关快捷键 
退出：Ctrl-D or exit
detach：Ctrl-P + Ctrl-Q
attach:docker attach CONTAINER-ID
##  命令选项 
```

Usage of docker:
  --api-enable-cors=false                Enable CORS headers in the remote API                      # 远程 API 中开启 CORS 头
  -b, --bridge=""                        Attach containers to a pre-existing network bridge         # 桥接网络
                                           use 'none' to disable container networking
  --bip=""                               Use this CIDR notation address for the network bridge's IP, not compatible with -b
                                         # 和 -b 选项不兼容，具体没有测试过
  -d, --daemon=false                     Enable daemon mode                                         # daemon 模式
  -D, --debug=false                      Enable debug mode                                          # debug 模式
  --dns=[]                               Force docker to use specific DNS servers                   # 强制 docker 使用指定 dns 服务器
  --dns-search=[]                        Force Docker to use specific DNS search domains            # 强制 docker 使用指定 dns 搜索域
  -e, --exec-driver="native"             Force the docker runtime to use a specific exec driver     # 强制 docker 运行时使用指定执行驱动器
  --fixed-cidr=""                        IPv4 subnet for fixed IPs (ex: 10.20.0.0/16)
                                           this subnet must be nested in the bridge subnet (which is defined by -b or --bip)
  -G, --group="docker"                   Group to assign the unix socket specified by -H when running in daemon mode
                                           use '' (the empty string) to disable setting of a group
  -g, --graph="/var/lib/docker"          Path to use as the root of the docker runtime              # 容器运行的根目录路径
  -H, --host=[]                          The socket(s) to bind to in daemon mode                    # daemon 模式下 docker 指定绑定方式[tcp or 本地 socket]
                                           specified using one or more tcp://host:port, unix:///path/to/socket, fd://* or fd://socketfd.
  --icc=true                             Enable inter-container communication                       # 跨容器通信
  --insecure-registry=[]                 Enable insecure communication with specified registries (no certificate verification for HTTPS and enable HTTP fallback) (e.g., localhost:5000 or 10.20.0.0/16)
  --ip="0.0.0.0"                         Default IP address to use when binding container ports     # 指定监听地址，默认所有 ip
  --ip-forward=true                      Enable net.ipv4.ip_forward                                 # 开启转发
  --ip-masq=true                         Enable IP masquerading for bridge's IP range
  --iptables=true                        Enable Docker's addition of iptables rules                 # 添加对应 iptables 规则
  --mtu=0                                Set the containers network MTU                             # 设置网络 mtu
                                           if no value is provided: default to the default route MTU or 1500 if no default route is available
  -p, --pidfile="/var/run/docker.pid"    Path to use for daemon PID file                            # 指定 pid 文件位置
  --registry-mirror=[]                   Specify a preferred Docker registry mirror                  
  -s, --storage-driver=""                Force the docker runtime to use a specific storage driver  # 强制 docker 运行时使用指定存储驱动
  --selinux-enabled=false                Enable selinux support                                     # 开启 selinux 支持
  --storage-opt=[]                       Set storage driver options                                 # 设置存储驱动选项
  --tls=false                            Use TLS; implied by tls-verify flags                       # 开启 tls
  --tlscacert="/root/.docker/ca.pem"     Trust only remotes providing a certificate signed by the CA given here
  --tlscert="/root/.docker/cert.pem"     Path to TLS certificate file                               # tls 证书文件位置
  --tlskey="/root/.docker/key.pem"       Path to TLS key file                                       # tls key 文件位置
  --tlsverify=false                      Use TLS and verify the remote (daemon: verify client, client: verify daemon) # 使用 tls 并确认远程控制主机
  -v, --version=false                    Print version information and quit                         # 输出 docker 版本信息

```
##  命令列表 
```

$ sudo docker   # docker 命令帮助

Commands:
    attach    Attach to a running container                 # 当前 shell 下 attach 连接指定运行镜像
    build     Build an image from a Dockerfile              # 通过 Dockerfile 定制镜像
    commit    Create a new image from a container's changes # 提交当前容器为新的镜像
    cp        Copy files/folders from the containers filesystem to the host path
              # 从容器中拷贝指定文件或者目录到宿主机中
    create    Create a new container                        # 创建一个新的容器，同 run，但不启动容器
    diff      Inspect changes on a container's filesystem   # 查看 docker 容器变化
    events    Get real time events from the server          # 从 docker 服务获取容器实时事件
    exec      Run a command in an existing container        # 在已存在的容器上运行命令
    export    Stream the contents of a container as a tar archive   
              # 导出容器的内容流作为一个 tar 归档文件[对应 import ]
    history   Show the history of an image                  # 展示一个镜像形成历史
    images    List images                                   # 列出系统当前镜像
    import    Create a new filesystem image from the contents of a tarball  
              # 从tar包中的内容创建一个新的文件系统映像[对应 export]
    info      Display system-wide information               # 显示系统相关信息
    inspect   Return low-level information on a container   # 查看容器详细信息
    kill      Kill a running container                      # kill 指定 docker 容器
    load      Load an image from a tar archive              # 从一个 tar 包中加载一个镜像[对应 save]
    login     Register or Login to the docker registry server   
              # 注册或者登陆一个 docker 源服务器
    logout    Log out from a Docker registry server         # 从当前 Docker registry 退出
    logs      Fetch the logs of a container                 # 输出当前容器日志信息
    port      Lookup the public-facing port which is NAT-ed to PRIVATE_PORT
              # 查看映射端口对应的容器内部源端口
    pause     Pause all processes within a container        # 暂停容器
    ps        List containers                               # 列出容器列表
    pull      Pull an image or a repository from the docker registry server
              # 从docker镜像源服务器拉取指定镜像或者库镜像
    push      Push an image or a repository to the docker registry server
              # 推送指定镜像或者库镜像至docker源服务器
    restart   Restart a running container                   # 重启运行的容器
    rm        Remove one or more containers                 # 移除一个或者多个容器
    rmi       Remove one or more images                 
              # 移除一个或多个镜像[无容器使用该镜像才可删除，否则需删除相关容器才可继续或 -f 强制删除]
    run       Run a command in a new container
              # 创建一个新的容器并运行一个命令
    save      Save an image to a tar archive                # 保存一个镜像为一个 tar 包[对应 load]
    search    Search for an image on the Docker Hub         # 在 docker hub 中搜索镜像
    start     Start a stopped containers                    # 启动容器
    stop      Stop a running containers                     # 停止容器
    tag       Tag an image into a repository                # 给源中镜像打标签
    top       Lookup the running processes of a container   # 查看容器中运行的进程信息
    unpause   Unpause a paused container                    # 取消暂停容器
    version   Show the docker version information           # 查看 docker 版本号
    wait      Block until a container stops, then print its exit code   
              # 截取容器停止时的退出状态值
Run 'docker COMMAND --help' for more information on a command.

```

##  常用命令 

**1. 查看docker信息（version、info）**
```

# 查看docker版本
$docker version
# 显示docker系统的信息
$docker info

```

**2. 对image的操作（search、pull、images、rmi、history）**
```

# 检索image
$docker search image_name
# 下载image
$docker pull image_name
# 列出镜像列表; -a, --all=false Show all images; --no-trunc=false Don't truncate output; -q, --quiet=false Only show numeric IDs
$docker images
# 删除一个或者多个镜像; -f, --force=false Force; --no-prune=false Do not delete untagged parents
$docker rmi image_name
# 显示一个镜像的历史; --no-trunc=false Don't truncate output; -q, --quiet=false Only show numeric IDs
$docker history image_name

```

**3. 启动容器（run）**
docker容器可以理解为在沙盒中运行的进程。这个沙盒包含了该进程运行所必须的资源，包括文件系统、系统类库、shell 环境等等。但这个沙盒默认是不会运行任何程序的。**你需要在沙盒中运行一个进程来启动某一个容器。这个进程是该容器的唯一进程，所以当该进程结束的时候，容器也会完全的停止。**

```

# 在容器中运行"echo"命令，输出"hello word"
$docker run image_name echo "hello word"

# 交互式进入容器中
$docker run -i -t image_name /bin/bash

# 在容器中安装新的程序
$docker run image_name apt-get install -y app_name
Note：  在执行apt-get 命令的时候，要带上-y参数。如果不指定-y参数的话，apt-get命令会进入交互模式，需要用户输入命令来进行确认，但在docker环境中是无法响应这种交互的。apt-get 命令执行完毕之后，容器就会停止，但对容器的改动不会丢失。

```

**4. 查看容器（ps）**
```

# 列出当前所有正在运行的container
$docker ps
# 列出所有的container
$docker ps -a
# 列出最近一次启动的container
$docker ps -l

```
**5. 保存对容器的修改（commit）**
` 当你对某一个容器做了修改之后（通过在容器中运行某一个命令），需要把对容器的修改保存下来，这样下次可以从保存后的最新状态运行该容器。 `
# 保存对容器的修改; -a, --author="" Author; -m, --message="" Commit message
$docker commit ID new_image_name
Note：  image相当于类，container相当于实例，不过可以动态给实例安装新软件，然后把这个container用commit命令固化成一个image。

**6. 对容器的操作**（rm、stop、start、kill、logs、diff、top、cp、restart、attach）
```

# 删除所有容器
$docker rm `docker ps -a -q`

# 删除单个容器; -f, --force=false; -l, --link=false Remove the specified link and not the underlying container; -v, --volumes=false Remove the volumes associated to the container
$docker rm Name/ID

# 停止、启动、杀死一个容器
$docker stop Name/ID
$docker start Name/ID
$docker kill Name/ID

# 从一个容器中取日志; -f, --follow=false Follow log output; -t, --timestamps=false Show timestamps
$docker logs Name/ID

# 列出一个容器里面被改变的文件或者目录，list列表会显示出三种事件，A 增加的，D 删除的，C 被改变的
$docker diff Name/ID

# 显示一个运行的容器里面的进程信息
$docker top Name/ID

# 从容器里面拷贝文件/目录到本地一个路径
$docker cp Name:/container_path to_path
$docker cp ID:/container_path to_path

# 重启一个正在运行的容器; -t, --time=10 Number of seconds to try to stop for before killing the container, Default=10
$docker restart Name/ID

# 附加到一个运行的容器上面; --no-stdin=false Do not attach stdin; --sig-proxy=true Proxify all received signal to the process
$docker attach ID
Note： attach命令允许你查看或者影响一个运行的容器。你可以在同一时间attach同一个容器。你也可以从一个容器中脱离出来，是从CTRL-C。

```

**7. 保存和加载镜像（save、load）**

当需要把一台机器上的镜像**迁移**到另一台机器的时候，**需要保存镜像与加载镜像。**

```

# 保存镜像到一个tar包; -o, --output="" Write to an file
$docker save image_name -o file_path
# 加载一个tar包格式的镜像; -i, --input="" Read from a tar archive file
$docker load -i file_path

# 机器a
$docker save image_name > /home/save.tar
# 使用scp将save.tar拷到机器b上，然后：
$docker load < /home/save.tar

```
**8、 登录registry server（login）**
# 登陆registry server; -e, --email="" Email; -p, --password="" Password; -u, --username="" Username
$docker login
**9. 发布image（push）**
```

# 发布docker镜像
$docker push new_image_name

```
**10.  根据Dockerfile 构建出一个容器**
```

#build
      --no-cache=false Do not use cache when building the image
      -q, --quiet=false Suppress the verbose output generated by the containers
      --rm=true Remove intermediate containers after a successful build
      -t, --tag="" Repository name (and optionally a tag) to be applied to the resulting image in case of success
$docker build -t image_name Dockerfile_path

```
##  docker run命令 
```

$ sudo docker run --help

Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

  -a, --attach=[]            Attach to stdin, stdout or stderr.
  -c, --cpu-shares=0         CPU shares (relative weight)                       # 设置 cpu 使用权重
  --cap-add=[]               Add Linux capabilities
  --cap-drop=[]              Drop Linux capabilities
  --cidfile=""               Write the container ID to the file                 # 把容器 id 写入到指定文件
  --cpuset=""                CPUs in which to allow execution (0-3, 0,1)        # cpu 绑定
  -d, --detach=false  Detached mode: Run container in the background, print new container id # 后台运行容器
  --device=[]                Add a host device to the container (e.g. --device=/dev/sdc:/dev/xvdc)
  --dns=[]                   Set custom dns servers                             # 设置 dns
  --dns-search=[]            Set custom dns search domains                      # 设置 dns 域搜索
  -e, --env=[]               Set environment variables                          # 定义环境变量
  --entrypoint=""            Overwrite the default entrypoint of the image      # ？
  --env-file=[]              Read in a line delimited file of ENV variables     # 从指定文件读取变量值
  --expose=[]                Expose a port from the container without publishing it to your host    # 指定对外提供服务端口
  -h, --hostname=""          Container host name                                # 设置容器主机名
  -i, --interactive=false  Keep stdin open even if not attached               # 保持标准输出开启即使没有 attached
  --link=[]                  Add link to another container (name:alias)         # 添加链接到另外一个容器
  --lxc-conf=[]              (lxc exec-driver only) Add custom lxc options --lxc-conf="lxc.cgroup.cpuset.cpus = 0,1"
  -m, --memory=""            Memory limit (format: <number><optional unit>, where unit = b, k, m or g) # 内存限制
  --name=""                  Assign a name to the container                     # 设置容器名
  --net="bridge"             Set the Network mode for the container             # 设置容器网络模式
                               'bridge': creates a new network stack for the container on the docker bridge
                               'none': no networking for this container
                               'container:<name|id>': reuses another container network stack
                               'host': use the host network stack inside the container.  Note: the host mode gives the container full access to local system services such as D-bus and is therefore considered insecure.
  -P, --publish-all=false  Publish all exposed ports to the host interfaces   # 自动映射容器对外提供服务的端口
  -p, --publish=[]           Publish a container's port to the host # 指定端口映射
format: ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort
(use 'docker port' to see the actual mapping)
  --privileged=false  Give extended privileges to this container         # 提供更多的权限给容器
  --restart=""               Restart policy to apply when a container exits (no, on-failure[:max-retry], always)
  --rm=false  Automatically remove the container when it exits (incompatible with -d) # 如果容器退出自动移除和 -d 选项冲突
  --security-opt=[]          Security Options
  --sig-proxy=true  Proxify received signals to the process (even in non-tty mode). SIGCHLD is not proxied.
  -t, --tty=false  Allocate a pseudo-tty                              # 分配伪终端
  -u, --user=""              Username or UID                                    # 指定运行容器的用户 uid 或者用户名
  -v, --volume=[]            Bind mount a volume (e.g., from the host: -v /host:/container, from docker: -v /container)     
                             # 挂载卷
  --volumes-from=[]          Mount volumes from the specified container(s)      # 从指定容器挂载卷
  -w, --workdir=""           Working directory inside the container             # 指定容器工作目录

```
##  docker清理命令 
杀死所有正在运行的容器
docker kill $(docker ps -a -q)
 删除所有已经停止的容器
docker rm $(docker ps -a -q)
 删除所有未打 dangling 标签的镜像
docker rmi $(docker images -q -f dangling=true)
 删除所有镜像
docker rmi $(docker images -q)
 为这些命令创建别名
# ~/.bash_aliases
# 杀死所有正在运行的容器.
alias dockerkill='docker kill $(docker ps -a -q)'
# 删除所有已经停止的容器.
alias dockercleanc='docker rm $(docker ps -a -q)'
# 删除所有未打标签的镜像.
alias dockercleani='docker rmi $(docker images -q -f dangling=true)'
# 删除所有已经停止的容器和未打标签的镜像.
alias dockerclean='dockercleanc || true && dockercleani'
另附上docker常用命令
docker version #查看版本
 docker search tutorial#搜索可用docker镜像
 docker pull learn/tutorial #下载镜像
 docker run learn/tutorial echo "hello word"#在docker容器中运行hello world!
 docker run learn/tutorial apt-get install -y ping#在容器中安装新的程序
##  绑定容器端口到主机接口 
```

基本语法
$ sudo docker run -p [([<host_interface>:[host_port]])|(<host_port>):]<container_port>[/udp] <image> <cmd>

```
默认不指定绑定 ip 则监听所有网络接口。
**绑定 TCP 端口**
```

# Bind TCP port 8080 of the container to TCP port 80 on 127.0.0.1 of the host machine.
$ sudo docker run -p 127.0.0.1:80:8080 <image> <cmd>
# Bind TCP port 8080 of the container to a dynamically allocated TCP port on 127.0.0.1 of the host machine.
$ sudo docker run -p 127.0.0.1::8080 <image> <cmd>
# Bind TCP port 8080 of the container to TCP port 80 on all available interfaces of the host machine. 
$ sudo docker run -p 80:8080 <image> <cmd>  #80指的是主机端口，8080指的是容器端口
# Bind TCP port 8080 of the container to a dynamically allocated TCP port on all available interfaces
$ sudo docker run -p 8080 <image> <cmd> #绑定容器80端口到一个主机动态分配的端口

```
**绑定 UDP 端口**
```

# Bind UDP port 5353 of the container to UDP port 53 on 127.0.0.1 of the host machine.
$ sudo docker run -p 127.0.0.1:53:5353/udp <image> <cmd>

```

##  docker中的容器互联 
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
##  Docker后台与前台执行Detached vs foreground 
当我们启动一个container时，首先需要确定这个container是**运行在前台模式还是运行在后台模式。**
-d=false: Detached mode: Run container in the background, print new container id
Detached (-d)
**如果在docker run 后面追加-d=true或者-d，则containter将会运行在后台模式(Detached mode)。此时所有I/O数据只能通过网络资源或者共享卷组来进行交互。**因为container不再监听你执行docker run的这个终端命令行窗口。但你可以通过执行docker attach 来重新挂载这个container里面。需要注意的时， 如果你选择执行-d使container进入后台模式，那么将无法配合"--rm"参数。

Foregroud
如果在docker run后面没有追加-d参数，则container将默认进入前台模式(Foregroud mode)。Docker会启动这个container，同时将当前的命令行窗口挂载到container的标准输入，标准输出和标准错误中。也就是container中所有的输出，你都可以再当前窗口中查看到。甚至docker可以虚拟出一个TTY窗口，来执行信号中断。这一切都是可以配置的：
-a=[]          　　　　 : Attach to `STDIN`, `STDOUT` and/or `STDERR`
-t=false        　　  : Allocate a pseudo-tty
--sig-proxy=true　: Proxify all received signal to the process (non-TTY mode only)
-i=false        　　  : Keep STDIN open even if not attached
如果在执行run命令时没有指定-a，那么docker默认会挂载所有标准数据流，包括输入输出和错误。你可以特别指定挂载哪个标准流。
$ sudo docker run -a stdin -a stdout -i -t ubuntu /bin/bash (只挂载标准输入输出)
对于执行容器内的交互式操作，例如shell脚本。我们必须使用 -i -t来申请一个控制台同容器进行数据交互。但是当通过管道同容器进行交互时，就不能使用-t. 例如下面的命令
echo test | docker run -i busybox cat

##  如何确保docker在后台一直处于运行状态，然后我们还可以交互运行一些shell命令.？ 

通过SSH可以后台进程方式长期运行此镜像实例：docker run -d -p 22 -p 80:8080 learn/tutorial /usr/sbin/sshd -D
然后客户端远程连接这个ssh进行操作。

##  让nginx在后台运行，前台提供shell终端 
实现这一步的方法有许多种，比如
**5.1 手动运行**/usr/sbin/nginx -c /etc/nginx/nginx.conf
也就是用第4步的方法先启动到/bin/bash，再手动运行/usr/sbin/nginx -c /etc/nginx/nginx.conf或service nginx start，很容易想到，但太麻烦。

**5.2 通过Dockerfile来build**
将装好vim的容器提交成新的image，然后通过Dockerfile来自定义要启动哪些服务。关于Dockerfile后面我也会写文章来单独介绍其语法。在主机下运行
# docker commit -m "nginx 14.10 with bash,vim" nginx_bash_vim seanlook/nginx:bash_vim
a06ab41a6565f0dbd5d35d44cb441d1a166beaae3bc49bffcb09d334a1e77a5c

使用Dockerfile来建立一个新的镜像，加入启动到容器是运行的命令
```

# vi Dockerfile
FROM seanlook/nginx:bash_vim
ENTRYPOINT /usr/sbin/nginx -c /etc/nginx/nginx.conf && /bin/bash

build新image，tag为bash_vim_Df
# docker build -t seanlook/nginx:bash_vim_Df .
# docker images |grep 'bash_vim'
seanlook/nginx      bash_vim_Df       d9bbd13f5066       About an hour ago   125.9 MB
seanlook/nginx      bash_vim          aa8516fa0bb7       About an hour ago   125.9 MB

```

运行由Dockerfile创建的image
```

# docker run --name nginx_bash_vim_Df -v /tmp/docker:/usr/share/nginx/html:ro \
> -i -t -p 8080:80 \
> d9bbd13f5066   --> 或seanlook/nginx:bash_vim_Df

```
最后一条docker run之后就会自动进入bash终端，同时发现nginx服务也启动了，可以通过vim来编辑配置文件。

**5.3 修改容器的/etc/bash.bashrc**
这是投机取巧但不失为最简单的一种办法，见Run a service automatically in a docker container。启动刚安装完vim的那个容器（不必用run）
# docker ` start ` nginx_bash_vim
连接到终端上
# docker ` attach ` nginx_bash_vim
root@3911d1104c3f:/# vi /etc/bash.bashrc 
# added by mis_zx for auto-service nginx  --> 在最后加入
/usr/sbin/nginx -c /etc/nginx/nginx.conf
保存后直接Ctrl+D退出，**在start就可以访问了，如果要进入终端就attach，如果需要可以commit成一个镜像。**

5.4 **通过supervisor来管理docker容器的多个任务**
[在Docker里使用（支持镜像继承的）supervisor管理进程](http://air.googol.im/2014/03/28/supervisor-with-docker-to-manage-processes.html)
从上面的操作中可以看出，start是可以保留run启动时的参数如-v、-p，而commit之后如果没在Dockerfile中指定，下次启动依然需要带上目录、端口的映射参数。
另外提一点， docker run -i -t seanlook/nginx:bash_vim启动便会同时进入一个shell界面（但没有启动nginx），因为它的“前身”容器是在shell交互界面下run来的，但也没有保留-v、-p指定的映射关系。

参考：http://segmentfault.com/a/1190000000755980