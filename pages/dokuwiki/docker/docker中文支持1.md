title: docker中文支持1 

#  Docker Ubuntu中文支持 
http://stackoverflow.com/questions/28405902/how-to-set-the-locale-inside-a-docker-container
https://github.com/abevoelker/docker-ubuntu-locale/blob/master/Dockerfile
http://blog.shiqichan.com/Input-Chinese-character-in-docker-bash/

**新建Dockerfile：**
` vim Dockerfile `输入以下内容:
```

FROM ubuntu:trusty

# Ensure UTF-8 locale
#COPY locale /etc/default/locale
RUN locale-gen zh_CN.UTF-8 &&\
  DEBIAN_FRONTEND=noninteractive dpkg-reconfigure locales
RUN locale-gen zh_CN.UTF-8  
ENV LANG zh_CN.UTF-8  
ENV LANGUAGE zh_CN:zh  
ENV LC_ALL zh_CN.UTF-8    

``` 
然后执行构建
```

docker build -t xbyubuntu .

```