title: cms环境部署文档 
createAt:2016-12-03 00:50:39
modifyAt:2016-12-03 00:50:39
location:dokuwiki\csswen\cms环境部署文档
author:xby@xbynet.net

#  CMS部署环境搭建手册 
##  基础软件与版本 
操作系统:centos7.0 x64
以下A服务器、B服务器只是一种假定情形，可为同一服务器也可为不同服务器，为了便于区分，我们以逻辑上的A、B服务器为例进行说明。
A服务器软件:
  * Nginx:1.10 - web代理服务器
  * Tomcat:7.0 - 应用服务器 x1
  * Mysql:5.6.31 -数据库服务器 - master
  * Memcache:1.4 - 缓存服务器
  * Rsync:3.x -文件同步客户端
  * Sersync:2.5 -文件推送
  * Supervisor 3.3.0 -进程管理
  * Jdk:1.7 -java环境
  * NFS - 网络磁盘共享

B服务器软件:
  * Nginx:1.10 - web代理服务器
  * Tomcat:7.0 - 应用服务器 x2
  * Mysql:5.6.31 -数据库服务器 - slave
  * Memcache:1.4 - 缓存服务器
  * Elasticsearch:1.x -检索服务器
  * Rsync:3.x -文件同步服务器
  * Supervisor 3.3.0 -进程管理
  * Jdk:1.7 -java环境
  * NFS - 网络磁盘共享

![](/data/dokuwiki/csswen/cms部署方案2.png|)

` 下面以方案一为例进行配置，方案二只要参考方案一的例子并结合部署方案图即可，过程类似。 `
##  NFS网络磁盘共享配置 
NFS 是Network File System的缩写，即网络文件系统。一种使用于分散式文件系统的协定，由Sun公司开发，于1984年向外公布。功能是通过网络让不同的机器、不同的操作系统能够彼此分享个别的数据，让应用程序在客户端通过网络访问位于服务器磁盘中的数据，是在类Unix系统间实现磁盘文件共享的一种方法。
NFS 的基本原则是“容许不同的客户端及服务端通过一组RPC分享相同的文件系统”，它是独立于操作系统，容许不同硬件及操作系统的系统共同进行文件的分享。
环境说明
nfs服务端系统：CentOS 7.0 x86_64
nfs服务端IP：10.12.10.90
nfs客户端系统：CentOS 7.0 x86_64
nfs客户端IP：10.12.10.49

----

**ServerB 安装NFS服务端**
Step-1：安装nfs-utils和rpcbind，运行以下命令
```

yum install -y nfs-utils rpcbind

```
上述命令将安装rpcbind服务和nfs服务。不过centos7默认已经安装好了，所以我们可以省略这一步。
Step-2：为NFS指定固定端口，(默认有些是动态端口导致被防火墙拦截)。
NFS 用到的服务有 portmapper nfs rquotad nlockmgr mountd
其中 portmapper nfs 服务端口是固定的分别是 111 2049
另外 rquotad nlockmgr mountd 服务端口是随机的。由于端口是随机的，这导致防火墙无法设置。
这时需要配置/etc/sysconfig/nfs 使 rquotad nlockmgr mountd 的端口固定。
运行以下命令：
```

vi /etc/sysconfig/nfs
RQUOTAD_PORT=875
LOCKD_TCPPORT=32803
LOCKD_UDPPORT=32769
MOUNTD_PORT=892

```
Step-3：开放防火墙中的上述端口，运行以下命令：
通过命令 rpcinfo -p 可查看nfs使用的端口
从centos7开始默认使用的是firewall作为防火墙，这里改为iptables防火墙。
```

1、关闭firewall：
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
2、安装iptables防火墙
yum install iptables-services #安装
vi /etc/sysconfig/iptables #编辑防火墙配置文件
systemctl restart iptables.service #最后重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动

```
创建脚本文件打开端口 vim enableport.sh
```

#!/bin/bash
iptables -I INPUT -s 10.12.10.0/24 -p tcp –dport 111 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp –dport 875 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp –dport 2049 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp –dport 32769 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp –dport 32803 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p tcp –dport 892 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp –dport 111 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp –dport 875 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp –dport 2049 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp –dport 32769 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp –dport 32803 -j ACCEPT
iptables -I INPUT -s 10.12.10.0/24 -p udp –dport 892 -j ACCEPT
iptables-save > /etc/sysconfig/iptables

```
保存之后```

sudo chmod +x enableport.sh
sudo ./enableport.sh

```
即可开启nfs相关端口。
Step-5：创建共享目录，运行以下命令：
```

mkdir -p /opt/webroot

```
Step-6：配置exports文件，运行以下命令：
```

sudo vim /etc/exports

```
在上述文件的末尾新增一行，如下所示：
```

/opt/webroot 10.12.10.0/24(rw,sync,no_root_squash)

```
这一行表示只有10.12.10.0/24网段的客户端能够以读写权限挂载共享目录
保存之后执行输出共享目录：
```

sudo exportfs –rv

```
Step-7：启动NFS相关服务，运行以下命令：
```

sudo systemctl enable nfs.service 
sudo systemctl enable rpcbind.service
sudo systemctl start rpcbind.service (注意顺序问题)
sudo systemctl start nfs.service

```
**通过showmount –e可以查看本机的共享目录
通过show mount –e IP地址 查看远程的共享目录**

----

**ServerA安装NFS客户端**
NFS客户端不需要启动NFS服务，但需要安装nfs-utils，运行以下命令：
```

yum install -y nfs-utils

```
不过centos7已经默认安装了。
手动挂载NFS共享目录
Step-1：确定挂载点，运行以下命令：
```

showmount -e 10.12.10.90

```
-e选项显示NFS服务端的导出列表。
Step-2：创建挂载目录，运行以下命令：
```

mkdir -p /opt/webroot

```
Step-3：挂载共享目录，运行以下命令：
```

mount -t nfs 10.12.10.90:/opt/webroot /opt/webroot

```
其中，-t选项用于指定文件系统的类型为nfs。
Step-4：共享目录使用结束之后，卸载共享目录，运行以下命令：
```

umount /opt/webroot

```
开机自动挂载
向fstab文件中添加共享目录的挂载条目，即可实现开机自动挂载，随后与NFS服务端的连接将始终处于活动状态。运行以下命令：
```

mkdir -p /opt/webroot
vim /etc/fstab

```
在上述文件末尾加入共享目录的挂载条目，如下所示：
```

10.12.10.90:/opt/webroot /opt/webroot nfs defaults 0 0

```
其中，第5个字段设置为0表示共享目录的文件系统不需要使用dump命令进行转储，第6个字段设置为0表示共享目录的文件系统不需要使用fsck命令进行检查。
4.查看是否挂载成功
```

df -h

```
5.卸载NFS挂载的共享目录
```

umount /opt/webroot

```

##  Java及tomcat安装 
使用winscp传到服务器
解压至/opt/下。
配置好环境变量。
设置可执行权限。

例如:JDK7.tar.gz
```

tar -zxvf JDK7.tar.gz
mv JDK7 /opt/jdk7

vim /etc/profile
export JAVA_HOME=/opt/jdk7
export PATH="$JAVA_HOME/bin:$PATH"

chmod 755 -R /opt/jdk7

```

##  elasticsearch安装 
ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎.

解压配置好环境变量即可.

防火墙配置:
```

sudo vim /etc/sysconfig/iptables
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 9200 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 9300 -j ACCEPT

```

```

sudo chmod 755 /opt/elasticsearch-rtf/bin/elasticsearch

```
启动初始化配置索引结构:
发送delete请求给http://10.12.10.90:9200/cms_index
发送PUT请求给,内容为http://10.12.10.90:9200/cms_index
```


 {
    "mappings": {
      "articles": {
		"dynamic": "false",
        "properties": {
          "_systemid": {
            "type": "string"
          },
          "articleAuthor": {
            "type": "string",
            "index": "not_analyzed"
          },
          "articleId": {
            "type": "string",
            "index": "not_analyzed"
          },
          "articlePublishPath": {
            "type": "string",
            "index": "not_analyzed"
          },
          "articlePublishTime": {
            "type": "string",
			      "index": "not_analyzed"
          },
          "articleContent": {
            "type": "string"
          },

          "articleTitle": {
            "type": "string"
          },
          "articleAbstract":{
            "type":"string",
            "index": "not_analyzed"
          },
          "genusSiteId":{
            "type":"string",
            "index": "not_analyzed"
          },
          "articleKeyword":{
            "type":"string",
            "index": "not_analyzed"
          },
          "articleCategory":{
            "type":"string",
            "index": "not_analyzed"
          }

        }
      }
    }
}

```
##  memcached安装 
memcache是一套分布式的高速缓存系统.
```

sudo yum install memcached
sudo systemctl enable memcached.service

```
##  安装与配置nginx 
Nginx ("engine x") 是一个高性能的HTTP和反向代理服务器.
1、添加软件源:
```

sudo vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1

```
2、更新软件缓存
```

sudo yum makecache

```
3、安装
```

sudo yum install nginx

```

###  ServerA配置,cms-manager端nginx配置 
cms-manager端，nginx配置文件如下:
```

sudo vim /etc/nginx/conf.d/default.conf

```
```

upstream myServer {   
        server 127.0.0.1:8080; 
    }

    server {
        listen       80;
        server_name  localhost;

        set $appdir "/opt/tomcat7/webapps/ROOT";
        set $publishdir "/opt/webroot";
      	client_max_body_size 100000m;
        location = / {
            #root   html;
            #index  index.html index.htm,index.php;
			proxy_set_header Host $host:$server_port;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header request_uri $request_uri;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://myServer;
        }
        location ^~ /sword {
			proxy_set_header Host $host:$server_port;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header request_uri $request_uri;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://myServer;
        }
		
		location ~ \.jsp$ {
			proxy_buffering off;
			proxy_set_header Host $host:$server_port;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header request_uri $request_uri;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_pass http://myServer;
		}
		
        location ~ \.(html|json)$ {
            if ($uri !~ ^/static) {
                root $publishdir;
                rewrite ^(.*)$ /$args$uri;
                error_page 405 =200 $uri;
                break;
                
            }
            root $appdir;
        }

        location ~ ^/.*?/static/{
            root $publishdir;
        }
        location  /static{
			root $appdir;           
        }

        location ^~ /resource {
            root $publishdir;
        }

    } 

```
###  ServerB配置,cms-web端nginx配置 
**ServerB配置,cms-web端nginx配置:**
```

sudo vim /etc/nginx/conf.d/default.conf

```
```

upstream webServer {
        server 127.0.0.1:8080;
        server 127.0.0.1:8081;
}

server {
        listen       80;
        server_name  localhost;
	
        set $publishdir "/opt/webroot";
  	client_max_body_size 100000m;
  
        location = / {
            #root   html;
            #index  index.html index.htm,index.php;
                        proxy_set_header Host $host:$server_port;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header request_uri $request_uri;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_pass http://webServer;
                        proxy_set_header   Cookie $http_cookie;
        }
        location ^~ /sword {
                        proxy_set_header Host $host:$server_port;
                        proxy_set_header X-Real-IP $remote_addr;
                        proxy_set_header request_uri $request_uri;
                        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                        proxy_pass http://webServer;
                        proxy_set_header   Cookie $http_cookie;
        }

        location ^~ /resource {
            root $publishdir;
        }

    }

```


##  rsync+sersync配置: 
rsync是类unix系统下的数据镜像备份工具——remote sync。一款快速增量备份工具 Remote Sync，远程同步 支持本地复制，或者与其他SSH、rsync主机同步。
sersync是基于Inotify开发的,可以记录下被监听目录中发生变化的（包括增加、删除、修改）具体某一个文件或某一个目录的名字，然后使用rsync同步的时候，只同步发生变化的这个文件或者这个目录。

**关闭SELINUX**
```

vi /etc/selinux/config #编辑防火墙配置文件
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq! #保存，退出
setenforce 0 #立即生效

```
###  ServerA(主服务器配置) 
1、下载inotify-tools :
```

wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz

```
编译安装:
```

sudo yum install gcc
sudo ./configure --prefix=/usr && sudo make && sudo su -c 'make install'

```
2、修改inotify默认参数（inotify默认内核参数值太小）
```

sysctl -w fs.inotify.max_queued_events="99999999"
sysctl -w fs.inotify.max_user_watches="99999999"
sysctl -w fs.inotify.max_user_instances="65535"

```

```

vi /etc/sysctl.conf #添加以下代码
fs.inotify.max_queued_events=99999999
fs.inotify.max_user_watches=99999999
fs.inotify.max_user_instances=65535

```
3、安装与配置sersync
```

cd ~
wget https://codeload.github.com/wsgzao/sersync/zip/master
mv master sersync.zip && unzip sersync.zip
cd sersync-master/
sudo tar -zxvf sersync2.5.4_64bit_binary_stable_final.tar.gz 
sudo mv GNU-Linux-x86 /opt/sersync
cd /opt/sersync

```
#配置下密码文件，因为这个密码是要访问服务器B需要的密码和上面服务器B的密码必须一致
```

echo "1234" > css.pass
#修改权限
chmod 600 css.pass
#修改confxml.conf
vim /opt/sersync/confxml.xml

```
```

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<head version="2.5">
    <host hostip="localhost" port="8008"></host>
    <debug start="false"/>
    <fileSystem xfs="false"/>
    <filter start="false">
        <exclude expression="(.*)\.svn"></exclude>
        <exclude expression="(.*)\.gz"></exclude>
        <exclude expression="^info/*"></exclude>
        <exclude expression="^static/*"></exclude>
    </filter>
    <inotify>
        <delete start="true"/>
        <createFolder start="true"/>
        <createFile start="false"/>
        <closeWrite start="true"/>
        <moveFrom start="true"/>
        <moveTo start="true"/>
        <attrib start="false"/>
        <modify start="false"/>
    </inotify>

    <sersync>
        <localpath watch="/opt/webroot/css">
            <remote ip="10.12.10.90" name="share2"/>
            <!--<remote ip="192.168.8.39" name="tongbu"/>-->
            <!--<remote ip="192.168.8.40" name="tongbu"/>-->
        </localpath>
        <rsync>
            <commonParams params="-artuz"/>
            <auth start="true" users="css" passwordfile="/opt/sersync/css.pass"/>
            <userDefinedPort start="true" port="873"/><!-- port=874 -->
            <timeout start="false" time="100"/><!-- timeout=100 -->
            <ssh start="false"/>
        </rsync>
        <failLog path="/tmp/rsync_fail_log.sh" timeToExecute="60"/><!--default every 60mins execute once-->
        <crontab start="true" schedule="30"><!--30mins-->
            <crontabfilter start="false">
                <exclude expression="*.php"></exclude>
                <exclude expression="info/*"></exclude>
            </crontabfilter>
        </crontab>
        <plugin start="false" name="command"/>
    </sersync>

    <plugin name="command">
        <param prefix="/bin/sh" suffix="" ignoreError="true"/>  <!--prefix /opt/tongbu/mmm.sh suffix-->
        <filter start="false">
            <include expression="(.*)\.php"/>
            <include expression="(.*)\.sh"/>
        </filter>
    </plugin>

</head>
```

```
###  ServerB配置(从服务器) 
开启防火墙tcp 873端口（Rsync默认端口）
```

vim /etc/sysconfig/iptables #编辑防火墙配置文件
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 873 -j ACCEPT
:wq! #保存退出
/etc/init.d/iptables restart #最后重启防火墙使配置生效

```
设置rsync开机自启:
```

sudo systemctl enable rsyncd.service

```
配置rsyncd.conf
```

sudo vim /etc/rsyncd.conf
max connections = 200
log file = /var/log/rsync.log
timeout = 600
[share2]
path = /opt/www/
comment = www share
uid = root
gid = root
read only = no
list = no
secrets file = /etc/css.pass
auth users = css #与ServerA中配置相对应

```
配置密码文件:#与ServerA中配置相对应
```

sudo vim /etc/css.pass #用户名:密码的形式
css:1234

```
#设置文件所有者读取、写入权限
```

chmod 600 /etc/rsyncd.conf  
chmod 600 /etc/rsync.pass 

``` 
加入开机自启:
```

sudo vim /etc/rc.local
/usr/bin/rsync --daemon

```
###  测试 
在ServerA执行:
主动使用rsync命令类似如下:
```

cd /opt/webroot/css
/usr/bin/rsync -artuz -R --delete ./  --port=873 css@10.12.10.90::share2 --password-file=/opt/sersync/css.pass

```
采用sersync自动同步类似如下:
```

cd /opt/sersync && sudo ./sersync2 -d -r -o confxml.xml

```
##  MySQL安装教程 
在centos 7上安装mysql ，需要卸载MariaDB，否则会冲突。
```

rpm -qa | grep mariadb
rpm -e –nodeps mariadb-libs-XXXXX.x86_64 （注意我这里的xxxx, 要根据上面命令出现的列表 )

```
yum源形式安装参考:http://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/
下载安装yum repo文件：
```

wget http://repo.mysql.com//mysql57-community-release-el7-8.noarch.rpm
rpm -ivh mysql57-community-release-el7-8.noarch.rpm
sudo vim /etc/yum.repos.d/mysql-community.repo #编辑配置安装版本为5.6,参考以上网址即可。

```
安装:
```

sudo yum -y install mysql-community-server

```
它会安装10个依赖包
(1/10): mysql-community-common-5.6.31-2.el7.x86_64.rpm                                                                                                                   
(2/10): mysql-community-libs-5.6.31-2.el7.x86_64.rpm                                                                                                                  
(3/10): perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64.rpm                                                                                              
(4/10): perl-Compress-Raw-Zlib-2.061-4.el7.x86_64.rpm                                                                                                     
(5/10): perl-Net-Daemon-0.48-5.el7.noarch.rpm                                                                                                                        
(6/10): perl-IO-Compress-2.061-2.el7.noarch.rpm                                                                                                                           
(7/10): perl-PlRPC-0.2020-14.el7.noarch.rpm                                                                                                            
(8/10): perl-DBI-1.627-4.el7.x86_64.rpm                                                                                                     
(9/10): mysql-community-client-5.6.31-2.el7.x86_64.rpm                                                                               
(10/10): mysql-community-server-5.6.31-2.el7.x86_64.rpm 

###  更改root密码: 
```

[root@rhel5 ~]# mysqld_safe --skip-grant-tables &
#####[admin@localhost ~]$ sudo mysql_secure_installation
[root@rhel5 ~]# mysql -u root
mysql> use mysql
mysql> update user set password = PASSWORD('1234') where user = 'root';
mysql> flush privileges;

```
###  默认字符编码设置: 
/etc/my.cnf最后加上编码配置
```

[mysql]
default-character-set =utf8

```
###  修改/var/lib/mysql权限: 
```

sudo chown mysql:mysql -R /var/lib/mysql

```

###  允许远程连接与访问: 
1、关闭firewall：
```

systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动

```
2、安装iptables防火墙
```

yum install iptables-services #安装
vim /etc/sysconfig/iptables #编辑防火墙配置文件
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 3306 -j ACCEPT
#### -A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
:wq! #保存退出

```
```

systemctl restart iptables.service #最后重启防火墙使配置生效
systemctl enable iptables.service #设置防火墙开机启动

```
**远程访问**
更改 “mysql” 数据库里的 “user” 表里的 “host” 项，从“localhost”改成“%”
```

[root@rhel5 ~]# mysqld_safe --skip-grant-tables &
[root@rhel5 ~]# mysql -u root
mysql>use mysql; 
mysql>update user set host = '%' where user = 'root' and host='localhost';
mysql>select host, user from user;
mysql>FLUSH PRIVILEGES;

``` 
**mysql远程登录失败**
修改mysql的配置文件/etc/mysql/my.conf，将[mysqld]下面的bind-address后面增加远程访问IP地址或者禁掉这句话就可以让远程机登陆访问了。
记得要重启mysql服务

##  MySQL主从配置(master-slave)： 
###  ServerA(master): 
```

mysql -u root -p
mysql> grant replication slave on *.* to 'slave1'@'10.12.10.90' identified by '1234'; //为slave服务器创建一个用户
mysql> flush privileges;
mysql> quit;
sudo systemctl restart mysqld.service

```
再重新登录查看master status
```

mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      409 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

```
记住上面这个表格。后面会用于slave服务器的配置

###  ServerB(slave)配置 
```

sudo vim /etc/my.cnf
server-id=200
log-bin=mysql-bin

```
```

sudo systemctl restart mysqld.service
mysql -u root -p
mysql> change master to master_host='10.12.10.49',master_port=3306,master_user='slave1',master_password='1234',master_log_file='mysql-bin.000003',master_log_pos=409;
#master_log_pos=154这个154不是随便填写的，它与master_log_file='mysql-bin.000001'一样，是根据上面再master mysql运行show master status得到的。
Mysql>start slave;    //启动从服务器复制功能
Mysql>show slave status\G;

```
![](/data/dokuwiki/csswen/pasted/20160620-144628.png)
红色框内两条状态都是：Yes，则表明配置成功。

##  多个MySQL实例在同一台机器上的主从配置 
1、创建一个新的MySQL datadir
```

mkdir -p /var/lib/mysql2
chmod --reference /var/lib/mysql /var/lib/mysql2
chown --reference /var/lib/mysql /var/lib/mysql2

```
2、在该数据目录下创建基础数据库
```

mysql_install_db --user=mysql --datadir=/var/lib/mysql2

```
3、单独对每一个mysql实例的相关操作
```

启动：mysqld_safe --defaults-file=/etc/my2.cnf &
连接不同数据库的方法：mysql -S  /var/lib/mysql/mysql2.sock -u username -p 或 mysql -h 127.0.0.1 -P 3307 -u root -p
关闭：mysqladmin -S /var/lib/mysql/mysql2.sock shutdown -p

```
4、配置mysqld_multi
配置样例可以通过mysqld_multi –example查看
对每个需要mysqld_multi管理的实例都配置一个同样的用户:
```

mysql> GRANT SHUTDOWN ON *.* TO 'multiadmin'@'localhost' IDENTIFIED BY '1234';
mysql> FLUSH PRIVILEGES;

```
上面对mysql-master,mysql-slave都得执行一遍。这个用户用来通过mysqld_multi来关闭mysqld进程。否则无法关闭。
```

vim /etc/my_multi.cnf

[mysqld_multi]
mysqld     = /usr/bin/mysqld_safe
mysqladmin = /usr/bin/mysqladmin
user       = multiadmin
pass   = 1234 #这里需要注意的是，网上流传的 password=1234是错误的写法

[mysqld1]
socket     = /var/lib/mysql/mysql.sock
port       = 3306
pid-file   = /var/run/mysqld/mysqld.pid
datadir    = /var/lib/mysql
user       = mysql
server-id  = 100
log-bin    = mysql-bin

[mysqld2]
socket     = /var/lib/mysql/mysql2.sock
port       = 3307
pid-file   = /var/run/mysqld/mysqld2.pid
datadir    = /var/lib/mysql2
user       = mysql
server-id  = 300
#skip-log-bin
relay-log-index = slave-relay-bin.index
relay-log = slave-relay-bin
#read_only = 1

[mysql]
default-character-set =utf8

```
启动查看关闭等命令：
```

mysqld_multi --defaults-extra-file=/etc/my_multi.cnf report 查看运行状态
mysqld_multi --defaults-extra-file=/etc/my_multi.cnf start
mysqld_multi --defaults-extra-file=/etc/my_multi.cnf stop
mysqld_multi --defaults-extra-file=/etc/my_multi.cnf start 2 （这个编号对应上面配置的mysqld2）
mysqld_multi --defaults-extra-file=/etc/my_multi.cnf stop 1

```

5、主从配置
上面有介绍，这里不再重复。
可能遇到的问题
主从连接正确，但是从库不拷贝，或者拷贝执行SQL报错（通过show slave status\G看到的）。遇到这种情形，直接先把主库的完整的sql导出拷贝到从库执行一遍。然后再重新配置一遍主从即可。


----

##  supervisor安装 
Supervisor是一个进程管理工具
可供参考文档:http://supervisord.org/installing.html
1、下载zip：https://pypi.python.org/pypi/supervisor
```

wget https://pypi.python.org/packages/44/80/d28047d120bfcc8158b4e41127706731ee6a3419c661e0a858fb0e7c4b2d/supervisor-3.3.0.tar.gz --no-check-certificate

```
解压安装:
```

python setup.py install

```
安装之后的路径为:
/usr/bin/supervisorctl
/usr/bin/supervisord

2、创建配置文件:
```

[root@localhost admin]# echo_supervisord_conf > /etc/supervisord.conf
sudo vim /etc/supervisord.conf
修改部分:
[supervisord]
logfile=/var/log/supervisor/supervisord.log
childlogdir=/var/log/supervisor
;取消下段注释，并修改：
[inet_http_server]         ; inet (TCP) server disabled by default
port=*:9010        ; (ip_address:port specifier, *:port for all iface)
username=css              ; (default is no username (open server))
password=1234               ; (default is no password (open server))

[include]
files = /etc/supervisor/conf.d/*.conf

```
创建日志
```

sudo mkdir /var/log/supervisor
sudo touch /var/log/supervisor/supervisord.log

```
添加开机启动:
```

sudo vim /etc/rc.local
/bin/supervisord
/usr/bin/supervisorctl start all

```
防火墙配置:
```

sudo vim /etc/sysconfig/iptables
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 9010 -j ACCEPT

```


###  ServerA端配置 
```

sudo mkdir -p /etc/supervisor/conf.d
sudo vim /etc/supervisor/conf.d/default.conf
[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
stopsignal=QUIT
priority=20
user=root
[program:memcached]
command=/usr/bin/memcached -u memcached -p 11211 -m 64 -c 1024
user=memcached
priority=30
[program:mysql]
command=/usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mysqld.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/lib/mysql/mysql.sock
user=mysql
priority=50
[program:tomcat]
command=/opt/tomcat7/bin/catalina.sh run
user=root
priority=90
[program:sersync]
command=/opt/sersync/sersync2  -r -o /opt/sersync/confxml.xml
directory=/opt/sersync
user=root
priority=100

```
###  ServerB端配置 
```

sudo mkdir -p /etc/supervisor/conf.d
sudo vim /etc/supervisor/conf.d/default.conf

```
```

[program:nginx]
command=/usr/sbin/nginx -g "daemon off;"
stopsignal=QUIT
priority=10
user=root
[program:memcached]
command=/usr/bin/memcached -u memcached -p 11211 -m 64 -c 1024
priority=10
user=memcached
[program:mysql]
command=/usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mysqld.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/lib/mysql/mysql.sock
priority=30
user=mysql
[program:tomcat1]
command=/opt/tomcat1/bin/catalina.sh run
priority=100
user=root
[program:tomcat2]
command=/opt/tomcat2/bin/catalina.sh run
priority=100
user=root
[program:elasticsearch]
command=/opt/elasticsearch-rtf/bin/elasticsearch
priority=200
user=root
[program:sitesync]
command=/opt/jdk7/bin/java -jar /home/admin/sitesync/cms-site-sync.jar /home/admin/sitesync/config.properties
priority=300
user=root

```

访问10.12.10.90:9010看到的supervisor监控界面截图如下:
![](/data/dokuwiki/csswen/pasted/20160621-145637.png)


##  客户网站服务器(示例) 
1、nginx配置示例：
```

    upstream myServer {   
        server 10.12.10.90; 
    }

    server {
        listen       80;
        server_name  www.youtdoamin.com;

        set $appdir "E:/workspace/eclipse/.metadata/.plugins/org.eclipse.wst.server.core/tmp1/wtpwebapps/cms-manager";
        set $publishdir "/opt/www";
		
		location  /{
			root $appdir;           
        }
        location ^~ /sword {
			proxy_set_header Host $host:$server_port;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header request_uri $request_uri;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://myServer;
        }		

        location ^~ /resource {
            proxy_pass http://myServer;
        }

    }    

```
2、cms-sitesync配置部署运行。

` 客户网站服务器配置流程： `
  * 安装并配置rsyncd服务软件
  * 安装并配置Nginx web server
  * 安装并配置cms-sitesync
  * 安装并配置supervisor
##  站点推送与校验配置说明 
登录管理界面可以看到，站点推送需要的信息如下：
![](/data/dokuwiki/csswen/pasted/20160622-110041.png)
  * 站点ID是自动生成的，这个ID需要拷贝到客户网站服务器主机上进行cms-sitesync配置。
  * 远程地址、端口是指客户网站服务器主机地址、以及rsyncd的端口（默认873）。
  * 远程登录用户名，不是指客户机系统登录用户，而是指客户机上运行的rsyncd服务配置的用户名。
  * 站点共享名，指的是客户机系统上配置的rsyncd服务配置的站点共享名。
  * 站点同步名，保留特性。
  * 密码文件路径，指的是运行cms-web的服务器上配置的对应该站点客户机的rsync密码文件。

` 新增站点流程： `
1、配置客户网站服务器主机中rsyncd用户，站点共享名，端口。
2、配置cms-web服务器主机中对应该客户机网站的rsync密码文件。
3、登录管理端新建一个站点，并设置授权。然后配置站点推送信息。记录站点ID
4、将上述步骤记录的站点ID用于配置客户网站服务器主机的cms-sitesyncp配置文件。


##  防火墙配置 
以iptables防火墙为例：
###  方案1 
**内网防火墙配置：**
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT   #SSH登录端口
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT #MySQL主库端口
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3307 -j ACCEPT #MySQL从库端口
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9010 -j ACCEPT #进程监控管理端口
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT #内网访问端口

----

**DMZ防火墙配置:**
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 22 -j ACCEPT  #SSH登录端口
#NFS配置
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 892 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 32803 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 32769 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 2049 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 875 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 111 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 892 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 32803 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 32769 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 2049 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 875 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 111 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 892 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 32803 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 32769 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 2049 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 875 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 111 -j ACCEPT
#检索相关
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 9200 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 9300 -j ACCEPT
#进程管理
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 9010 -j ACCEPT
#外网访问端口
-A INPUT -m state --state NEW  -p tcp -m tcp --dport 80 -j ACCEPT

----
###  方案2 
**内网防火墙配置：**
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3307 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9010 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9200 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9300 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 8081 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 8082 -j ACCEPT

----

**DMZ防火墙配置:**
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 22 -j ACCEPT  #SSH登录端口
#NFS配置
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 892 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 32803 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 32769 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 2049 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 875 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 111 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 892 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 32803 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 32769 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 2049 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 875 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p udp -m udp --dport 111 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 892 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 32803 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 32769 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 2049 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 875 -j ACCEPT
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 111 -j ACCEPT
#进程管理
-A INPUT -s 10.12.10.0/24 -p tcp -m tcp --dport 9010 -j ACCEPT
#外网访问端口
-A INPUT -m state --state NEW  -p tcp -m tcp --dport 80 -j ACCEPT

##  附录 
###  附录1-mysql重装相关说明 
/usr/share/mysql（mysql.server命令及配置文件）
/etc/my.cnf配置文件
相关命令/usr/bin（mysqladmin mysqldump等命令）
启动脚本/etc/rc.d/init.d/（启动脚本文件mysql的目录）
数据库目录/var/lib/mysql/

卸载mysql
1、查找以前是否装有mysql
命令：rpm -qa|grep -i mysql
可以看到mysql的包：
2、删除mysql
删除命令：rpm -e --nodeps 包名
3、删除老版本mysql的开发头文件和库
命令：rm -fr /usr/lib/mysql
rm -fr /usr/include/mysql
注意：卸载后/var/lib/mysql中的数据及/etc/my.cnf不会删除，如果确定没用后就手工删除
rm -f /etc/my.cnf
rm -fr /var/lib/mysql


###  附录2-supervisorctl进程管理指令 
```

supervisorctl start your-pragramname
supervisorctl stop your-pragramname
supervisorctl start all
supervisorctl stop all

```
