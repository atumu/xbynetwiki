title: phplearn35 

#  PHP学习之Session与Memcached 
PHP默认的会话处理程序会拖慢大型应用，因为这个处理程序把绘画数据存储在硬盘中，需要创建不必要的文件IO，浪费时间。我们应该把会话数据保存在内存中。可以使用Memcached或者Redis。这么做还可以支持分布式。
首先安装PECL扩展包:
```

apt-get install php5-memcached

```
php.ini配置：
```

session.save_handler='memcached'
session.save_path='127.0.0.1:11211'

```