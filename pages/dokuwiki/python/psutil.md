title: psutil 

#  系统性能信息与进程管理库psutil 
psutil：系统性能信息与进程管理库。用来管理进程信息以及CPU, memory, disks, network信息。主要用于系统资源的监控，分析，以及对进程
进行一定的管理。通过psutil可以实现如ps，top，lsof，netstat，ifconfig， who，df，kill，free，nice，ionice，iostat，iotop，uptime，pidof，tty，taskset，pmap。在Linux，windows，OSX等系统上工作

官网：http://pythonhosted.org/psutil/
Demo：https://github.com/giampaolo/psutil/tree/master/scripts
目前版本4.4.0

pip install psutil

##  系统相关 
###  CPU相关 
psutil.cpu_times(precpu=False)返回系统CPU运行时间的元组，时间为秒。
psutil.cpu_percent(interval=None, percpu=False)
返回一个浮点书，代表当前cpu的利用率的百分比，包括sy+user. 当interval为0或者None时，表示的是interval时间内的sys的利用率。当percpu为True返回是每一个cpu的利用率。
psutil.cpu_times_percent(interval=None, percpu=False)
psutil.cpu_count(logical=True)

###  Memory 
psutil.virtual_memory()返回一个内存信息的元组，大小为字节
psutil.swap_memory() 返回系统的swap信息的元组，大小为字节

###  Disks 
psutil.disk_partitions(all=False)
返回所有挂载的分区的信息的列表，列表中的每一项类似于df命令的格式输出，包括分区，挂载点，文件系统格式，挂载参数等，会忽略掉/dev/shm,/proc/filesystem等

psutil.disk_usage(path)
返回硬盘，分区或者目录的使用情况，单位字节。如果不存在会报“OSError”错误。

psutil.disk_io_counters(perdisk=False)返回当前磁盘的io情况

###  Network 
psutil.net_connections(kind='inet')返回系统的整个socket连接的信息，可以选择查看哪些类型的连接信息，类似于netstat命令

psutil.net_io_counters(pernic=False)返回整个系统的网络信息
bytes_sent: 发送的字节数
bytes_recv: 接收的字节数
packets_sent: 发送到数据包的个数
packets_recv: 接受的数据包的个数
。。
**如果 pernic值为True，会显示具体各个网卡的信息。**
```

>>> import psutil
>>> psutil.net_io_counters()
snetio(bytes_sent=14508483, bytes_recv=62749361, packets_sent=84311, packets_recv=94888, errin=0, errout=0, dropin=0, dropout=0)
>>>
>>> psutil.net_io_counters(pernic=True)
{'lo': snetio(bytes_sent=547971, bytes_recv=547971, packets_sent=5075, packets_recv=5075, errin=0, errout=0, dropin=0, dropout=0),
'wlan0': snetio(bytes_sent=13921765, bytes_recv=62162574, packets_sent=79097, packets_recv=89648, errin=0, errout=0, dropin=0, dropout=0)}

```

###  Other system info 
psutil.users()返回当前系统用户登录信息的元祖

psutil.boot_time() 返回开机启动的时间
```

import psutil, datetime
psutil.boot_time()
boot=datetime.datetime.fromtimestamp(psutil.boot_time()).strftime("%Y-%m-%d %H:%M:%S")
print(boot)  #2016-11-02 18:51:24

```
##  进程管理 
psutil.pids()返回当前运行的进程pid列表
psutil.pid_exists(pid)是否存在次pid.
psutil.process_iter()返回一个包含Process对象的迭代器。
```

import psutil
for proc in psutil.process_iter():
    try:
        pinfo = proc.as_dict(attrs=['pid', 'name'])
    except psutil.NoSuchProcess:
        pass
    else:
        print(pinfo)

```
psutil.wait_procs(procs, timeout=None, callback=None) Convenience function which waits for a list of Process instances to terminate.
###  Exceptions 
class psutil.Error基础异常，psutil的其他异常都继承这个
class psutil.NoSuchProcess(pid, name=None, msg=None)当进程不在进程列表中，或者进程不存在时触发。
class psutil.AccessDenied(pid=None, name=None, msg=None)没有权限时，被触发
class psutil.TimeoutExpired(seconds, pid=None, name=None, msg=None)当Process.wait() 超时，并且Process 一直在运行时.
###  Process class 
class psutil.Process(pid=None)
Process类是一个带有pid的进程。如果没有指定pid，则默认的进程为os.getpid()所得进程。Process会触发NoSuchProcess（当进程不存在时）和AccessDenied异常，
Process是通过pid绑定的。如果在一个Process实例，在psutil运行中pid进程死掉，而这个pid又绑定给了别的新的进程。为了保证Process的安全性可以通过pid+createion time方式来确认进程是否是同一个。
该类的方法或属性
pid进程的PID
ppid()父进程pid. On Windows the return value is cached after first call.
name()进程名.
exe()进程运行命令的绝对路径。
cmdline()The command line this process has been called with.
create_time()进程创建时间
```

import psutil, datetime
p = psutil.Process()
p.create_time()
datetime.datetime.fromtimestamp(p.create_time()).strftime("%Y-%m-%d %H:%M:%S")

```
**as_dict(attrs=None, ad_value=None) 返回进程信息的哈希字典的实用方法**，attrs指定的值必须是Process的属性值，例如（['cpu_times','name']）

parent()返回父进程，如果不存在父进程，则返回None。
status()进程当前运行状态，string形式。
cwd()进程运行的所在的目录
username()哪个用户下运行的进程
uids()返回real=uid，effective，saved用户的uid

gids() UNIX下可用
terminal() UNIX下可用
nice(value=None)UNIX下可用 获取或者设置进程的nice值。

is_running()判断进程是否存活
send_signal(signal)发送新号给进程，This is the same as os.kill(pid, sig)，On Windows only SIGTERM is valid
terminate()
kill()
wait(timeout=None)

###  Popen class 
class psutil.Popen(*args, * *kwargs)
###  Constants 
psutil.STATUS_RUNNING
psutil.STATUS_SLEEPING
psutil.STATUS_DISK_SLEEP
psutil.STATUS_STOPPED
psutil.STATUS_TRACING_STOP
psutil.STATUS_ZOMBIE
psutil.STATUS_DEAD
psutil.STATUS_WAKE_KILL
psutil.STATUS_WAKING
psutil.STATUS_IDLE
psutil.STATUS_LOCKED
psutil.STATUS_WAITING
