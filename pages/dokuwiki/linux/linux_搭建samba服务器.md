title: linux_搭建samba服务器 

#  linux_搭建samba服务器 
` sudo apt-get install samba `

配置
1.windows 访问 ubuntu
第一步创建共享目录： 比如要创建/home/用户名/share首先创建这个文件夹 （这个用户名就是你的用户名，为了方便易懂我才这样写的，到时记得自己改啊）
代码:
mkdir /home/用户名/share    （新建share文件夹）
chmod 777 /home/用户名/share   （设置该文件夹的权限使其让所有用户可读可写可运行）
备份并编辑smb.conf允许网络用户访问 （养成随时备份的好习惯，在关键的时候你会发现当初的备份是多么的明智！） 代码:
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf_backup
sudo gedit /etc/samba/smb.conf
搜寻这一行文字 代码:
; security = user
用下面这几行取代
代码:
security = user
username map = /etc/samba/smbusers
将下列几行新增到文件的最后面，假设允许访问的用户为：new。而文件夹的共享名为 Share ＃这里之所以这么写就是因为后面我们要创建一个smb用户new，并且让XP用户通过这个new来和我们进行数据交流。当然你可以写为自己喜欢的名字 只不过前后要一致就可以了
代码:
[Share]
comment = Shared Folder with username and password
path = /home/用户名/share
public = yes
writable = yes
valid users = new
create mask = 0700
directory mask = 0700
force user = nobody
force group = nogroup
available = yes
browseable = yes
然后顺便把这里改一下，找到［global］把 workgroup = MSHOME 改成 :（注意，这里的WORKGROUP是共享中的工作组名称） 代码:
workgroup = WORKGROUP
display charset = UTF-8
unix charset = UTF-8
dos charset = cp936
` 后面的三行是为了防止出现中文目录乱码的情况。 ` 现在要添加new这个网络访问帐户。如果系统中当前没有这个帐户，那么
代码:
sudo useradd new
要注意，上面只是增加了new这个用户，却没有给用户赋予本机登录密码。所以这个用户将只能从远程访问，不能从本机登录。而且samba的登录密码可以和本机登录密码不一样。现在要新增网络使用者的帐号：
代码:
sudo smbpasswd -a new （设置你的new密码，这个密码不是开机登录时候用的，是你要访问WIN共享文件或者WIN共享文件访问你的时候要填的密码） 

参考：http://wiki.ubuntu.org.cn/Samba
http://my.oschina.net/junn/blog/171388