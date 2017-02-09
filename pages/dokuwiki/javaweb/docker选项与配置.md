title: docker选项与配置 

#  docker选项与配置 
**4.SELinux 或 AppArmor**
Linux的内部安全模块，例如通过访问控制的安全策略来配置安全增强型Linux（SELinux）和AppArmor，从而实现强制性的访问控制（MAC）一套有限的系统资源的限制进程，如果先前已经安装和配置过SELinux，那么它可以使用setenforce 1在容器中被激活。Docker程序的SELinux支持是默认无效的，并且需要使用—selinux功能来被激活。通过使用新增的—security-opt来加载SELinux或者AppArmor的策略对容器的标签限制进行配置。该功能已经在Docker版本1.3[9]中介绍过。例如：
```

docker run --security-opt=secdriver:name:value -i -t centos bash

``` 
**5.守护特权**
不要使用--privileged命令行选项。这本来允许容器来访问主机上的所有设备，并为容器提供一个特定的LSM配置（例如SELinux或AppArmor），而这将给予如主机上运行的程序同样水平的访问。避免使用--privileged有助于减少主机泄露的攻击面和潜力。然而，这并不意味着程序将没有优先权的运行，当然这些优先权在最新的版本中还是必须的。发布新程序和容器的能力只能被赋予到值得信任的用户上。通过利用-u选项尽量减少容器内强制执行的权限。例如：
docker run -u <username> -it <container_name> /bin/bash 
Docker组的任何用户部分可能最终从容器中的主机上获得根源。

**6.cgroups**
为了防止通过系统资源耗尽的DDoS攻击，可以使用特定的命令行参数被来**进行一些资源限制。**

CPU使用率：
docker run -it --rm --cpuset=0,1 -c 2 ... 
内存使用：
docker run -it --rm -m 128m ... 
存储使用：
docker -d --storage-opt dm.basesize=5G 
磁盘I/O：
目前不支持Docker。BlockIO*属性可以通过systemd暴露，并且在支持操作系统中被用来控制磁盘的使用配额。

参考：http://cloud.51cto.com/art/201501/464287.htm