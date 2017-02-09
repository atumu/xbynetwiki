title: windows中杀死占用某个端口的进程 

#  Windows中杀死占用某个端口的进程 
http://www.tuicool.com/articles/bmM3Ev
第一步，根据端口号查找对应的进程号
netstat -ano | findstr 80 / /列出进程极其占用的端口，且包含 80
第二步， 据进程号寻找进程名称
tasklist | findstr 2000
taskkill -PID <进程号> -F //强制关闭某个进程