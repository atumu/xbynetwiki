title: phplearn019 

#  PHP学习之Eclipse PDT与Xdebug断点调试 
视频教程:https://www.youtube.com/watch?v=Dm2ivX3uwR4
1、安装Xdebug
在https://xdebug.org/download.php下载对应版本的xdebug.dll
然后移动到php\ext目录下。配置php.ini:
<blockquote>[Xdebug]
zend_extension = "C:\Users\Administrator\Desktop\nginx\php\ext\php_xdebug.dll"
xdebug.remote_enable = 1  
xdebug.remote_host = "127.0.0.1"  ;本地服务器地址，这个本地服务器不是Eclipse里面的Server而是我们本地下载的。如nginx，apapche。
xdebug.remote_port = 19012   ;该端口用于xdebug和IDE通信。
xdebug.remote_handler = "dbgp"
xdebug.profiler_enable = 1
xdebug.profiler_enable_trigger = 1
xdebug.profiler_output_dir = "D:\log"
xdebug.trace_output_dir = "D:\log"</blockquote>

phpinfo()查看一下xdebug是否安装正确。或者命令行通过php -m查看模块列表。

2、配置nginx+php，略。

3、Eclipse安装PDT
4、配置PDT
打开PHP->debug->Debugger->xdebug.配置xdebug端口
![](/data/dokuwiki/php/pasted/20160411-214941.png)
![](/data/dokuwiki/php/pasted/20160411-215135.png)
![](/data/dokuwiki/php/pasted/20160411-215156.png)
PHP->servers配置debug。
![](/data/dokuwiki/php/pasted/20160411-220006.png)

5、` 首先启动外部的nginx服务器程序。(不然会出问题，切记！！！) `然后打断点，Run-As-PHPWebPages即可。

问题:
1、waiting xdebug session 解决，外部Nginx服务器没有启动
2、Web Launching Already。 解决，切换到debug视图。点击终止按钮
![](/data/dokuwiki/php/pasted/20160411-215701.png)


如果需要使用外部浏览器进行调试，请参考:
http://www.jb51.net/article/65118.htm