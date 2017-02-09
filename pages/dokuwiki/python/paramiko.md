title: paramiko 

#  SSH连接paramiko与Fabric 
##  paramiko 
paramiko是基于Python实现的SSH2远程安全连接，支持认证及密钥方法。可以实现远程命令执行，文件传输，中间SSH代理等功能，相对于Pexpect，封装层次更高。
#pip install Paramiko
#http://www.paramiko.org/
#demo:https://github.com/paramiko/paramiko/tree/master/demos

如果在linux环境，还需安装依赖：crypto,ecdsa，python3-devel。

paramiko包含两个核心组件:SSHClient类，SFTPClient类


###  密钥方式登录 

```

import paramiko, base64,getpass

paramiko.util.log_to_file('syslogin.log') #日志记录
try:
        key=paramiko.RSAKey.from_private_key_file('pk_path')
except paramiko.PasswordRequiredException:
        password = getpass.getpass('RSA key password: ')
        key = paramiko.RSAKey.from_private_key_file('pk_path', password)    # 需要口令的私钥
#key = paramiko.RSAKey(data=base64.decodestring('AAA...'))
client = paramiko.SSHClient()
# client.get_host_keys().add('ssh.example.com', 'ssh-rsa', key)
client.load_system_host_keys()#~/.ssh/known_hosts
client.connect('ssh.example.com', 22,username='strongbad', password='thecheat',pkey=key)
stdin, stdout, stderr = client.exec_command('ls')
# stdin, stdout, stderr=ssh.exec_command('sudo su')
# stdin.write('123456')
for line in stdout:
    print('... ' + line.strip('\n'))
#使用send
# cmds=['sudo su\n', 'cd /var/log\n', 'ls\n'] #利用send函数发送cmd到SSH server，添加'\n'做回车来执行shell命令。注意不同的情况，如果执行完telnet命令后，telnet的换行符是\r\n
# ssh=s.invoke_shell() #在SSH server端创建一个交互式的shell，且可以按自己的需求配置伪终端，可以在invoke_shell()函数中添加参数配置。
# for cmd in cmds:
#         time.sleep(1)
#         ssh.send(cmd) #利用send函数发送cmd到SSH server，
#         out = ssh.recv(1024) #.recv(bufsize)通过recv函数获取回显。
#         print out
client.close()

```
###  用户名密码方式登录 
```

#####################################################################################
import paramiko

paramiko.util.log_to_file('syslogin.log') #日志记录
client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
client.connect('192.168.8.248', 22, username='root', password='password', timeout=4)
stdin, stdout, stderr = client.exec_command('ls -l')
#print(stdout.read())
for line in stdout.readlines():
  print(line)
client.close()

```

SSHClient方法参数说明
connect(hostname, port=22, username=None, password=None, pkey=None, key_filename=None, timeout=None, allow_agent=True, look_for_keys=True, compress=False, sock=None, gss_auth=False, gss_kex=False, gss_deleg_creds=True, gss_host=None, banner_timeout=None)
  * pkey-私钥类型
  * key_filename-str or list(str) 私钥文件或其列表
  * timeout-以秒为单位
  * allow_agent-为False时禁用连接到SSH代理
  * look_for_keys-为False时禁用在~/.ssh中搜索私钥文件

exec_command(command, bufsize=-1, timeout=None, get_pty=False)
command-字符串

load_system_host_keys(filename=None)指定公钥文件,默认为~/.ssh/known_hosts

**set_missing_host_key_policy**(policy):设置连接的远程主机没有本地主机密钥时的策略。目前支持三种： RejectPolicy (the default), **AutoAddPolicy**, WarningPolicy


###  上传与下载文件 
```

#上传批量文件到远程主机
import paramiko
import os
import datetime

hostname = '74.63.229.*'
username = 'root'
password = 'abc123'
port = 22
local_dir = '/tmp/'
remote_dir = '/tmp/test/'
if __name__ == "__main__":
    #    try:
    t = paramiko.Transport((hostname, port))
    t.connect(username=username, password=password)
    sftp = paramiko.SFTPClient.from_transport(t)
    #        files=sftp.listdir(dir_path)
    files = os.listdir(local_dir)
    for f in files:
        '#########################################'
        'Beginning to upload file %s ' % datetime.datetime.now()
        'Uploading file:', os.path.join(local_dir, f)
        # sftp.get(os.path.join(dir_path,f),os.path.join(local_path,f))
        sftp.put(os.path.join(local_dir, f), os.path.join(remote_dir, f))
        'Upload file success %s ' % datetime.datetime.now()
    t.close()

```
SFTPClient类相关方法说明：

#参考http://www.cnblogs.com/yangshine/p/5709510.html

##  Fabric 
Fabric是基于paramiko的基础上做了一层更高的封装，操作起来更加方便。
官网：http://www.fabfile.org/index.html
github:https://github.com/fabric/fabric/

依赖crypto，paramiko.注意：fabric目前不支持Python3.不过github上有个支持py3的版本https://github.com/mathiasertl/fabric/

在windows上的安装：
1、安装pycrypto.
有几种方式安装：
A.win7下安装 MSVC2010,然后通过pip install pycrypto编译安装.[编译教程](http://www.devdungeon.com/content/installing-pycryptoparamiko-python3-x64-windows)
B.选择别人编译好的。
[pycrypto-for-python-3-2及以下](http://www.voidspace.org.uk/python/modules.shtml#pycrypto)
[pycrypto-for-python-3-4](https://flintux.wordpress.com/2014/04/30/pycrypto-for-python-3-4-on-windows-7-64bit/)
[pycrypto-for-python-3-4](https://github.com/axper/python3-pycrypto-windows-installer)

2、支持Python3的版本安装：pip install Fabric3

```

from fabric.api import run
def host_type():
    run('uname -s')

```
通过fab命令执行。-f指定文件，-H指定主机列表.
```

$ fab -f fabfile.py -H localhost,linuxbox host_type

```
**fab参数说明：**
-f 指定入口3文件
-g 指定网关设备(中转，堡垒机)IP
-H 指定目标主机，多个用","分割
-P 异步运行多主机任务
-R 指定角色，以角色来区分机组
-t 设备连接超时时间，秒
-T 远程主机命令执行超时时间，秒
-w 当命令执行失败，发出警告，而不是终止任务。
当然我们完全可以在代码中设定这些选项值，而无需在命令行指定。如下：全局属性设定
env对象的作用是定义fabfile的全局设定，支持多个属性及自定义属性。
  * env.hosts,定义目标主机，列表
  * env.exclude_hosts,排除主机，列表
  * env.user,定义用户名,str
  * env.port , 定义端口，str
  * env.password,定义密码，str
  * env.passwords,字典，但是形式如下：env.passwords={ 'root@192.168.1.21:22':'123456','root@192.168.2.21:22':'1234'}
  * env.key_filename=None 指定SSH密钥文件,str or list
  * env.gateway指定网关设备(中转，堡垒机)IP,str
  * env.roledefs定义角色分组，字典：env.roledefs={ 'web':['192.168.1.21','192.168.1.23'],'db':['192.168.1.22','192.168.1.24']}
  * env.parallel=False是否并发执行任务
  * env.path=' ' 定义在run/sudo/local使用的$PATH环境变量
  * env.command_timeout=None
  * env.timeout=10
  * env.shell="/bin/bash -l -c"
  * env.ssh_config_path="$HOME/.ssh/config"
  * env.sudo_password=None
  * env.sudo_passwords={}
  * env.use_ssh_config=False
  * env.warn_only=False,如果为True,当操作遇到错误时，发出警告并继续执行，而不是终止
  * env.变量名 自定义变量

例如：
```

@roles('web')
def webtask():
	run('/etc/init.d/nginx start')
@roles('db')
def dbtask():
	run('/etc/init.d/mysql start')

@roles('web','db')
def publicstask():
	run('uptime')
def deploy():
	execute(webtask)
	execute(dbtask)
	execute(publictask)

```
然后终端执行命令就可以了
```

$ fab deploy

```

命令行传参：
```

def hello(name="world"):
    print("Hello %s!" % name)

```
```

$ fab hello:name=Jeff
Hello Jeff!
Done.

```


##  常用API 
**fabric.api**模块:
  * local,执行本地命令,如local('uname -s')
  * lcd,切换本地目录,如lcd('/home')
  * cd,切换远程目录
  * run,执行远程命令
  * sudo,sudo方式执行远程命令
  * put,上传文件到远程主机 put('/home/aaa','/home/xby/aaa')
  * get,从远程主机下载文件到本地 get('/opt/bbb','/home/bbb')
  * prompt,获取用户输入
  * confirm，获得提示信息确认,如confirm('Continue[Y/N]?')
  * reboot,重启远程主机，如reboot()

  * @task函数装饰器，标识函数为fab可调用的，否则对fab不可见
  * @runs_once,标识函数只会执行一次，不受多台主机影响。
  * @roles,表示函数执行时的主机角色
  * @parallel(pool_size=)
  * @with_settings()


**fabric.contrib.console.confirm(question, default=True)** 用户输入Y/n,返回True/False

示例1：查看本地与远程主机信息：
```

from fabric.api import *

env.user='root'
env.hosts=['192.168.1.2','192.168.1.3']
env.password='123'

@runs_once #即使有多台主机，但它只会执行一次
def local_task():
	local('uname -a')
def remote_task():
	with cd("/data/logs"): #这个with的作用是让后面的表达式语句继承当前的状态，实现"cd /data/logs && ls -l"的效果。
		run("ls -l")

```
$ fab -f sample.py local_task
$ fab -f sample.py remote_task

示例2：动态获取远程目录
```

from fabric.api import *
from fabric.contrib.console import confirm

env.user='root'
env.hosts=['192.168.1.2','192.168.1.3']
env.password='123'

@runs_once
def input_raw():
	return prompt("please input dir name:",default='/home')
def worktask(dirname):
	run("ls -l "+dirname)
@task
def go():
	dirname=input_raw()
	worktask(dirname)

```

示例3：网关模式文件上传与执行
其实只要定义好env.gateway的ip就行了。相比paramiko确实简化了不少。
```

from fabric.api import *
from fabric.contrib.console import confirm
from fabric.context_managers import *

env.user='root'
env.hosts=['192.168.1.2','192.168.1.3']
env.password='123'
env.gateway='192.168.22.2'

lpath='/home/install/lnmp.tar.gz'
rpath='/tmp/install'

@task
def put_task():
  	run("mkdir -p /tmp/install")
      with settings(warn_only=True): #put出现异常时，发出警告，继续执行，不要终止。
        result=put(lpath,rpath) #上传
       if result.failed and not confirm("put failed,continue[Y/N]?"):
        abort("Aborting")
@task 
def run_task():
  	with cd("/tmp/install"):
     	run("tar -zxvf lnmp.tar.gz")
       with cd("lnmp"):
        run("./install.sh")
        
@task go():
  put_task()
  run_task()
  

```
##  多彩输出 
fabric.colors.blue(text, bold=False)
fabric.colors.cyan(text, bold=False)
fabric.colors.green(text, bold=False)
fabric.colors.magenta(text, bold=False)
fabric.colors.red(text, bold=False)
fabric.colors.white(text, bold=False)
fabric.colors.yellow(text, bold=False)
```

from fabric.colors import red, green
print(red("This sentence is red, except for " + green("these words, which are green") + "."))

```

##  示例-Fabric部署Flask应用 
示例1：它可以把当前的源代码上传至服务器，并安装到一个预先存在 的 virtual 环境:
```

from fabric.api import *
# 使用远程命令的用户名
env.user = 'appuser'
# 执行命令的服务器
env.hosts = ['server1.example.com', 'server2.example.com']
def pack():
    # 创建一个新的分发源，格式为 tar 压缩包
    local('python setup.py sdist --formats=gztar', capture=False)
def deploy():
    # 定义分发版本的名称和版本号
    dist = local('python setup.py --fullname', capture=True).strip()
    # 把 tar 压缩包格式的源代码上传到服务器的临时文件夹
    put('dist/%s.tar.gz' % dist, '/tmp/yourapplication.tar.gz')
    # 创建一个用于解压缩的文件夹，并进入该文件夹
    run('mkdir /tmp/yourapplication')
    with cd('/tmp/yourapplication'):
        run('tar xzf /tmp/yourapplication.tar.gz')
        # 现在使用 virtual 环境的 Python 解释器来安装包
        run('/var/www/yourapplication/env/bin/python setup.py install')
    # 安装完成，删除文件夹
    run('rm -rf /tmp/yourapplication /tmp/yourapplication.tar.gz')
    # 最后 touch .wsgi 文件，让 mod_wsgi 触发应用重载
    run('touch /var/www/yourapplication.wsgi')

```
还可以参考廖雪峰的这篇[入门](http://www.liaoxuefeng.com/article/001373892650475818672edc83c4c978a45195eab8dc753000)