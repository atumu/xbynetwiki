title: linux4 

#  CentOS7服务与开机管理systemctl命令 
CentOS 7.0中已经没有service命令，而是启用了systemctl服务器命令
摘要
systemctl 是系统服务管理器命令，它实际上将 service 和 chkconfig 这两个命令组合到一起。
任务	旧指令	新指令
使某服务自动启动	chkconfig –level 3 httpd on	systemctl enable httpd.service
使某服务不自动启动	chkconfig –level 3 httpd off	systemctl disable httpd.service
检查服务状态	service httpd status	systemctl status httpd.service （服务详细信息） systemctl is-active httpd.service （仅显示是否 Active)
显示所有已启动的服务	chkconfig –list	systemctl list-units –type=service
启动某服务	service httpd start	systemctl start httpd.service
停止某服务	service httpd stop	systemctl stop httpd.service
重启某服务	service httpd restart	systemctl restart httpd.service


下面以nfs服务为例：
1.启动nfs服务
systemctl start nfs-server.service
2.设置开机自启动
systemctl enable nfs-server.service
3.停止开机自启动
systemctl disable nfs-server.service
4.查看服务当前状态
systemctl status nfs-server.service
5.重新启动某服务
systemctl restart nfs-server.service
6.查看所有已启动的服务
systemctl list -units --type=service
开启防火墙22端口
iptables -I INPUT -p tcp --dport 22 -j ACCEPT
如果仍然有问题，就可能是SELinux导致的
关闭SElinux：
修改/etc/selinux/config文件中的SELINUX=”” 为 disabled，然后重启
彻底关闭防火墙：
sudo systemctl status  firewalld.service
sudo systemctl stop firewalld.service          
sudo systemctl disable firewalld.service
参考:http://linux.it.net.cn/CentOS/fast/2014/0720/3212.html