title: docker时区设置 

#  Docker Ubuntu时区设置 
##  方式一、通过Dockerfile设置(推荐) 
http://serverfault.com/questions/683605/docker-container-time-timezone-will-not-reflect-changes
The secret here is that ` dpkg-reconfigure tzdata ` simply creates ` /etc/localtime ` as a copy, hardlink or symlink (a symlink is preferred) to a file in ` /usr/share/zoneinfo `. So it is possible to do this entirely from your`  Dockerfile `. Consider:
```

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

```
And as a bonus, TZ will be set correctly in the container as well.
This is also distribution-agnostic, so it works with pretty much anything Linux.

或者也可以直接这样：（未测试）
```

# Set the timezone.
RUN sudo echo "America/New_York" > /etc/timezone
RUN sudo dpkg-reconfigure -f noninteractive tzdata

```
更多参考：
https://github.com/docker/docker/issues/12084
http://tedwise.com/2015/05/02/setting-the-timezone-in-a-docker-image
##  方式二、运行容器修改 
https://github.com/ma6174/blog/issues/9
docker container默认是UTC时间的，如果要使用北京时间需要修改时区，有以下几种方式：
**1. 使用` dpkg-reconfigure `命令**
```

$ docker run -i -t ubuntu bash
root@1f8ccb4c3dc1:/# date
Wed Sep 10 16:02:38 UTC 2014
root@1f8ccb4c3dc1:/# dpkg-reconfigure tzdata

Current default time zone: 'Asia/Shanghai'
Local time is now:      Thu Sep 11 00:02:50 CST 2014.
Universal Time is now:  Wed Sep 10 16:02:50 UTC 2014.

root@1f8ccb4c3dc1:/# date
Thu Sep 11 00:02:52 CST 2014

```
**2. 挂载使用系统` /etc/localtime `**
```

$ docker run -i -t -v /etc/localtime:/etc/localtime ubuntu bash
root@6213cd50d722:/# date
Thu Sep 11 00:08:56 CST 2014
root@6213cd50d722:/#
旧版本的dockere可能文件是/etc/localtime:ro，需要修改挂载方式为：-v /etc/localtime:/etc/localtime:ro

```

**3. 直接覆盖` /etc/localtime `**
```

$ docker run -i -t ubuntu bash
root@d4b926fb2c75:/# date
Wed Sep 10 16:11:15 UTC 2014
root@d4b926fb2c75:/# apt-get install -y wget > /dev/null 2>&1
root@d4b926fb2c75:/# wget http://ma6174.u.qiniudn.com/localtime -O /etc/localtime -o /dev/null
root@d4b926fb2c75:/# date
Thu Sep 11 00:13:05 CST 2014
root@d4b926fb2c75:/#

```