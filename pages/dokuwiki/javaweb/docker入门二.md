title: docker入门二 

#  docker入门二 
确保Docker已经安装运行就绪：
sudo docker info
运行:
docker run -it ubuntu /bin/bash
  * -i代表开启标准输入STDIN
  * -t代表分派一个伪tty终端
连起来就是提供一个交互式shell

使用第一个容器：
#hostname
# cat /etc/hosts
# ip a
# ps -aux

` /var/lib/docker `存放容器的镜像，容器，以及容器的配置
所有容器都保存在` /var/lib/docker/containers `下。
##  容器命名 
```

--name标志
sudo docker run --name my_container -it ubuntu /bin/bash

```
##  启动相关 
docker start abcfdd0e3421
docker restart my_container
重附到容器:sudo docker attach my_container
docker stop/kill my_container

##  自动重启容器 
```

--restart标志会通过检查容器的退出代码来决定是否需要自动重新启动该容器。
--restart=always不会退出码是否为0都重启
--restart=on-failure:退出码为非0时候重启
--restart=on-failure:5退出码为非0时候重启，但是最多重启5次。

```

**创建守护式容器**
docker run --name my_container -d ubuntu /bin/sshd
##  查看容器 
docker ps  查看正在运行的容器
docker ps -a 查看所有容器
docker ps -l 查看最后运行的容器

##  操作运行中的容器 
  * 查看日志：docker logs -ft my_container
  * 查看容器内进程：docker top my_container

在容器内运行进程：通过` docker exec `命令启动额外新进程。
  * 1、运行后台任务：docker exec -d my_container touch /etc/haha
  * 2、运行交互式命令:docker exec -it my_container /bin/bash
##  查看容器更多数据 
```

docker inspect my_container返回名称，配置信息，命令，网络配置等等
docker inspect  --format='![](/data/dokuwiki .State.Running )' my_container 返回运行状态，如false
docker inspect  --format='![](/data/dokuwiki .NetworkSettings.IPAddress  )' my_container 返回IP地址，如172.17.0.2
docker inspect  --format='![](/data/dokuwiki .Name  ) ![](/data/dokuwiki.State.Running)' my_container my_container2 返回多个容器的状态

```
##  删除容器 
批量删除 docker rm `docker ps -aq`
注意：正在运行的容器是不能被删除的。


##  Docker镜像和仓库 
镜像分层
基础镜像》父镜像》a镜像
只读层》可写层

列出镜像
docker images
docker images ubuntu

所有镜像都是基于基础镜像构建
Docker基础镜像如ubuntu并不是一个完整的操作系统，而是一个极度精简版。
仓库、标签、镜像

拉取镜像：
docker pull ubuntu

docker run -it --name ubuntu1 ubuntu /bin/bash

查找镜像:
docker search ubuntu

构建镜像：
  * 1、docker commit
  * 2、docker build和Dockerfile文件

docker commit -m="msg" --author="xby" 4223eed nginximg
基于Dockerfile: docker build -t="xby/web" .

删除镜像：
docker rmi nginximg
docker rmi `docker images -aq`

docker私有仓库Registry
docker run -p 5000:5000 registry

推送至私有库：1、先为本地的镜像打上远程私有库的标签 docker tag 242dfdrwe example.com:5000/nginximg
2、推送: docker push example.com:5000/nginximg
