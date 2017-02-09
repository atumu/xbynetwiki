title: linux下的安装相关问题 

#  Linux下的安装相关问题 
##  ubuntu安装Python3 
ubuntu14.04默认安装了python3.4，那么12.04怎么安装呢？
参考http://askubuntu.com/questions/802279/how-to-install-python-3-4-5-from-apt
先[安装add-apt-repository:](http://askubuntu.com/questions/593433/error-sudo-add-apt-repository-command-not-found)
对于ubuntu12.04
sudo apt-get install python-software-properties
对于14.04:
sudo apt-get install software-properties-common

```

sudo add-apt-repository ppa:fkrull/deadsnakes
sudo apt-get update
sudo apt-get install python3.4

```
/usr/bin/python3指向/usr/bin/python3.5.所以你应该使用/usr/bin/python3.4来运行python3.4,当然你也可以改变这一策略：
设置Python 3.3为默认的命令：
rm /usr/bin/python3
ln -s /usr/bin/python3.4 /usr/bin/python3
