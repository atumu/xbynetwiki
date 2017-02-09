title: cms环境部署文档2 

#  CMS环境离线安装版 
` 注意：以下所有涉及下载的操作只是用于环境准备操作。正式安装时需提前下载好这些安装包。 `
操作系统:CentOS7 x64

##  使用yum仅下载rpm及其依赖 
yum命令本身就可以用来下载一个RPM包，标准的yum命令提供了-downloadonly(只下载)的选项来达到这个目的。
```

$ sudo yum install --downloadonly <package-name>

```
默认情况下，一个下载的RPM包会保存在下面的目录中:
/var/cache/yum/x86_64/[centos/fedora-version]/[repository]/packages
以上的[repository]表示下载包的来源仓库的名称(例如：base、fedora、updates)
如果你想要将一个包下载到一个指定的目录(如/tmp)：
```

$ sudo yum install --downloadonly --downloaddir=/tmp <package-name>

```
` 注意，如果下载的包包含了任何没有满足的依赖关系，yum将会把所有的依赖关系包下载，但是都不会被安装。 `
另外一个重要的事情是，**在CentOS/RHEL 6或更早期的版本中，你需要安装一个单独yum插件(名称为 yum-plugin-downloadonly)才能使用–downloadonly命令选项**：
```

$ sudo yum install yum-plugin-downloadonly

```
如果没有该插件，你会在使用yum时得到以下错误：
Command line error: no such option: –downloadonly
补充：**RHEL7/CentOS7 使用yum安装的过程中有 [y/d/n] 的选项，按d就是下载了**。


----
虽然上面的便利方法，但是不能满足所有条件。
安装包及依赖下载还需要分为以下几种情况：(甚至更多)

1、对于官方yum源参考包含的软件及依赖: 
以gcc为例:
```

$ sudo yum install --downloadonly --downloaddir=/home/admin/packages/gcc gcc

```

2、对于相Nginx,MySQL之类的需要第三方源的软件及依赖，以Nginx，MySQL为例：
2.1、对于Nginx：
```

$ sudo vim /etc/yum.repos.d/nginx.repo
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1

$ mkdir -p /home/admin/packages/nginx
$ sudo yum makecache
$ sudo yum install --downloadonly --downloaddir=/home/admin/packages/nginx nginx

```

2.2、对于MySQL：
```

wget http://repo.mysql.com//mysql57-community-release-el7-8.noarch.rpm
rpm -ivh mysql57-community-release-el7-8.noarch.rpm
$ sudo vim /etc/yum.repos.d/mysql-community.repo #编辑配置安装版本为5.6,参考网址http://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/。

$ mkdir -p /home/admin/packages/mysql
$ sudo yum install --downloadonly --downloaddir=/home/admin/packages/mysql mysql-community-server

```

3、对于Supervisor这类Python程序及依赖下载:
```

$ mkdir -p /home/admin/packages/supervisor
$ sudo yum install --downloadonly --downloaddir=/home/admin/packages/supervisor python-setuptools
wget https://pypi.python.org/packages/44/80/d28047d120bfcc8158b4e41127706731ee6a3419c661e0a858fb0e7c4b2d/supervisor-3.3.0.tar.gz

```
下载依赖python-meld3:
源网址：https://pkgs.org/centos-7/epel-x86_64/python-meld3-0.6.10-1.el7.x86_64.rpm.html
```

wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-7.noarch.rpm
rpm -Uvh epel-release-7-7.noarch.rpm
$ sudo yum install --downloadonly --downloaddir=/home/admin/packages/supervisor python-meld3

```

4、对于需要源码编译安装的包下载.直接下载即可。如：
```

wget https://pypi.python.org/packages/45/a0/317c6422b26c12fe0161e936fc35f36552069ba8e6f7ecbd99bbffe32a5f/​meld3-1.0.2.tar.gz

```

----
##  安装 
将所有下载的rpm包拷贝至服务器上，然后可使用如下命令安装：
```

$ sudo yum install *.rpm
或者
rpm -ivh *.rpm

```


##  配置并使用本地yum repo源安装基础依赖(备选) 
1、首先挂载ISO镜像.
```

mkdir -p /mnt/cdrom
mount -o loop CentOS-7-x86_64-DVD-1511.iso /mnt/cdrom
# PS：如果是挂载DVD光驱做软件源，则用下面这条命令：
# mount -t iso9660 /dev/cdrom /mnt/cdrom/

```

----

2、修改yum配置：
```

cd /etc/yum.repos.d/
mkdir -p bak
# 接下来将之前的yum配置文件移动到上面创建的bak文件夹中 
mv *.repo bak/
# 接下来添加一个新的yum源配置文件
vi /etc/yum.repos.d/localrepo.repo
# 按“Insert”键进入编辑模式，复制下面的内容到配置文件

```
```

[localrepo]
name=Unixmen Repository
baseurl=file:///mnt/cdrom
gpgcheck=0
enabled=1

```
完成本地源配置过后，接下来就可以用 yum 进行 RPM 包的补装了。首先，查看刚刚配置好的 yum 源。
```

yum list或yum repolist

```

四、清理yum：
```

yum clean all

```
五、更新源，安装软件：
```

yum install 你要安装的软件包名

```



参考：
http://www.yimiju.com/articles/531.html
https://www.unixmen.com/setup-local-yum-repository-centos-7/
http://blog.csdn.net/tsangchoonhsia/article/details/6414780

##  Suse版 
系统版本： SUSE Linux Enterprise 11.0 x64

**1、本地源：**
挂载ISO镜像：
```

mount -o loop -t iso9660 /usr/local/tooldisk/mydisk3.iso  /dev/sr0

```
修改SUSE repo指向这个地址. /dev/sr0
```

vim /etc/zypp/repos.d/SUSE-Linux-Enterprise-Server-11\ 11-0.repo

[SUSE-Linux-Enterprise-Server-11 11-0]
name=SUSE-Linux-Enterprise-Server-11 11-0
enabled=1
autorefresh=0
baseurl=cd:///?devices=/dev/sr0
path=/
type=yast2
keeppackages=0

```

**2、使用本地源安装gcc,及gcc-c++**
```

zypper in gcc gcc-c++

```

**3、memcached安装:**
依赖:libevent:
```

wget https://github.com/libevent/libevent/releases/download/release-2.0.22-stable/libevent-2.0.22-stable.tar.gz
tar zxvf libevent-2.0.10-stable.tar.gz
cd libevent-2.0.10-stable
./configure -prefix=/usr
make
make install

```
测试libevent是否安装成功 ls -al /usr/lib| grep libevent
安装memcached，同时需要指定libevent的安装位置：
```

wget http://memcached.org/latest
tar -zxvf memcached-1.x.x.tar.gz
cd  memcached-1.x.x
./configure -prefix=/usr  -with-libevent=/usr
make
make install

```

启动Memcached的服务器端：
/usr/local/bin/memcached -d  -u root 

----

**4、Nginx安装:**
(注:suse下如果是在线安装的话应该zypper addrepo -G -t yum -c 'http://nginx.org/packages/sles/12' nginx && sudo zypper in nginx)
编译环境gcc g++ 开发库之类的需要提前装好，这里默认你已经装好。
一般我们都需要先装pcre, zlib，前者为了重写rewrite，后者为了gzip压缩。
4.1.选定源码目录
可以是任何目录，本文选定的是/usr/local/src
cd /usr/local/src

4.2.安装PCRE库
ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/ 下载最新的 PCRE 源码包，使用下面命令下载编译和安装 PCRE 包：
```

cd /usr/local/src
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
tar -zxvf pcre-8.39.tar.gz
cd pcre-8.39
./configure
make
make install

```

4.3.安装zlib库
http://zlib.net/zlib-1.2.8.tar.gz 下载最新的 zlib 源码包，使用下面命令下载编译和安装 zlib包：
```

cd /usr/local/src
wget http://zlib.net/zlib-1.2.8.tar.gz
tar -zxvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure
make
make install

```

4.4、安装openssl：
```

cd /usr/local/src
wget http://www.openssl.org/source/openssl-1.0.1c.tar.gz
tar -zxvf openssl-1.0.1c.tar.gz

```
4.5.安装nginx
下面是把 Nginx 安装到 /usr/local/nginx 目录下的详细步骤：
```

cd /usr/local/src
wget http://nginx.org/download/nginx-1.11.1.tar.gz
tar -zxvf nginx-1.11.1.tar.gz
cd nginx-1.11.1
./configure --sbin-path=/usr/local/nginx/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.39 --with-zlib=/usr/local/src/zlib-1.2.8 --with-openssl=/usr/local/src/openssl-1.0.1c
make
make install

mkdir -p  /etc/nginx/
cp conf/* /etc/nginx/nginx.conf

```

说明：
- -with-pcre=/usr/src/pcre-8.39 指的是pcre-8.39 的源码路径。
- -with-pcre=/usr/src/zlib-1.2.8 指的是zlib-1.2.8 的源码路径。

安装成功后 /usr/local/nginx 目录下如下
运行/usr/local/nginx/nginx 命令来启动 Nginx，
sudo /usr/local/nginx/nginx 

nginx的configure命令常用参数：
```

--prefix=path 定义一个目录，存放服务器上的文件 ，也就是nginx的安装目录。默认使用 /usr/local/nginx。 
--sbin-path=path 设置nginx的可执行文件的路径，默认为 prefix/sbin/nginx. 
--conf-path=path 设置在nginx.conf配置文件的路径。nginx允许使用不同的配置文件启动，通过命令行中的-c选项。默认为prefix/conf/nginx.conf. 
--pid-path=path 设置nginx.pid文件，将存储的主进程的进程号。安装完成后，可以随时改变的文件名 ， 在nginx.conf配置文件中使用 PID指令。默认情况下，文件名 为prefix/logs/nginx.pid. 
--error-log-path=path 设置主错误，警告，和诊断文件的名称。安装完成后，可以随时改变的文件名 ，在nginx.conf配置文件中 使用 的error_log指令。默认情况下，文件名 为prefix/logs/error.log. 
--http-log-path=path 设置主请求的HTTP服务器的日志文件的名称。安装完成后，可以随时改变的文件名 ，在nginx.conf配置文件中 使用 的access_log指令。默认情况下，文件名 为prefix/logs/access.log. 
--user=name 设置nginx工作进程的用户。安装完成后，可以随时更改的名称在nginx.conf配置文件中 使用的 user指令。默认的用户名是nobody。 
--group=name 设置nginx工作进程的用户组。安装完成后，可以随时更改的名称在nginx.conf配置文件中 使用的 user指令。默认的为非特权用户。 
--with-http_ssl_module — 使用https协议模块。默认情况下，该模块没有被构建。建立并运行此模块的OpenSSL库是必需的。 
--with-pcre=path — 设置PCRE库的源码路径。PCRE库的源码（版本4.4 - 8.30）需要从PCRE网站下载并解压。其余的工作是Nginx的./ configure和make来完成。正则表达式使用在location指令和 ngx_http_rewrite_module 模块中。 
--with-zlib=path —设置的zlib库的源码路径。要下载从 zlib（版本1.1.3 - 1.2.5）的并解压。其余的工作是Nginx的./ configure和make完成。ngx_http_gzip_module模块需要使用zlib 。 

```


----
5、rsync+sersync数据镜像工具安装:
suse自带了rsync.所以不需要额外安装.
5.1、下载inotify-tools :
```

wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz
./configure --prefix=/usr && sudo make && sudo su -c 'make install'

```

5.2、安装与配置sersync
```

cd ~
wget https://codeload.github.com/wsgzao/sersync/zip/master
mv master sersync.zip && unzip sersync.zip
cd sersync-master/
sudo tar -zxvf sersync2.5.4_64bit_binary_stable_final.tar.gz 
sudo mv GNU-Linux-x86 /opt/sersync
cd /opt/sersync

```

6、MySQL安装：
从MySQL官网下载:
```

MySQL-client-5.6.31-1.sles11.x86_64.rpm  
MySQL-server-5.6.31-1.sles11.x86_64.rpm  
MySQL-shared-5.6.31-1.rhel5.x86_64.rpm 

rpm -ivh *.rpm

```

ROOT密码文件会产生在~/.mysql_secret.然后第一次连接时记得通过set password来修改密码。
配置文件在/usr/my.cnf


7、supervisor安装
先安装python-setuptools
```

wget https://pypi.io/packages/source/s/setuptools/setuptools-24.0.2.zip --no-check-certificate
unzip setuptools-24.0.2.zip
cd setuptools-24.0.2/
python setup.py install

```
安装python-meld3
```

wget https://pypi.python.org/packages/45/a0/317c6422b26c12fe0161e936fc35f36552069ba8e6f7ecbd99bbffe32a5f/meld3-1.0.2.tar.gz --no-check-certificate
tar -zxvf meld3-1.0.2.tar.gz
cd meld3-1.0.2
python setup.py install

```
安装supervisor
```

wget https://pypi.python.org/packages/44/80/d28047d120bfcc8158b4e41127706731ee6a3419c661e0a858fb0e7c4b2d/supervisor-3.3.0.tar.gz
tar -zxvf supervisor-3.3.0.tar.gz
cd supervisor-3.3.0
python setup.py install

```
