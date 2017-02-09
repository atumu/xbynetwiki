title: windows_bat批处理脚本学习2 

#  Windows批处理(cmd/bat)常用命令小结 
批处理命令简介
  * echo
  * rem 注释
  * pause
  * call
  * start 开启一个新进程
  * goto
  * set设置环境变量
  * cls清屏
  * itle设置cmd窗口的标题
  * ver显示系统版本
  * date 和 time日期和时间
批处理符号简介
  * 回显屏蔽 @
  * 重定向1 >与> >
  * 重定向2 <
  * 管道符号 |
  * 转义符 ^
  * 逻辑命令符包括：&、&&、||
常用DOS命令
**文件夹管理**
  * cd 显示当前目录名或改变当前目录。
  * md 创建目录。
  * rd 删除一个目录。
  * dir 显示目录中的文件和子目录列表。
  * tree 以图形显示驱动器或路径的文件夹结构。
  * path 为可执行文件显示或设置一个搜索路径。
  * xcopy 复制文件和目录树。
**文件管理**
  * type 显示文本文件的内容。
  * copy 将一份或多份文件复制到另一个位置。
  * del 删除一个或数个文件。
  * move 移动文件并重命名文件和目录。(Windows XP Home Edition中没有)
  * ren 重命名文件。
  * replace 替换文件。
  * attrib 显示或更改文件属性。
  * find 搜索字符串。
  * fc 比较两个文件或两个文件集并显示它们之间的不同
**网络命令**
  * ping 进行网络连接测试、名称解析
  * ftp 文件传输
  * net 网络命令集及用户管理
  * telnet 远程登陆
  * ipconfig显示、修改TCP/IP设置
  * msg 给用户发送消息
  * arp 显示、修改局域网的IP地址-物理地址映射列表
系统管理
  * at 安排在特定日期和时间运行命令和程序
  * shutdown立即或定时关机或重启
  * tskill 结束进程
  * taskkill结束进程(比tskill高级，但WinXPHome版中无该命令)
  * tasklist显示进程列表(Windows XP Home Edition中没有)
  * sc 系统服务设置与控制
  * reg 注册表控制台工具
  * powercfg控制系统上的电源设置
对于以上列出的所有命令，在cmd中输入命令+/?即可查看该命令的帮助信息。如find /?

##  find
查找命令
find "abc" c:test.txt
在 c:test.txt 文件里查找含 abc 字符串的行
如果找不到，将设 errorlevel 返回码为1
find /i “abc” c:test.txt
查找含 abc 的行，忽略大小写
find /c "abc" c:test.txt
显示含 abc 的行的行数

more (外部命令)
逐屏显示
more c:test.txt     #逐屏显示 c:test.txt 的文件内容

%0 %1 %2 %3 %4 %5 %6 %7 %8 %9 %*
命令行传递给批处理的参数
%0 批处理文件本身
%1 第一个参数
%9 第九个参数
%* 从第一个参数开始的所有参数


参考:http://www.tuicool.com/articles/nQZNVb