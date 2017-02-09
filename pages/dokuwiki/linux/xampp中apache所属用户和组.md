title: xampp中apache所属用户和组 

#  xampp中apache所属用户和组 

最近搭建了一个新的服务器，使用XAMPP，但是很奇怪即使我执行了以下权限设置:
chown www-data:www-data -R htdocs/
chmod 775 -R htdocs/
访问时任然提示权限问题。只有设置chmod 777 -R htdocs/才正常。
让我很纳闷。
猜想可能是apache进程所属用户和组被XAMPP改动了。
在lampp/etc/httpd.conf在搜索User或者Group（注意大小写）：
结果发现：
![](/data/dokuwiki/linux/pasted/20150517-144253.png)
apache用户和组都改为了daemon，晕死。弄了我很久。
chown daemon:daemon -R htdocs/
chmod 775 -R htdocs/
一切正常。
参考：
http://blog.csdn.net/liruxing1715/article/details/39205415