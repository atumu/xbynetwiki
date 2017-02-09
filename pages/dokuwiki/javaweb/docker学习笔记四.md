title: docker学习笔记四 

#  docker学习笔记之构建私有库 
构建私有库
Docker 官方提供了 ` docker registry ` 的构建方法 ` docker-registry `
##  10.1 快速构建 
快速构建 docker registry 通过以下两步:
安装 docker
运行 registry: ```

docker run -p 5000:5000 -v /opt/registry:/tmp/registry registry

```
这种方法通过 Docker hub 使用官方镜像 official image from the Docker hub
##  10.2 不使用容器构建 registry 
安装必要的软件
```

$ sudo apt-get install build-essential python-dev libevent-dev python-pip liblzma-dev

```
配置 docker-registry
```

sudo pip install docker-registry

```
或者 使用 github clone 手动安装
```

$ git clone https://github.com/dotcloud/docker-registry.git
$ cd docker-registry/
$ cp config/config_sample.yml config/config.yml
$ mkdir /data/registry -p
$ pip install .

```

**运行**
```

docker-registry

```

##  10.3 提交指定容器到私有库 
docker server的配置文件/etc/init/docker.conf

创建好自己的私有仓库之后，可以使用**docker tag 一个镜像，然后push，然后在别的机器上pull下来就好了**。这样我们的局域网私有docker仓库就搭建好了
```

先标记要上传哪个镜像
$ docker tag ubuntu:12.04 私有库IP:5000/ubuntu:12.04
接下来把打了tag的镜像上传到私有仓库。
$ docker push 私有库IP:5000/ubuntu

```
默认情况下，会将仓库存放于容器内的` /tmp/registry `目录下，这样如果容器被删除，则存放于容器中的镜像也会丢失，**所以我们一般情况下会指定本地一个目录挂载到容器内的/tmp/registry下**，如下：
使用 docker run -p 5000:5000`  -v /opt/registry:/tmp/registry ` registry 在局域网的一台机器上开启一个容器之后，我的局域网私有仓库地址为192.168.7.26:5000
###  关于私有库提交https的问题 
从docker1.3.2版本开始默认docker registry使用的是https，当你用docker pull 非https的docker regsitry的时候会报下面错误：
如下问题：
```

root@gerryyang:~# docker push 104.131.173.242:5000/ubuntu_sshd_gcc_gerry:14.04  
FATA[0002] Error: Invalid registry endpoint https://104.131.173.242:5000/v1/: Get https://104.131.173.242:5000/v1/_ping: EOF. If this private registry supports only HTTP or HTTPS with an unknown CA certificate, please add `--insecure-registry 104.131.173.242:5000` to the daemon's arguments. In the case of HTTPS, if you have access to the registry's CA certificate, no need for the flag; simply place the CA certificate at /etc/docker/certs.d/104.131.173.242:5000/ca.crt  

```
解决方法： 参考：http://www.aixchina.net/Question/137833?p=2 http://wiselyman.iteye.com/blog/2166669
**cnetos下：**
` vim /etc/sysconfig/docker `:修改为如下 注意参数key,value之间没有=号。
```

other_args=" --insecure-registry 10.12.4.52:5000"

```
此时重启docker server发现并没有生效。
还需要修改` vim  /lib/systemd/system/docker.service `:
要加` EnvironmentFile=-/etc/sysconfig/docker `和编辑ExecStart=/usr/bin/docker -d ` $other_args ` -H fd:/ /
如下：
```

[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
EnvironmentFile=-/etc/sysconfig/docker
ExecStart=/usr/bin/docker -d $other_args -H fd://
MountFlags=slave
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity

[Install]
WantedBy=multi-user.target

然后重启docker:重启Docker
sudo systemctl daemon-reload
sudo service docker restart

```


先在本机看下现有的images
```

WaitFish ~ # docker images
REPOSITORY TAG IMAGE ID CREATED VIRTUAL
SIZE
ubuntu latest ba5877dc9bec 6 weeks ago 192.7 MB
ubuntu 14.04 ba5877dc9bec 6 weeks ago 192.7 MB
使用 docker tag 将ba58这个image标记为192.168.7.26:5000/test
WaitFish ~ # docker tag ba58 -t 192.168.7.26:5000/test
WaitFish ~ # docker images
REPOSITORY TAG IMAGE ID CREATED VIRTUAL
SIZE
ubuntu 14.04 ba5877dc9bec 6 weeks ago 192.7 MB
ubuntu latest ba5877dc9bec 6 weeks ago 192.7 MB
192.168.7.26:5000/test latest ba5877dc9bec 6 weeks ago 192.7 MB
使用docker push 上传我们标记的新image，这里因为我的服务器上已经有这个images，所有在上传文
件层的时候，都跳过了，但是标记还是不一样的。
WaitFish ~ # docker push 192.168.7.26:5000/test

```
宿主主机my_registry的目录结构
root@gerryyang:~/my_registry# tree  
私有仓库查询方法
curl http://104.131.173.242:5000/v1/search
说明：使用curl查看仓库104.131.173.242:5000中的镜像。在结果中可以查看到ubuntu_sshd_gcc_gerry，说明已经上传成功了。
参考：http://blog.csdn.net/delphiwcdj/article/details/43099877
http://blog.csdn.net/wangtaoking1/article/details/44180901