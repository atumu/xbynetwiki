title: tortoisegit配置ssh提交github 

#  tortoisegit配置SSH提交github 
在使用Git与tortoisegit的时候，指定远程版本库的地址有2种方式：
  * **使用https方式**的git地址非常直接（https://xxx.oschina.net/xxx.git），基本上什么都不需要配置，不管是git bash还是tortoisegit都能完美使用，` 但是每次需要连接远程服务器时，都要提示我输入用户名与密码 `，非常不爽；
  * **使用ssh方式**的git地址非常爽快（git@git.oschina.net:xxxx/xxx.git），` 不需要输入密码，但是需要配置 `。
第一种方式没啥说的，第二种方式的应用，我配置的时候出现了一个问题：
配置了tortoisegit的putty后，直接用tortoisegit可以不输入密码直接完成操作；但是当我使用git bash的时候，使用git pull之类的命令还需要我输入密码...
**这里又有两种方式...**
  * 调整tortoisegit的settings中的network选项，将tortoisegitplink.exe改成git安装目录的下bin\ssh.exe(好像我本地没有这个文件，只有个sh.exe文件)。如果先前用ssh-keygen.exe配置好了git下的ssh话，改完就能直接用，没配置好的话...等下说。
  * 默认安装tortoisegit，会使用PuTTY（plink）作为默认的ssh方式，声称对windows集成更好，如果不想改这种方式的话，就只能让git的ssh.exe使用PuTTY的密钥了，tortoisegit继续使用PuTTY。
注意：tortoisegit可以自动载入putty key，使用puttygen程序可以生成对应的公钥与私钥。

解决方法
我先前已经配置好了PuTTY，只是bash中的openssh不能用，于是我采用第二种方式。公钥是相同的，需要转换一下私钥。
定位putty的ppk文件，用puttygen（在tortoisegit目录里面）打开（conversions>import key）
![](/data/dokuwiki/git/pasted/20161013-165000.png)
然后点击` conversions>export openSSH key,保存文件为id_rsa文件，不要拓展名 `。
![](/data/dokuwiki/git/pasted/20161013-165019.png)
然后再点击下面的save public key按钮，保存为id_rsa.pub文件，效果如下：
找到自己%home%下（~）的.ssh文件夹,把刚才的两个文件扔进去。

拷贝id_rsa.pub内容到github：
![](/data/dokuwiki/git/pasted/20161013-165147.png)![](/data/dokuwiki/git/pasted/20161013-165159.png)

运行TortoiseGit开始菜单中的Pageant程序，程序启动后将自动停靠在任务栏中，双击该图标，弹出key管理列表。
![](/data/dokuwiki/git/pasted/20161013-165224.png)![](/data/dokuwiki/git/pasted/20161013-165231.png)
在弹出的key管理列表中点击add key,将第4步中保存的私钥（.ppk）文件加进来，关闭对话框即可。
经上述配置后，就可以使用TortoiseGit进行push、pull操作了，不用每次都输入密码了。

<wrap em>注意：git地址的区分：git@与https:/ /
https:/ /不管你有没有配置ssh key都需要输入密码，而git@则配置了sshkey之后不需要输密码。这点特别注意
选地址：</wrap>
![](/data/dokuwiki/git/pasted/20161013-165413.png)
![](/data/dokuwiki/git/pasted/20161013-165435.png)

参考：
http://www.tuicool.com/articles/Av2yMj
http://jingyan.baidu.com/article/63f236280f7e750209ab3d60.html