title: linuxssh1 

#  LinuxSSH登录配置 
正常密码登录
```

ssh root@192.168.0.1

```

免密码SSH密钥对认证:
1、创建SSH密钥对:
```

ssh-keygen.

```
输入相关信息后，会在本地产生两个文件: ~/.ssh/id_rsa.pub(公钥)和~/.ssh/id_rsa(私钥)。
2、复制**公钥**到远程服务器:
```

scp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys demo@192.168.0.1 (远程服务器必须事先存在~/.ssh文件夹)

```
3、登录远程服务器，修改文件权限
```

chown -R demo:demo ~/.ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

```

禁止密码登录
vim /etc/ssh/sshd_config
找到PaswordAuthentication设置为no

禁止root用户登录
vim /etc/ssh/sshd_config
找到PermitRootLogin设置为no

让改动生效 sudo service ssh restart 或者 centos下:sudo systemctl restart sshd.service

建议安装防火墙:ubuntu下UFW,Centos下iptables
