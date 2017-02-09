title: pip_更换软件镜像源 

#  pip 更换软件镜像源 
pipy国内镜像目前有：
http://pypi.douban.com/  豆瓣
http://pypi.hustunique.com/  华中理工大学
http://pypi.sdutlinux.org/  山东理工大学
http://pypi.mirrors.ustc.edu.cn/  中国科学技术大学

临时使用
pip install pythonModuleName -i http://pypi.douban.com --trusted-host=pypi.douban.com

**修改配置文件**
pip源配置文件可以放置的位置：https://pip.pypa.io/en/stable/user_guide/
**个人电脑:**
  * On Unix  is: $HOME/.config/pip/pip.conf which respects the XDG_CONFIG_HOME environment variable.
  * On Mac OS X is $HOME/Library/Application Support/pip/pip.conf.
  * On Windows is **%APPDATA%\pip\pip.ini.**
There are also a legacy per-user configuration file which is also respected, these are located at:
On Unix and Mac OS X the configuration file is: $HOME/.pip/pip.conf
On Windows the configuration file is: %HOME%\pip\pip.ini
You can set a custom path location for this config file using the environment variable **PIP_CONFIG_FILE**.

**virtualenv:**
  * On Unix and Mac OS X the file is $VIRTUAL_ENV/pip.conf
  * On Windows the file is: %VIRTUAL_ENV%\pip.ini

**服务端**
  * On Unix the file may be located in /etc/pip.conf. Alternatively it may be in a "pip" subdirectory of any of the paths set in the environment variable XDG_CONFIG_DIRS (if it exists), for example /etc/xdg/pip/pip.conf.
  * On Mac OS X the file is: /Library/Application Support/pip/pip.conf
  * On Windows XP the file is: C:\Documents and Settings\All Users\Application Data\pip\pip.ini
  * On Windows 7 and later the file is hidden, but writeable at C:\ProgramData\pip\pip.ini

配置文件查找顺序：
  * Firstly the site-wide file is read, then
  * The per-user file is read, and finally
  * The virtualenv-specific file is read.
pip配置的主要一些配置：
```

[global]
index-url = http://pypi.douban.com/simple #豆瓣源，可以换成其他的源
trusted-host = pypi.douban.com            #添加豆瓣源为可信主机，要不然可能报错
#disable-pip-version-check = true          #取消pip版本检查，排除每次都报最新的pip
timeout = 120

```
在pip.conf中，添加以上内容，就修改了默认的软件源。以后pip命令会直接从制定的软件源安装软件。

问题
http://pypi.douban.com不提供HTTPS连接，关心安全问题的话，请三思后再决定是否使用。` 这个问题也导致在配置时，需要添加- -trusted-host参数 `，假设软件源是安全的。
虽然修改了软件源**，但是pip search命令还是不能使用的，因为搜索软件使用的协议与安装软件不同。pip search基于xmlrpclib实现，pip install基于urllib2实现。**同样地，对pip search设置代理，也是不起作用的。

修正：
**pypi.douban.com现已提供https形式访问。即https://pypi.doubanio.com/**

参考：http://www.jianshu.com/p/785bb1f4700d
http://topmanopensource.iteye.com/blog/2004853