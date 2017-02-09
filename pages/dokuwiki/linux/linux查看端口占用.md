title: linux查看端口占用 

#  linux查看端口占用 

ps -aux | grep tomcat 发现并没有8080端口的Tomcat进程
sudo netstat –apn | grep 8080 查看端口占用