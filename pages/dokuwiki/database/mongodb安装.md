title: mongodb安装 

#  mongodb安装 
##  ubuntu安装mongodb 
参考：https://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/
```

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list

sudo apt-get update
sudo apt-get install -y mongodb-org 

/usr/bin/mongod --dbpath /opt/mongodbdata(注：service mongod start无效，无法启动。估计是个BUG)

```
运行时用户mongodb
数据目录/var/lib/mongodb 日志目录/var/log/mongodb  See systemLog.path and storage.dbPath for additional information.
注意：修改数据目录之后需要修改目录权限chown mongodb:mongodb -R /opt/mongodbdata && chmod 755 /opt/mongodbdata


配置文件/etc/mongod.conf 注释 bind_ip = 127.0.0.1 ，默认端口port为27017

**Uninstall MongoDB**
```

sudo apt-get purge mongodb-org*
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb

```


##  MongoDB安装为Windows服务方法 
http://blog.csdn.net/chaijunkun/article/details/7227967
http://www.mongodb.org/downloads
D:\MongoDB目录作为数据文件存放目录，于是建立目录
安装服务：
```

D:\mongodb-win32-i386-2.0.2\bin>mongod --install --serviceName MongoDB --serviceDisplayName MongoDB --logpath c:\MongoDB.Log --dbpath c:\MongoDB --directoryperdb  

```
这里简单介绍一下使用的参数及其含义：
--install：安装MongoDB服务
--serviceName：安装Windows服务时使用的服务名
--serviceDisplayName：在Windows服务管理器中显示的服务名，如下所示：
MongoDB服务显示名
--logpath：MongoDB日志输出文件名称。虽说该参数直译是“日志路径”，其实要指定的是一个具体的完整文件名。这里我使用的是C盘根目录下的MongoDB.Log文件。该文件不用事先创建，直接指定就是了。
--dbpath：指定MongoDB数据存放的路径。这个就是最关键的参数了，不仅该目录要存在，并且最好不要以“\”结尾。
--directoryperdb：这个参数很好理解，让MongoDB按照数据库的不同，针对每一个数据库都建立一个目录，所谓的“目录每数据库”
好了，执行了上面的命令后，服务就可以成功注册了。