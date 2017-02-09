title: docker学习笔记三 

#  docker学习笔记之数据共享 
docker管理数据的方式有两种：
  * 数据卷
  * 数据卷容器
##  8.1 数据卷 
数据卷是一个或多个容器专门指定绕过 Union File System 的目录，**为持续性或共享数据提供一些有用的功能：**
  * 数据卷可以在容器间共享和重用
  * 数据卷数据改变是直接修改的
  * 数据卷数据改变不会被包括在容器中
  * 数据卷是持续性的，直到没有容器使用它们
  * 
**添加一个数据卷**
你可以使用`  -v ` 选项添加一个数据卷，或者可以使用多次 -v 选项为一个 docker 容器运行挂载多个数据卷。

```

$ sudo docker run --name data -v /data -t -i ubuntu:14.04 /bin/bash
# 创建数据卷绑定到到新建容器，新建容器中会创建 /data 数据卷
bash-4.1# ls -ld /data/
drwxr-xr-x 2 root root 4096 Jul 23 06:59 /data/
bash-4.1# df -Th
Filesystem    Type    Size  Used Avail Use% Mounted on
... ...
              ext4     91G  4.6G   82G   6% /data
创建的数据卷可以通过 docker inspect 获取宿主机对应路径

$ sudo docker inspect data
... ...
    "Volumes": {
        "/data": "/var/lib/docker/vfs/dir/151de401d268226f96d824fdf444e77a4500aed74c495de5980c807a2ffb7ea9"
    }, # 可以看到创建的数据卷宿主机路径
... ...

```
或者直接指定获取
```

$ sudo docker inspect --format="![](/data/dokuwiki .Volumes )" data
map[/data: /var/lib/docker/vfs/dir/151de401d268226f96d824fdf444e77a4500aed74c495de5980c807a2ffb7ea9]

```

**挂载宿主机目录为一个数据卷**
-v 选项除了可以创建卷，也可以挂载当前主机的一个目录到容器中。
```

$ sudo docker run --name web -v /source/:/web -t -i ubuntu:14.04 /bin/bash
bash-4.1# ls -ld /web/
drwxr-xr-x 2 root root 4096 Jul 23 06:59 /web/
bash-4.1# df -Th
... ...
              ext4     91G  4.6G   82G   6% /web
bash-4.1# exit

```
**默认挂载卷是可读写的，可以在挂载时指定只读**
```

$ sudo docker run --rm --name test -v /source/:/test:ro -t -i ubuntu:14.04 /bin/bash
可以通过`  --name ` 选项给容器自定义命名

```


##  8.2 创建和挂载一个数据卷容器 
**如果你有一些持久性的数据并且想在容器间共享**，或者想用在非持久性的容器上，**最好的方法是创建一个数据卷容器，然后从此容器上挂载数据。**
**创建数据卷容器**
```

$ sudo docker run -d -v /test --name test ubuntu:14.04 echo hello

```
使用 ` --volumes-from ` 选项在另一个容器中挂载 /test 卷。**不管 test 容器是否运行，其它容器都可以挂载该容器数据卷，当然如果只是单独的数据卷是没必要运行容器的。**
```

$ sudo docker run -d --volumes-from dbdata --name db1 training/postgres
$ sudo docker run -d --volumes-from dbdata --name db2 training/postgres

```
#还可以使用多个--volumes-from 参数来从多个容器挂载多个数据卷
#也可以从其他已经挂载了容器卷的容器来挂载数据卷
```

$ sudo docker run -d --name db3 --volumes-from db1 training/postgres

```
如果你移除了挂载的容器，包括初始容器，或者后来的db1 db2，这些卷在有容器使用它的时候不会被删除。这可以让我们在容器之间升级和移动数据。




##  8.3 备份、恢复或迁移数据卷 
**备份**
$ sudo docker run --rm --volumes-from test -v $(pwd):/backup ubuntu:14.04 tar cvf /backup/test.tar /test
tar: Removing leading `/' from member names
/test/
/test/b
/test/d
/test/c
/test/a
启动一个新的容器并且从 test 容器中挂载卷，然后挂载当前目录到容器中为 backup，并备份 test 卷中所有的数据为 test.tar，执行完成之后删除容器 --rm，此时备份就在当前的目录下，名为 test.tar。
$ ls        # 宿主机当前目录下产生了 test 卷的备份文件 test.tar
test.tar

**恢复**
你可以恢复给同一个容器或者另外的容器，新建容器并解压备份文件到新的容器数据卷

$ sudo docker run -t -i -d -v /test --name test4 ubuntu:14.04  /bin/bash
$ sudo docker run --rm --volumes-from test4 -v $(pwd):/backup ubuntu:14.04 tar xvf /backup/test.tar -C /
# 恢复之前的文件到新建卷中，执行完后自动删除容器
test/
test/b
test/d
test/c
test/a
##  8.4 删除 Volumes 
Volume 只有在下列情况下才能被删除：
docker rm -v 删除容器时添加了 -v 选项
docker run --rm 运行容器时添加了 --rm 选项
否则，会在 /var/lib/docker/vfs/dir 目录中遗留很多不明目录。
