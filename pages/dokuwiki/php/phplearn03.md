title: phplearn03 

#  Linux下PHP环境搭建 
PHP-FPM(PHP FastCGI Process Manager):用于管理PHP进程池的软件，用于接收和处理来自Web服务器的请求。
PHP-FPM配置文件:ubuntu:/etc/php5/fpm/php-fpm.conf, centos:/etc/php-fpm.conf
进程池配置:/etc/php5/fpm/pool.d/*.conf, centos下/etc/php-fpm.d/*.conf
主要配置www.conf文件：
```

user=www-data
group=www-data
listen=127.0.0.1:9000
listen.allowed_clients=127.0.0.1
pm.max_children=51

```
重启:sudo service php5-fpm restart ， centos下:sudo systemctl restart php-fpm.service
```

sudo add-apt-repository ppa:ondrej/php5-5.6
sudo apt-get install php5 php5-cgi php5-cli php5-fpm php5-common php5-curl php5-gd php5-imap php5-intl php5-mysqlnd php5-pspell php5-sqlite php5-tidy  php5-json  php5-mcrypt php5-readline php5-xmlrpc php5-enchant php5-xsl php5-interbase php5-xdebug php5-memcached

``` 


支持git的自动部署应用：Capistrano

本机安装ruby,然后
gem install capistrano
初始化项目
cap install 
编辑deplay.rb文件
```

# config valid only for current version of Capistrano
lock '3.3.5'
set :application, 'APP'
set :repo_url, 'git@github.com:USERNAME/REPO.git'

set :deploy_to, '/home/deploy/apps/my_app'

set :keep_releases, 5

namespace :deploy do
  desc "Build"
  after :updated, :build do
      on roles(:web) do
          within release_path  do
            execute :composer, "install --no-dev --quiet"
          end
      end
  end
end


```
production.rb
```

role :web, %w{deploy@123.456.78.90}

```

远程服务器安装git ,composer:
sudo apt-get install git


部署应用:本机打开终端输入cap production deploy
回滚:cap production deploy:rollback
