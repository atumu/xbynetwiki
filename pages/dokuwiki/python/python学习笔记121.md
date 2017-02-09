title: python学习笔记121 

#  Python总结之常用模块 
##  sys 
		sys.argv              # 取参数列表
		sys.exit(2)           # 退出脚本返回状态 会被try截取
		sys.exc_info()        # 获取当前正在处理的异常类
		sys.version           # 获取Python解释程序的版本信息
		sys.maxint            # 最大的Int值  9223372036854775807
		sys.maxunicode        # 最大的Unicode值
		sys.modules           # 返回系统导入的模块字段，key是模块名，value是模块
		sys.path              # 返回模块的搜索路径，初始化时使用PYTHONPATH环境变量的值
		sys.platform          # 返回操作系统平台名称
		sys.stdout            # 标准输出
		sys.stdin             # 标准输入
		sys.stderr            # 错误输出
		sys.exec_prefix       # 返回平台独立的python文件安装的位置
		sys.stdin.readline()  # 从标准输入读一行
		sys.stdout.write("a") # 屏幕输出a 
##  tmpfile模块 
temp = tempfile.TemporaryFile()
##  os 
		# 相对sys模块 os模块更为底层 os._exit() try无法抓取
		os.popen('id').read()      # 执行系统命令得到返回结果
		os.system()                # 得到返回状态 返回无法截取
		os.name                    # 返回系统平台 Linux/Unix用户是'posix'
		os.getenv()                # 读取环境变量
		os.putenv()                # 设置环境变量
		os.getcwd()                # 当前工作路径
		os.chdir()                 # 改变当前工作目录
		os.walk('/root/')          # 递归路径
		文件处理
			mkfifo()/mknod()       # 创建命名管道/创建文件系统节点
			remove()/unlink()      # 删除文件
			rename()/renames()     # 重命名文件
			*stat()                # 返回文件信息
			symlink()              # 创建符号链接
			utime()                # 更新时间戳
			walk()                 # 遍历目录树下的所有文件名
		目录/文件夹
			chdir()/fchdir()       # 改变当前工作目录/通过一个文件描述符改变当前工作目录
			chroot()               # 改变当前进程的根目录
			listdir()              # 列出指定目录的文件
			getcwd()/getcwdu()     # 返回当前工作目录/功能相同,但返回一个unicode对象
			mkdir()/makedirs()     # 创建目录/创建多层目录
			rmdir()/removedirs()   # 删除目录/删除多层目录
		
		访问/权限
			saccess()              # 检验权限模式
			chmod()                # 改变权限模式
			chown()/lchown()       # 改变owner和groupID功能相同,但不会跟踪链接
			umask()                # 设置默认权限模式
			
		文件描述符操作
			open()                 # 底层的操作系统open(对于稳健,使用标准的内建open()函数)
			read()/write()         # 根据文件描述符读取/写入数据 按大小读取文件部分内容
			dup()/dup2()           # 复制文件描述符号/功能相同,但是复制到另一个文件描述符
		
		设备号
			makedev()              # 从major和minor设备号创建一个原始设备号
			major()/minor()        # 从原始设备号获得major/minor设备号
		
###  os.path模块 
os.path.expanduser('~/.ssh/key')   # 家目录下文件的全路径
			分隔
				os.path.basename()         # 去掉目录路径,返回文件名
				os.path.dirname()          # 去掉文件名,返回目录路径
				os.path.join()             # 将分离的各部分组合成一个路径名
				os.path.spllt()            # 返回(dirname(),basename())元组
				os.path.splitdrive()       # 返回(drivename,pathname)元组
				os.path.splitext()         # 返回(filename,extension)元组
			
			信息
				os.path.getatime()         # 返回最近访问时间
				os.path.getctime()         # 返回文件创建时间
				os.path.getmtime()         # 返回最近文件修改时间
				os.path.getsize()          # 返回文件大小(字节)
			
			查询
				os.path.exists()          # 指定路径(文件或目录)是否存在
				os.path.isabs()           # 指定路径是否为绝对路径
				os.path.isdir()           # 指定路径是否存在且为一个目录
				os.path.isfile()          # 指定路径是否存在且为一个文件
				os.path.islink()          # 指定路径是否存在且为一个符号链接
				os.path.ismount()         # 指定路径是否存在且为一个挂载点
				os.path.samefile()        # 两个路径名是否指向同一个文件
		
###  相关模块 
			base64              # 提供二进制字符串和文本字符串间的编码/解码操作
			binascii            # 提供二进制和ASCII编码的二进制字符串间的编码/解码操作
			bz2                 # 访问BZ2格式的压缩文件
			csv                 # 访问csv文件(逗号分隔文件)
			csv.reader(open(file))
			filecmp             # 用于比较目录和文件
			fileinput           # 提供多个文本文件的行迭代器
			getopt/optparse     # 提供了命令行参数的解析/处理
			glob/fnmatch        # 提供unix样式的通配符匹配的功能
			gzip/zlib           # 读写GNU zip(gzip)文件(压缩需要zlib模块)
			shutil              # 提供高级文件访问功能
			c/StringIO          # 对字符串对象提供类文件接口
			tarfile             # 读写TAR归档文件,支持压缩文件
			tempfile            # 创建一个临时文件
			uu                  # uu格式的编码和解码
			zipfile             # 用于读取zip归档文件的工具
			environ['HOME']     # 查看系统环境变量
		
		子进程
			os.fork()    # 创建子进程,并复制父进程所有操作  通过判断pid = os.fork() 的pid值,分别执行父进程与子进程操作，0为子进程
			os.wait()    # 等待子进程结束
###  跨平台os模块属性 
			linesep         # 用于在文件中分隔行的字符串
			sep             # 用来分隔文件路径名字的字符串
			pathsep         # 用于分割文件路径的字符串
			curdir          # 当前工作目录的字符串名称
			pardir          # 父目录字符串名称

##  commands 
		commands.getstatusoutput('id')       # 返回元组(状态,标准输出)
		commands.getoutput('id')             # 只返回执行的结果, 忽略返回值
		commands.getstatus('file')           # 返回ls -ld file执行的结果
			
##  文件和目录管理 
		import shutil
		shutil.copyfile('data.db', 'archive.db')             # 拷贝文件
		shutil.move('/build/executables', 'installdir')      # 移动文件或目录

##  文件通配符 
		import glob
		glob.glob('*.py')    # 查找当前目录下py结尾的文件
##  随机模块 
		import random
		random.choice(['apple', 'pear', 'banana'])   # 随机取列表一个参数
		random.sample(xrange(100), 10)  # 不重复抽取10个
		random.random()                 # 随机浮点数
		random.randrange(6)             # 随机整数范围

##  解压缩 
		gzip压缩
			import gzip
			f_in = open('file.log', 'rb')
			f_out = gzip.open('file.log.gz', 'wb')
			f_out.writelines(f_in)
			f_out.close()
			f_in.close()

		gzip压缩1
			File = 'xuesong_18.log'
			g = gzip.GzipFile(filename="", mode='wb', compresslevel=9, fileobj=open((r'%s.gz' %File),'wb'))
			g.write(open(r'%s' %File).read())
			g.close()

		gzip解压
			g = gzip.GzipFile(mode='rb', fileobj=open((r'xuesong_18.log.gz'),'rb'))
			open((r'xuesong_18.log'),'wb').write(g.read())

		压缩tar.gz
			import os
			import tarfile
			tar = tarfile.open("/tmp/tartest.tar.gz","w:gz")   # 创建压缩包名
			for path,dir,files in os.walk("/tmp/tartest"):     # 递归文件目录
				for file in files:
					fullpath = os.path.join(path,file)
					tar.add(fullpath)                          # 创建压缩包
			tar.close()
		解压tar.gz
			import tarfile
			tar = tarfile.open("/tmp/tartest.tar.gz")
			#tar.extract("/tmp")                           # 全部解压到指定路径
			names = tar.getnames()                         # 包内文件名
			for name in names:
				tar.extract(name,path="./")                # 解压指定文件
			tar.close()

		zip压缩
			import zipfile,os
			f = zipfile.ZipFile('filename.zip', 'w' ,zipfile.ZIP_DEFLATED)    # ZIP_STORE 为默认表不压缩. ZIP_DEFLATED 表压缩
			#f.write('file1.txt')                              # 将文件写入压缩包
			for path,dir,files in os.walk("tartest"):          # 递归压缩目录
				for file in files:
					f.write(os.path.join(path,file))           # 将文件逐个写入压缩包         
			f.close()

		zip解压
			if zipfile.is_zipfile('filename.zip'):        # 判断一个文件是不是zip文件
				f = zipfile.ZipFile('filename.zip')
				for file in f.namelist():                 # 返回文件列表
					f.extract(file, r'/tmp/')             # 解压指定文件
				#f.extractall()                           # 解压全部
				f.close()

##  时间 
		import time
		time.time()                          # 时间戳[浮点]
		time.localtime()[1] - 1              # 上个月
		int(time.time())                     # 时间戳[整s]
		tomorrow.strftime('%Y%m%d_%H%M')     # 格式化时间
		time.strftime('%Y-%m-%d_%X',time.localtime( time.time() ) )              # 时间戳转日期
		time.mktime(time.strptime('2012-03-28 06:53:40', '%Y-%m-%d %H:%M:%S'))   # 日期转时间戳
		
```

		判断输入时间格式是否正确
			#encoding:utf8
			import time
			while 1:
				atime=raw_input('输入格式如[14.05.13 13:00]:')
				try:
					btime=time.mktime(time.strptime('%s:00' %atime, '%y.%m.%d %H:%M:%S'))
					break
				except:
					print '时间输入错误,请重新输入，格式如[14.05.13 13:00]'


		上一个月最后一天
			import datetime
			lastMonth=datetime.date(datetime.date.today().year,datetime.date.today().month,1)-datetime.timedelta(1)
			lastMonth.strftime("%Y/%m")

		前一天
			(datetime.datetime.now() + datetime.timedelta(days=-1) ).strftime('%Y%m%d')

		两日期相差天数

			import datetime
			d1 = datetime.datetime(2005, 2, 16)
			d2 = datetime.datetime(2004, 12, 31)
			(d1 - d2).days

		向后加10个小时

			import datetime
			d1 = datetime.datetime.now()
			d3 = d1 + datetime.timedelta(hours=10)
			d3.ctime()

```
## hash 
```

		import md5
		m = md5.new('123456').hexdigest()
		
		import hashlib
		m = hashlib.md5()
		m.update("Nobody inspects")    # 使用update方法对字符串md5加密
		m.digest()                     # 加密后二进制结果
		m.hexdigest()                  # 加密后十进制结果
		hashlib.new("md5", "string").hexdigest()               # 对字符串加密
		hashlib.new("md5", open("file").read()).hexdigest()    # 查看文件MD5值


```
##  隐藏输入密码 
		import getpass
		passwd=getpass.getpass()

	string打印a-z
		import string
		string.lowercase       # a-z小写
		string.uppercase       # A-Z大小

##  paramiko [ssh客户端] 
安装
```

			sudo apt-get install python-setuptools 
			easy_install
			sudo apt-get install python-all-dev
			sudo apt-get install build-essential

```
###  paramiko实例(账号密码登录执行命令) 
```

			#!/usr/bin/python
			#ssh
			import paramiko
			import sys,os

			host = '10.152.15.200'
			user = 'peterli'
			password = '123456'

			s = paramiko.SSHClient()                                 # 绑定实例
			s.load_system_host_keys()                                # 加载本地HOST主机文件
			s.set_missing_host_key_policy(paramiko.AutoAddPolicy())  # 允许连接不在know_hosts文件中的主机
			s.connect(host,22,user,password,timeout=5)               # 连接远程主机
			while True:
					cmd=raw_input('cmd:')
					stdin,stdout,stderr = s.exec_command(cmd)        # 执行命令
					cmd_result = stdout.read(),stderr.read()         # 读取命令结果
					for line in cmd_result:
							print line,
			s.close()


```
###  paramiko实例(传送文件) 
```

			#!/usr/bin/evn python
			import os
			import paramiko
			host='127.0.0.1'
			port=22
			username = 'peterli'
			password = '123456'
			ssh=paramiko.Transport((host,port))
			privatekeyfile = os.path.expanduser('~/.ssh/id_rsa') 
			mykey = paramiko.RSAKey.from_private_key_file( os.path.expanduser('~/.ssh/id_rsa'))   # 加载key 不使用key可不加
			ssh.connect(username=username,password=password)           # 连接远程主机
			# 使用key把 password=password 换成 pkey=mykey
			sftp=paramiko.SFTPClient.from_transport(ssh)               # SFTP使用Transport通道
			sftp.get('/etc/passwd','pwd1')                             # 下载 两端都要指定文件名
			sftp.put('pwd','/tmp/pwd')                                 # 上传
			sftp.close()
			ssh.close()

		paramiko实例(密钥执行命令)

			#!/usr/bin/python
			#ssh
			import paramiko
			import sys,os
			host = '10.152.15.123'
			user = 'peterli'
			s = paramiko.SSHClient()
			s.load_system_host_keys()
			s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
			privatekeyfile = os.path.expanduser('~/.ssh/id_rsa')             # 定义key路径
			mykey = paramiko.RSAKey.from_private_key_file(privatekeyfile)
			# mykey=paramiko.DSSKey.from_private_key_file(privatekeyfile,password='061128')   # DSSKey方式 password是key的密码
			s.connect(host,22,user,pkey=mykey,timeout=5)
			cmd=raw_input('cmd:')
			stdin,stdout,stderr = s.exec_command(cmd)
			cmd_result = stdout.read(),stderr.read()
			for line in cmd_result:
					print line,
			s.close()

```

###  ssh并发(Pool控制最大并发) 
```

			#!/usr/bin/env python
			#encoding:utf8
			#ssh_concurrent.py

			import multiprocessing
			import sys,os,time
			import paramiko

			def ssh_cmd(host,port,user,passwd,cmd):
				msg = "-----------Result:%s----------" % host

				s = paramiko.SSHClient()
				s.load_system_host_keys()
				s.set_missing_host_key_policy(paramiko.AutoAddPolicy())
				try:
					s.connect(host,22,user,passwd,timeout=5) 
					stdin,stdout,stderr = s.exec_command(cmd)

					cmd_result = stdout.read(),stderr.read()
					print msg
					for line in cmd_result:
							print line,

					s.close()
				except paramiko.AuthenticationException:
					print msg
					print 'AuthenticationException Failed'
				except paramiko.BadHostKeyException:
					print msg
					print "Bad host key"	

			result = []
			p = multiprocessing.Pool(processes=20)
			cmd=raw_input('CMD:')
			f=open('serverlist.conf')
			list = f.readlines()
			f.close()
			for IP in list:
				print IP
				host=IP.split()[0]
				port=int(IP.split()[1])
				user=IP.split()[2]
				passwd=IP.split()[3]
				result.append(p.apply_async(ssh_cmd,(host,port,user,passwd,cmd)))

			p.close()

			for res in result:
				res.get(timeout=35)

```
##  扫描主机开放端口 
```

		#!/usr/bin/env python
		import socket
		def check_server(address,port):
			s=socket.socket()
			try:
				s.connect((address,port))
				return True
			except socket.error,e:
				return False

		if __name__=='__main__':
			from optparse import OptionParser
			parser=OptionParser()
			parser.add_option("-a","--address",dest="address",default='localhost',help="Address for server",metavar="ADDRESS")
			parser.add_option("-s","--start",dest="start_port",type="int",default=1,help="start port",metavar="SPORT")
			parser.add_option("-e","--end",dest="end_port",type="int",default=1,help="end port",metavar="EPORT")
			(options,args)=parser.parse_args()
			print 'options: %s, args: %s' % (options, args)
			port=options.start_port
			while(port<=options.end_port):
				check = check_server(options.address, port)
				if (check):
					print 'Port  %s is on' % port
				port=port+1      

```
## mysql 

```

	#apt-get install mysql-server
	#apt-get install python-MySQLdb
	help(MySQLdb.connections.Connection)      # 查看链接参数
	conn=MySQLdb.connect(host='localhost',user='root',passwd='123456',db='fortress',port=3306)    # 定义连接
	#conn=MySQLdb.connect(unix_socket='/var/run/mysqld/mysqld.sock',user='root',passwd='123456')   # 使用socket文件链接
	cur=conn.cursor()                                            # 定义游标
	conn.select_db('fortress')                                   # 选择数据库
	sqlcmd = 'insert into user(name,age) value(%s,%s)'           # 定义sql命令
	cur.executemany(sqlcmd,[('aa',1),('bb',2),('cc',3)])         # 插入多条值
	cur.execute('delete from user where id=20')                  # 删除一条记录
	cur.execute("update user set name='a' where id=20")          # 更细数据
	sqlresult = cur.fetchall()                                   # 接收全部返回结果
	conn.commit()                                                # 提交
	cur.close()                                                  # 关闭游标
	conn.close()                                                 # 关闭连接
	
	import MySQLdb
	def mydb(dbcmdlist):
		try:
			conn=MySQLdb.connect(host='localhost',user='root',passwd='123456',db='fortress',port=3306)
			cur=conn.cursor()
			
			cur.execute('create database if not exists fortress;')  # 创建数据库
			conn.select_db('fortress')                              # 选择数据库
			cur.execute('drop table if exists log;')                # 删除表
			cur.execute('CREATE TABLE log ( id BIGINT(20) NOT NULL AUTO_INCREMENT, loginuser VARCHAR(50) DEFAULT NULL, remoteip VARCHAR(50) DEFAULT NULL, PRIMARY KEY (id) );')  # 创建表
			
			result=[]
			for dbcmd in dbcmdlist:
				cur.execute(dbcmd)           # 执行sql
				sqlresult = cur.fetchall()   # 接收全部返回结果
				result.append(sqlresult)
			conn.commit()                    # 提交
			cur.close()
			conn.close()
			return result
		except MySQLdb.Error,e:
			print 'mysql error msg: ',e
	sqlcmd=[]
	sqlcmd.append("insert into log (loginuser,remoteip)values('%s','%s');" %(loginuser,remoteip))
	mydb(sqlcmd)

	sqlcmd=[]
	sqlcmd.append("select * from log;")
	result = mydb(sqlcmd)
	for i in result[0]:
		print i     

```
##   处理信号 
	信号的概念

		信号(signal): 进程之间通讯的方式，是一种软件中断。一个进程一旦接收到信号就会打断原来的程序执行流程来处理信号。
		发送信号一般有两种原因:
			1(被动式)  内核检测到一个系统事件.例如子进程退出会像父进程发送SIGCHLD信号.键盘按下control+c会发送SIGINT信号
			2(主动式)  通过系统调用kill来向指定进程发送信号
		操作系统规定了进程收到信号以后的默认行为，可以通过绑定信号处理函数来修改进程收到信号以后的行为，有两个信号是不可更改的 SIGTOP 和 SIGKILL
		如果一个进程收到一个SIGUSR1信号，然后执行信号绑定函数，第二个SIGUSR2信号又来了，第一个信号没有被处理完毕的话，第二个信号就会丢弃。
		进程结束信号 SIGTERM 和 SIGKILL 的区别:  SIGTERM 比较友好，进程能捕捉这个信号，根据您的需要来关闭程序。在关闭程序之前，您可以结束打开的记录文件和完成正在做的任务。在某些情况下，假如进程正在进行作业而且不能中断，那么进程可以忽略这个SIGTERM信号。

	常见信号
		kill -l      # 查看linux提供的信号

		SIGHUP  1          A     # 终端挂起或者控制进程终止
		SIGINT  2          A     # 键盘终端进程(如control+c)
		SIGQUIT 3          C     # 键盘的退出键被按下
		SIGILL  4          C     # 非法指令
		SIGABRT 6          C     # 由abort(3)发出的退出指令
		SIGFPE  8          C     # 浮点异常
		SIGKILL 9          AEF   # Kill信号  立刻停止
		SIGSEGV 11         C     # 无效的内存引用
		SIGPIPE 13         A     # 管道破裂: 写一个没有读端口的管道
		SIGALRM 14         A     # 闹钟信号 由alarm(2)发出的信号 
		SIGTERM 15         A     # 终止信号,可让程序安全退出 kill -15
		SIGUSR1 30,10,16   A     # 用户自定义信号1
		SIGUSR2 31,12,17   A     # 用户自定义信号2
		SIGCHLD 20,17,18   B     # 子进程结束自动向父进程发送SIGCHLD信号
		SIGCONT 19,18,25         # 进程继续（曾被停止的进程）
		SIGSTOP 17,19,23   DEF   # 终止进程
		SIGTSTP 18,20,24   D     # 控制终端（tty）上按下停止键
		SIGTTIN 21,21,26   D     # 后台进程企图从控制终端读
		SIGTTOU 22,22,27   D     # 后台进程企图从控制终端写
		
		缺省处理动作一项中的字母含义如下:
			A  缺省的动作是终止进程
			B  缺省的动作是忽略此信号，将该信号丢弃，不做处理
			C  缺省的动作是终止进程并进行内核映像转储(dump core),内核映像转储是指将进程数据在内存的映像和进程在内核结构中的部分内容以一定格式转储到文件系统，并且进程退出执行，这样做的好处是为程序员提供了方便，使得他们可以得到进程当时执行时的数据值，允许他们确定转储的原因，并且可以调试他们的程序。
			D  缺省的动作是停止进程，进入停止状况以后还能重新进行下去，一般是在调试的过程中（例如ptrace系统调用）
			E  信号不能被捕获
			F  信号不能被忽略

	Python提供的信号
		import signal
		dir(signal)
		['NSIG', 'SIGABRT', 'SIGALRM', 'SIGBUS', 'SIGCHLD', 'SIGCLD', 'SIGCONT', 'SIGFPE', 'SIGHUP', 'SIGILL', 'SIGINT', 'SIGIO', 'SIGIOT', 'SIGKILL', 'SIGPIPE', 'SIGPOLL', 'SIGPROF', 'SIGPWR', 'SIGQUIT', 'SIGRTMAX', 'SIGRTMIN', 'SIGSEGV', 'SIGSTOP', 'SIGSYS', 'SIGTERM', 'SIGTRAP', 'SIGTSTP', 'SIGTTIN', 'SIGTTOU', 'SIGURG', 'SIGUSR1', 'SIGUSR2', 'SIGVTALRM', 'SIGWINCH', 'SIGXCPU', 'SIGXFSZ', 'SIG_DFL', 'SIG_IGN', '__doc__', '__name__', 'alarm', 'default_int_handler', 'getsignal', 'pause', 'signal']

```

	绑定信号处理函数
		#encoding:utf8
		import os,signal
		from time import sleep
		def onsignal_term(a,b):
			print 'SIGTERM'      # kill -15
		signal.signal(signal.SIGTERM,onsignal_term)     # 接收信号,执行相应函数

		def onsignal_usr1(a,b):
			print 'SIGUSR1'      # kill -10
		signal.signal(signal.SIGUSR1,onsignal_usr1)

		while 1:
			print 'ID',os.getpid()
			sleep(10)

	通过另外一个进程发送信号
		import os,signal
		os.kill(16175,signal.SIGTERM)    # 发送信号，16175是绑定信号处理函数的进程pid，需要自行修改
		os.kill(16175,signal.SIGUSR1)

	父进程接收子进程结束发送的SIGCHLD信号
		#encoding:utf8
		import os,signal
		from time import sleep
		   
		def onsigchld(a,b):
			print '收到子进程结束信号'
		signal.signal(signal.SIGCHLD,onsigchld)
		   
		pid = os.fork()                # 创建一个子进程,复制父进程所有资源操作
		if pid == 0:                   # 通过判断子进程os.fork()是否等于0,分别同时执行父进程与子进程操作
		   print '我是子进程,pid是',os.getpid()
		   sleep(2)
		else:
			print '我是父进程,pid是',os.getpid()
			os.wait()      # 等待子进程结束

	接收信号的程序，另外一端使用多线程向这个进程发送信号，会遗漏一些信号
		#encoding:utf8
		import os
		import signal
		from time import sleep  
		import Queue
		QCOUNT = Queue.Queue()  # 初始化队列  
		def onsigchld(a,b):  
			` '收到信号后向队列中插入一个数字1 `'
			print '收到SIGUSR1信号'
			sleep(1)
			QCOUNT.put(1)       # 向队列中写入
		signal.signal(signal.SIGUSR1,onsigchld)   # 绑定信号处理函数
		while 1:
			print '我的pid是',os.getpid()
			print '现在队列中元素的个数是',QCOUNT.qsize()
			sleep(2)

		多线程发信号端的程序

			#encoding:utf8
			import threading
			import os
			import signal
			def sendusr1():
			print '发送信号'
				os.kill(17788, signal.SIGUSR1)     # 这里的进程id需要写前一个程序实际运行的pid
			WORKER = []
			for i in range(1, 7):                  # 开启6个线程
				threadinstance = threading.Thread(target = sendusr1)
				WORKER.append(threadinstance)  
			for i in WORKER:
				i.start()
			for i in WORKER:
				i.join()
			print '主线程完成'


```
##  缓存数据库 
###  python使用memcache 
```

		easy_install python-memcached   # 安装(python2.7+)
		import memcache
		mc = memcache.Client(['10.152.14.85:12000'],debug=True)
		mc.set('name','luo',60)
		mc.get('name')
		mc.delete('name1')
		
		保存数据

			set(key,value,timeout)      # 把key映射到value，timeout指的是什么时候这个映射失效
			add(key,value,timeout)      # 仅当存储空间中不存在键相同的数据时才保存
			replace(key,value,timeout)  # 仅当存储空间中存在键相同的数据时才保存

		获取数据

			get(key)                    # 返回key所指向的value
			get_multi(key1,key2,key3)   # 可以非同步地同时取得多个键值， 比循环调用get快数十倍

```
###  python使用mongodb 
```

		easy_install pymongo      # 安装(python2.7+)
		import pymongo
		connection=pymongo.Connection('localhost',27017)   # 创建连接
		db = connection.test_database                      # 切换数据库
		collection = db.test_collection                    # 获取collection
		# db和collection都是延时创建的，在添加Document时才真正创建

		文档添加, _id自动创建
			import datetime
			post = {"author": "Mike",
				"text": "My first blog post!",
				"tags": ["mongodb", "python", "pymongo"],
				"date": datetime.datetime.utcnow()}
			posts = db.posts
			posts.insert(post)
			ObjectId('...')

		批量插入
			new_posts = [{"author": "Mike",
				"text": "Another post!",
				"tags": ["bulk", "insert"],
				"date": datetime.datetime(2009, 11, 12, 11, 14)},
				{"author": "Eliot",
				"title": "MongoDB is fun",
				"text": "and pretty easy too!",
				"date": datetime.datetime(2009, 11, 10, 10, 45)}]
			posts.insert(new_posts)
			[ObjectId('...'), ObjectId('...')]
		
		获取所有collection
			db.collection_names()    # 相当于SQL的show tables
			
		获取单个文档
			posts.find_one()

		查询多个文档
			for post in posts.find():
				post

		加条件的查询
			posts.find_one({"author": "Mike"})

		高级查询
			posts.find({"date": {"$lt": "d"}}).sort("author")

		统计数量
			posts.count()

		加索引
			from pymongo import ASCENDING, DESCENDING
			posts.create_index([("date", DESCENDING), ("author", ASCENDING)])

		查看查询语句的性能
			posts.find({"date": {"$lt": "d"}}).sort("author").explain()["cursor"]
			posts.find({"date": {"$lt": "d"}}).sort("author").explain()["nscanned"]


```
###  python使用redis 
```

		https://pypi.python.org/pypi/redis
		pip install redis  OR easy_install redis
		import redis
		r = redis.StrictRedis(host='localhost', port=6379, db=0)
		r.set('foo', 'bar')
		r.get('foo')
		r.save()
		
		分片 # 没搞懂
			redis.connection.Connection(host='localhost', port=6379, db=0,  parser_class=<class 'redis.connection.PythonParser'>)
			redis.ConnectionPool( connection_class=<class 'redis.connection.Connection'>, max_connections=None, **connection_kwargs)


```
##  python使用kestrel队列 
```

		# pykestrel
		import kestrel

		q = kestrel.Client(servers=['127.0.0.1:22133'],queue='test_queue') 
		q.add('some test job') 
		job = q.get()    # 从队列读取工作
		job = q.peek()   # 读取下一份工作
		# 读取一组工作
		while True:
			job = q.next(timeout=10) # 完成工作并获取下一个工作，如果没有工作，则等待10秒
			if job is not None:
				try:
					# 流程工作
				except:
					q.abort() # 标记失败工作

		q.finish()  # 完成最后工作
		q.close()   # 关闭连接
		
		kestrel状态检查
			# kestrel支持memcache协议客户端
			#!/usr/local/bin/python
			# 10.13.81.125 22133  10000

			import memcache
			import sys
			import traceback

			ip="%s:%s" % (sys.argv[1],sys.argv[2])
			try:
				mc = memcache.Client([ip,])
				st=mc.get_stats()
			except:
				print "kestrel connection exception"
				sys.exit(2)

			if st:
				for s in st[0][1].keys():
					if s.startswith('queue_') and s.endswith('_mem_items'):
						num = int(st[0][1][s])
						if num > int(sys.argv[3]):
							print "%s block to %s" %(s[6:-6],num)
							sys.exit(2)
				print "kestrel ok!"
				sys.exit(0)
			else:
				print "kestrel down"
				sys.exit(2)

	python使用tarantool

		# pip install tarantool-queue

		from tarantool_queue import Queue
		queue = Queue("localhost", 33013, 0)     # 连接读写端口 空间0
		tube = queue.tube("name_of_tube")        # 
		tube.put([1, 2, 3])

		task = tube.take()
		task.data     # take task and read data from it
		task.ack()    # move this task into state DONE                     

```
