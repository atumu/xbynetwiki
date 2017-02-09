title: python-nmap 

#  端口扫描python-nmap 
NMap，也就是Network Mapper，最早是Linux下的网络扫描和嗅探工具包。
nmap是一个网络连接端扫描软件，用来扫描网上电脑开放的网络连接端。确定哪些服务运行在哪些连接端，并且推断计算机运行哪个操作系统（这是亦称 fingerprinting）。
nmap 也是不少黑客及骇客（又称脚本小子）爱用的工具 。系统管理员可以利用nmap来探测工作环境中未经批准使用的服务器，但是黑客会利用nmap来搜集目标电脑的网络设定，从而计划攻击的方法。
基本功能有三个，一是探测一组主机是否在线；其次是扫描 主机端口，嗅探所提供的网络服务；还可以推断主机所用的操作系统 。
sudo apt-get install nmap
进行ping扫描，打印出对扫描做出响应的主机,不做进一步测试(如端口扫描或者操作系统探测)：
nmap -sP 192.168.1.0/24
仅列出指定网络上的每台主机，不发送任何报文到目标主机：
nmap -sL 192.168.1.0/24
探测目标主机开放的端口，可以指定一个以逗号分隔的端口列表(如-PS22，23，25，80)：
nmap -PS 192.168.1.234
使用UDP ping探测主机：
nmap -PU 192.168.1.0/24
使用频率最高的扫描选项：SYN扫描,又称为半开放扫描，它不打开一个完全的TCP连接，执行得很快：
nmap -sS 192.168.1.0/24

 nmap常用的几个扫描参数
-sS TCP SYN 扫描 (又称半开放,或隐身扫描)

扫描技术类型：
  -sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
  -sU: UDP Scan
  -sN/sF/sX: TCP Null, FIN, and Xmas scans
  --scanflags <flags>: Customize TCP scan flags
  -sI <zombie host[:probeport]>: Idle scan
  -sY/sZ: SCTP INIT/COOKIE-ECHO scans
  -sO: IP protocol scan
  -b <FTP relay host>: FTP bounce scan

端口
-p <port ranges>: Only scan specified ports ，Ex: -p22; -p1-65535; -p U:53,111,137,T:21-25,80,139,8080,S:9
- -exclude-ports <port ranges>: Exclude the specified ports from scanning

-P0 允许你关闭 ICMP pings.

-sV 打开系统版本检测
-O 尝试识别远程操作系统
-v 详细输出扫描情况.

python-nmap是对nmap的python封装。不能用于windows。
http://xael.org/pages/python-nmap-en.html

sudo apt-get install nmap
pip install python-nmap

python-nmap模块有两个常用类，一个为PortScanner（）类，实现一个nmap工具的端口扫描功能封装；另一个为PortScannerHostDict（）类，实现存储于访问主机的扫描结果

scan(self,hosts='127.0.0.1',port=None,arguments='-sV')方法，实现指定主机，端口，nmap命令行参数的扫描。
示例1:创建PortScanner实例，然后扫描159.239.210.26这个IP的20-443端口。
```

import nmap
nm = nmap.PortScanner()
ret = nm.scan('115.239.210.26','20-443')
print(ret)
print nm.scaninfo()
# {u'tcp': {'services': u'20-443', 'method': u'syn'}}
print nm.command_line() 
# u'nmap -oX - -p 20-443 -sV 115.239.210.26' 
print nm.all_hosts() #查看有多少个host
查看该host的详细信息
nm['115.239.210.26']
查看该host包含的所有协议
nm['115.239.210.26'].all_protocols() 
查看该host的哪些端口提供了tcp协议
nm['115.239.210.26']['tcp']
nm['115.239.210.26']['tcp'].keys() 
查看该端口是否提供了tcp协议及其状态，包括四种状态（up，down，unknown，skipped）
nm['115.239.210.26'].has_tcp(21)
还可以像这样设置nmap执行的参数
nm.scan(hosts='192.168.1.0/24', arguments='-n -sP -PE -PA21,23,80,3389') 

```
## 示例2：端口扫描 
http://www.tianfeiyu.com/?p=1360
```

import sys
import nmap
 
'''
    端口扫描
'''
 
scan_row = []
input_date = input('please input hosts and ports:')
scan_row = input_date.split(' ')
 
if len(scan_row)!=2:
    print('input errors,example "192.168.1.0/24 80,443,22"')
    sys.exit(0)
 
hosts = scan_row[0]     #接受用户输入的主机
port = scan_row[1]      #接受用户输入的端口
 
try : 
    nm = nmap.PortScanner()     #创建端口扫描对象
except nmap.PortScannerError :
    print('nmap not found',sys.exc_info()[0])   #!!
    sys.exit()
except :
    print('unexpected error:',sys.exc_info()[0])
    sys.exit()
    
try:
    #调用扫描方法，参数指定扫描主机hosts，nmap扫描命令行参数arguments
    nm.scan(hosts=hosts,arguments=' -v -sS -p '+port)
except Exception as e:
    print('scan error:'+str(e))
    
for host in nm.all_hosts():         #遍历扫描主机
    print('----------------------------------------------')
    print('Host : %s (%s)' % (host,nm[host].hostname()))        #输出主机及主机名
    print('State : %s' % nm[host].state())                  #输出主机状态信息，如up，down
    for proto in nm[host].all_protocols():                  #遍历扫协议，如tcp，udp
        print('---------------------------')
        print('Protocol : %s' %proto)                       #输入协议名
        
        lport = nm[host][proto].keys()          #获取协议的所有扫描端口,输出为字典形式
        sorted(lport)                           #端口列表排序
        for port in lport:          #遍历端口及输出端口与状态
            print('port : %s \tstate : %s' % (port,nm[host][proto][port]['state']))

```
