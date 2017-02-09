title: linux常用命令 

#  linux常用命令 
##  shell命令相关 
命令别名 alias ll='ls -l'
##  服务器操作 
linux检查端口:
sudo netstat -tlnp|grep 8080

检查用户所属组：
groups 查看当前用户所属的组
groups <user1> <user2> <user3> 查看<user1>, <user2> 和 <user3>所属的组

查看进程所属用户：
ps aux|grep nginx

添加用户到已存在组
adduser xby nginx
#  压缩与解压 

在linux中用tar来存储或者展开tar的存档文件.必须配合参数使用
1:压缩一个文件为tar.gz后缀.  (注意,tar保存的目录是#pwd 所在的目录)
#tar zcvf test1.tar.gz /home/www    
2:解压一个后缀为tar.gz的文件.
#tar zxvf test1.tar.gz       (解压后会保存成 test1的一个文件夹)
用一个命令完成压缩
#tar cvf - /home/www/ | gzip -qc > test3.tar.gz
5:tar打包时,去掉某些子目录或者指定文件,使用 -exclude {dirname,filename} 
比如我们要打包/home/www网站全站,放在/home目录下,除了一些比如db_m(phpmyadmin)之类的,本地有就不需要
#tar zcvf /home/bk0406-1717.tar.gz /home/www  --exclude /home/www/db_m --exclude /home/www/images --exclude /home/docs --exclude=/home/www/config.inc.php
**这里需要注意的是,exclude 后面的目录不能带 /  带了 /就会失效**
6:查看tar包里面的内容
#tar -tvf /home/bk0406-1717.tar.gz

压缩 
tar –cvf jpg.tar *.jpg / /将目录里所有jpg文件打包成tar.jpg 
tar –czf jpg.tar.gz *.jpg / /将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz 
tar –cjf jpg.tar.bz2 *.jpg / /将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2 
tar –cZf jpg.tar.Z *.jpg / /将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z 
rar a jpg.rar *.jpg / /rar格式的压缩，需要先下载rar for linux 
zip jpg.zip *.jpg / /zip格式的压缩，需要先下载zip for linux 

**linux下tar命令解压到指定的目录**
#tar zxvf /bbs.tar.zip -C /zzz/bbs    
把根目录下的bbs.tar.zip解压到/zzz/bbs下，**前提要保证存在/zzz/bbs这个目录存在** 
这个和cp命令有点不同，cp命令如果不存在这个目录就会自动创建这个目录！

解压 
tar –xvf file.tar //解压 tar包 
tar -xzvf file.tar.gz //解压tar.gz 
tar -xjvf file.tar.bz2 //解压 tar.bz2 
tar –xZvf file.tar.Z //解压tar.Z 
unrar e file.rar //解压rar 
unzip file.zip //解压zip 

参考：http://blog.csdn.net/rainysia/article/details/7433041
http://blog.sina.com.cn/s/blog_62449fcf0100nfar.html