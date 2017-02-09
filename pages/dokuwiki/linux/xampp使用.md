title: xampp使用 

xampp**默认不允许外网访问**。需要修改配置文件。
` xampp security `命令用于初始化设置各项密码。
##  linux下xampp忘记mysql root密码 
如果在ubuntu 下面 使用xampp这个集成开发环境，却忘记mysql密码。
找回密码的步骤如下：
1、停止mysql服务器
sudo /opt/lampp/lampp stopmysql
2、使用`--skip-grant-tables' 参数来启动 mysqld
sudo /opt/lampp/sbin/mysqld --skip-grant-tables
3、再开一个终端(在终端中直接右键+B) 进入mysql
sudo /opt/lampp/bin/mysql -uroot
现在会直接进入mysql
4、连接mysql权限数据库
use mysql;
5、修改root用户的密码
update user set password=password("123456") where user="root";
6、刷新权限表（必须要有这一步）
flush privileges;
7、退出mysql
quit;
8、重启mysql服务
sudo /opt/lampp/lampp startmysql
ok 现在就可以使用刚才设置的密码登录msql了

参考：
http://www.cnblogs.com/abinxm/archive/2011/10/09/2204824.html
http://www.cnblogs.com/ylqmf/archive/2011/12/29/2305730.html
http://blog.sina.com.cn/s/blog_9c581bd30101c4ba.html

##  xampp下mysql开启远程连接 
当然还需要修改user那张表下面的host，改成%，比如要让root远程可访问，就用把root的host设置成%
进入phpmyadmin管理界面首页，点击“权限”后，再“添加新用户”` ，用户名：root（随意），主机：%（必须是%） `，对应的密码可以设置可以不设置，即可开通远程连接；