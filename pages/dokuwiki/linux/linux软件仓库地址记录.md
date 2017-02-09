title: linux软件仓库地址记录 

#  Linux软件仓库地址记录 
##  开源镜像: 
阿里云:http://mirrors.aliyun.com/
163:http://mirrors.163.com/

##  Ubuntu PPA 
添加示例:
```

sudo apt-get install python-software-properties
sudo add-apt-repository ppa:nginx/stable 
sudo apt-get update
sudo apt-get install <package name>

```
PHP:https://launchpad.net/~ondrej/+archive/ubuntu/php5
  * PHP 5.4: ppa:ondrej/php5-oldstable (Ubuntu 12.04 LTS)
  * PHP 5.5: ppa:ondrej/php5 (Ubuntu 14.04 LTS)
  * PHP 5.6: ppa:ondrej/php5-5.6 (Ubuntu 14.04 LTS - Ubuntu 16.04 LTS)
  * PHP 5.6 and PHP 7.0: ppa:ondrej/php (Ubuntu 14.04 LTS - Ubuntu 16.04 LTS)
```

sudo add-apt-repository ppa:ondrej/php5-5.6

``` 

Nginx:
```

sudo add-apt-repository ppa:nginx/stable

``` 

[NodeJS](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions):
```

curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
sudo apt-get install -y nodejs

```

Ruby:
```

sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
sudo apt-get install ruby2.3

```