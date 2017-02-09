title: ubuntu初始化1 

#  ubuntu初始化操作 
0、修改字符编码为UTF-8 (默认)

1、安装vim、nano:
```

sudo apt-get install vim nano

```
2、修改软件源:参考http://wiki.ubuntu.org.cn/Template:14.04source
```

sudo  vim /etc/apt/sources.list

```

Ubuntu 官方（欧洲，国内较慢，无同步延迟）
http://archive.ubuntu.com/ubuntu/
Ubuntu 官方中国（目前是阿里云）
http://cn.archive.ubuntu.com/ubuntu/
网易（广东广州电信/联通千兆双线接入）
http://mirrors.163.com/ubuntu/
搜狐（山东联通千兆接入）
http://mirrors.sohu.com/ubuntu/
阿里云（北京万网/浙江杭州阿里云服务器双线接入）
http://mirrors.aliyun.com/ubuntu/

3、安装基础包:
```

sudo apt-get install build-essential python-software-properties software-properties-common

``` 

4、添加常用软件ppa：
```

sudo add-apt-repository ppa:nginx/stable 
sudo add-apt-repository ppa:ondrej/php5-5.6

```
移除PPA：
```

sudo add-apt-repository ppa:ondrej/php5-5.6

```
5、安装软件
```

sudo apt-get install nginx 
sudo apt-get install php5 php5-cgi php5-cli php5-fpm php5-common php5-curl php5-gd php5-imap php5-intl php5-mysqlnd php5-pspell php5-sqlite php5-tidy  php5-json  php5-mcrypt php5-readline php5-xmlrpc php5-enchant php5-xsl php5-interbase php5-xdebug

sudo apt-get install wget curl

```

##  python环境初始化 
sudo apt install python3-pip
sudo apt-get install build-essential libssl-dev libffi-dev python-dev python3-dev
sudo pip3 install psutil selenium ipy python-nmap fabric paramiko gunicorn  xlrd xlsxwriter simplejson markdown click wtforms scapy-python3 celery sqlalchemy scrapy requests passlib itsdangerous 
sudo pip3 install flask flask-moment flask-security flask-testing flask-cache flask-principal flask-babel flask-script flask-sqlalchemy flask-migrate flask-wtf flask-login flask-bootstrap flask-mail 

##  工具包 
sudo apt-get install aria2  axel wget curl
sudo add-apt-repository ppa:webupd8team/sublime-text-3
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5 (config.ini is located under ~/.config/shadowsocks-qt5/ on *nix platforms)
sudo apt-get install sublime-text-installer

axel -n 10 https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

