title: python学习笔记14 

#  Python学习笔记之安装第三方库 
安装第三方库
要是用第三方库，第一步就是要安装，在本地安装完毕，就能如同标准库一样使用了。其安装方法如下：
##  方法一：利用源码安装 
在github.com网站可以下载第三方库的源码（或者其它途径），得到源码之后，在本地安装。
一般情况，得到的码格式大概都是 zip 、 tar.zip、 tar.bz2格式的压缩包。解压这些包，进入其文件夹，**通常会看见一个 setup.py 的文件**。如果是Linux或者Mac(我是用ubuntu，特别推荐哦)，就在这里运行shell，执行命令：
```

python setup.py install

```
如果用的是windows，需要打开命令行模式，执行上述指令即可。
如此，就能把这个第三库安装到系统里。具体位置，要视操作系统和你当初安装python环境时设置的路径而定。
默认条件下,windows是在C:\Python2.7\Lib\site-packages，Linux在/usr/local/lib/python2.7/dist-packages（这个只是参考，不同发行版会有差别，具体请读者根据自己的操作系统，自己找找），Mac在 /Library/Python/2.7/site-packages。
有安装就要有卸载，卸载所安装的库非常简单，只需要到相应系统的site-packages目录，直接删掉库文件即卸载。

##  方法二：pip 
用源码安装，不是我推荐的，我推荐的是用第三方库的管理工具安装。
有一个网站，是专门用来存储第三方库的，所有在这个网站上的，都能用` pip `或者easy_install这种安装工具来安装。
这个网站的地址：https://pypi.python.org/pypi
首先，要安装pip（python官方推荐这个，我当然要顺势了，所以，就只介绍并且后面也只使用这个工具）。如果读者跟我一样，用的是ubuntu或者其它某种Linux，基本不用这个操作，在安装操作系统的时候已经默认把这个东西安装好了。如果因为什么原因，没有安装，可以使用如下方法：
```

Debian and Ubuntu:
sudo apt-get install python-pip
Fedora and CentOS:
sudo yum install python-pip

```
当然，也可以这里下载文件[get-pip.py](https://bootstrap.pypa.io/get-pip.py)，然后执行python get-pip.py来安装。这个方法也适用于windows。
pip安装好了。如果要安装第三方库，只需要执行pip install XXXXXX（XXXXXX代表第三方库的名字）即可。
当第三方库安装完毕，接下来的使用就如同前面标准库一样。

**用依赖文件安装依赖：**
```

pip install -r requirements.txt

```

pip自动生成requirements.txt依赖关系清单
```

pip freeze > requirements.txt

```