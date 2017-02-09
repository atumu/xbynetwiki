title: nginx_php-fpm配置实战 

#  nginx+php-fpm配置实战 
##  基本安装 
以ubuntu为例：
1、添加nginx软件源：
sudo vim /etc/apt/sources.list
```

deb http://nginx.org/packages/mainline/ubuntu/ trusty nginx
deb-src http://nginx.org/packages/mainline/ubuntu/ trusty nginx

```
<note tip>如果提示： W: GPG error: http://nginx.org precise Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY ABF5BD827BD9BF62 由于官方不信任该源
解决方法：```

 sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ABF5BD827BD9BF62

```</note>

2、更新和安装update and install
sudo apt-get update 
sudo apt-get install nginx


当然你也可以通过添加PPA的方式:
```

sudo add-apt-repository ppa:nginx/stable 
sudo apt-get update
sudo apt-get install nginx

```
3、启动nginx
sudo service nginx start

4、check version
nginx -v

6、配置php+mysql
sudo apt-get install mysql-server php5-cli php5-cgi  php5-mysql  php5-curl php5-gd  php-pear php5-imagick php5-imap  php5-memcache   php5-sqlite php5-tidy php5-mongo php-pecl-apc php-gd php-mcrypt php-pear php-mbstring php-xmlrpc php-dom

7、安装php-fpm:
sudo apt-get install  php5-fpm 
启动sudo service php5-fpm

##  配置 nginx 虚拟主机 
下面来看一下为 nginx 配置虚拟主机。先进入到 nginx 配置文件目录：
cd /etc/nginx/conf.d
复制这个目录里的 default.conf ，复制以后的名字可以使用你的虚拟主机名字。比如创建一个 nginx.ninghao.net 的虚拟主机。复制文件可以使用 cp 命令，像这样：
cp default.conf nginx.ninghao.net.conf
再去编辑一下这个复制以后的配置文件，可以使用 vim 命令：
vim nginx.ninghao.net.conf
你会看到像这样的代码：
server {
 listen 80;
 server_name localhost;
 #charset koi8-r;
 #access_log   /var/log/nginx/log/host.access.log main;
 location / {
 root /usr/share/nginx/html;
 index index.html index.htm;
}
...
}
server_name 就是主机名，也就是跟这个虚拟主机绑定在一块儿的域名，所以，server_name 后面的东西就是 nginx.ninghao.net 。紧接着 server_name 下面可以是一个 
root，就是这个虚拟主机的根目录，也就是网站所在的目录。比如我们要把 nginx.ninghao.net 这个网站的文件放在 /home/www/nginx.ninghao.net 下面，那么这个 root 就是这个路径。
然后去掉 location / 里面的 root 这行代码。再在 index 后面加上一种索引文件名，也就是默认打开的文件，这里要加上一个 index.php ，这样访问 nginx.ninghao.net 就可以直接打开 root 目录下面的 index.php 了。稍后我们再去安装 php 。修改之后，看起来像这样：
```

server {
 listen 80;
 server_name nginx.ninghao.net;
 root /home/www/nginx.ninghao.net;
 #charset koi8-r;
 #access_log /var/log/nginx/log/host.access.log main;

 location / {
 index index.php index.html index.htm;
 }
...
}

```
重启 nginx 或者重新加载 nginx 可以让配置文件生效。
service nginx reload
##  配置php-fpm让 nginx 可以执行 php 
现在我们应该就可以让 nginx 去执行 php 了。不过你需要修改一下 nginx 的配置文件，之前我们在配置虚拟主机的时候，创建了一个 nginx.ninghao.net.conf 的配置文件，需要去修改下 nginx 的这个配置文件，才能去执行 php 。使用 vim 命令去编辑它：
vim /etc/nginx/conf.d/nginx.ninghao.net.conf
注意你的配置文件不一定叫 nginx.ninghao.net.conf，应该是你自己命名的配置文件。打开以后，找到下面这段字样的代码：
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}
这是 nginx 默认给我们的用来执行 php 的配置，从 location 开始取消注释，会让这个配置生效，然后我们还得简单去修改一下：
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
    #   root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
注意 root 那里仍然是被注释掉的，还有 SCRIPT_FILENAME 后面修改了一下，把 /scripts 换成了 $document_root 。保存并退出。然后重新启动 nginx：
这个$document_root指定是server里面的root 而不是location里面的，所以我们还需要将root移到server下。如下
```

server {
    listen       80;
    server_name  abc.oscc.top;
    root   /opt/www; #必须放这里，不然php配置中的$document_root会出问题，获取不到值
    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
    #    root   /usr/share/nginx/html;#这里就注视掉
        
        index  index.php index.html index.htm index.php;
    }

```
service nginx restart

###  目录与权限问题
网站上面有些目录或文件需要有写入权限，这样你得为这些目录和文件分配合适的权限，**一般可以把它们的拥有者设置成 php 和 nginx 使用的用户**，
**默认 nginx 的用户就是 nginx ，而 php-fpm 使用的用户默认是 apache。**我们可以把它们**改成一个统一的用户**，**可以修改 php-fpm 的用户为 nginx** 。
你可以使用下面的命令去查看一下 nginx 和 php-fpm 所使用的用户名：
```

ps aux|grep php
ps aux|grep nginx

```
修改所使用的用户，可以通过使用 nginx 和 php-fpm 的配置文件，nginx 的配置文件是：  /etc/nginx/nginx.conf ，php-fpm 的配置文件是：/etc/php-fpm.conf，还有在 /etc/php-fpm.d/* 这个目录里的所有文件都是 php-fpm 的配置文件。默认这个目录里有一个 www.conf ，你可以编辑这个文件来修改 php-fpm 所使用的用户名称。使用 vim 命令：
```

vim /etc/php5/fpm/pool.d/www.conf

```
修改用户：
```

;user = www-data
user = nginx
;group = www-data
group = nginx

listen.owner = nginx
listen.group = nginx

```
重启下 php-fpm，这样**我们的 nginx 服务器还有 php-fpm 会使用同一个用户：nginx**，你可以把要可以有写入权限的目录与文件的拥有者修改成 nginx 就行了。可以使用 chown 命令：
```

chown -R nginx 目录名/文件名

```

###  测试是否可以执行 php 
现在，我们已经安装了 php-fpm，并修改了 nginx 的配置文件让它可以去执行 php，下面，我们得去测试一下，可以使用 php 的 phpinfo(); 函数，方法是在你的虚拟主机根目录下面，创建一个 php 文件，命名为 phpinfo.php，然后在这个文件里输入：
<?php phpinfo(); ?>
` 浏览器输入地址报错(502)： `
检查日志:less /var/log/nginx/error.log
[error] 190#190: *1 connect() failed (111: Connection refused) while connecting to upstream （127.0.0.1:9000)
检查php5-fpm是否正在运行 service php5-fpm status.正在运行。
` 检查9000端口：netstat -lnp |grep 9000 竟然没有。这就奇怪了。 `
原来是没有在 php-fpm配置文件中指定端口。所以会出现这样的错误。那么
vim /etc/php5/fpm/pool.d/www.conf
```

;listen = /var/run/php5-fpm.sock
listen = 127.0.0.1:9000

```
重启php5-fpm ,nginx。访问正常了。ok!

##  完整配置文件： 
/etc/nginx/config.d/wiki.conf:
```

server {
    listen       80;
    server_name  abc.oscc.top;
    root   /opt/www; #必须放这里，不然php配置中的$document_root会出问题，获取不到值
    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
    #    root   /usr/share/nginx/html;
        
        index  index.php index.html index.htm index.php;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        #root           /opt/www;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;

    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```



参考：鸟哥教程http://www.laruence.com/2009/07/28/1030.html
http://ninghao.net/blog/1368
http://jingyan.baidu.com/article/6d704a131b1e7828db51cacb.html
http://www.nginx.cn/231.html
http://www.kankanews.com/ICkengine/archives/36593.shtml