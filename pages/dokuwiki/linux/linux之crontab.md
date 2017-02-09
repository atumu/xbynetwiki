title: linux之crontab 

#  linux之crontab 
##  at仅运行一次的工作排程 
atd 的启动与 at 运行的方式
root@www ~]# /etc/init.d/atd restart
正在停止 atd:                          [  确定  ]
正在启动 atd:                          [  确定  ]

# 再配置一下启动时就启动这个服务，免得每次重新启动都得再来一次！
[root@www ~]# chkconfig atd on

[root@www ~]# at [-mldv] TIME
[root@www ~]# at -c 工作号码
选项与参数：
-m  ：当 at 的工作完成后，即使没有输出信息，亦以 email 通知使用者该工作已完成。
-l  ：at -l 相当於 atq，列出目前系统上面的所有该使用者的 at 排程；
-d  ：at -d 相当於 atrm ，可以取消一个在 at 排程中的工作；
-v  ：可以使用较明显的时间格式列出 at 排程中的工作列表；
-c  ：可以列出后面接的该项工作的实际命令内容。

TIME：时间格式，这里可以定义出『什么时候要进行 at 这项工作』的时间，格式有：
  HH:MM				ex> 04:00
	在今日的 HH:MM 时刻进行，若该时刻已超过，则明天的 HH:MM 进行此工作。
  HH:MM YYYY-MM-DD		ex> 04:00 2009-03-17
	强制规定在某年某月的某一天的特殊时刻进行该工作！
  HH:MM[am|pm] [Month] [Date]	ex> 04pm March 17
	也是一样，强制在某年某月某日的某时刻进行！
  HH:MM[am|pm] + number [minutes|hours|days|weeks]
	ex> now + 5 minutes	ex> 04pm + 3 days
	就是说，在某个时间点『再加几个时间后』才进行。


范例一：再过五分钟后，将 /root/.bashrc 寄给 root 自己
[root@www ~]# at now + 5 minutes  <==记得单位要加 s 喔！
at> /bin/mail root -s "testing at job" < /root/.bashrc
at> <EOT>   <==这里输入 [ctrl] + d 就会出现 <EOF> 的字样！代表结束！
job 4 at 2009-03-14 15:38
# 上面这行资讯在说明，第 4 个 at 工作将在 2009/03/14 的 15:38 进行！
# 而运行 at 会进入所谓的 at shell 环境，让你下达多重命令等待运行！

范例二：将上述的第 4 项工作内容列出来查阅
[root@www ~]# at -c 4
#!/bin/sh               <==就是透过 bash shell 的啦！
# atrun uid=0 gid=0
# mail     root 0
umask 22
....(中间省略许多的环境变量项目)....
cd /root || {           <==可以看出，会到下达 at 时的工作目录去运行命令
         echo 'Execution directory inaccessible' >&2
         exit 1
}

/bin/mail root -s "testing at job" < /root/.bashrc
# 你可以看到命令运行的目录 (/root)，还有多个环境变量与实际的命令内容啦！

范例三：由於机房预计於 2009/03/18 停电，我想要在 2009/03/17 23:00 关机？
[root@www ~]# at 23:00 2009-03-17
at> /bin/sync
at> /bin/sync
at> /sbin/shutdown -h now
at> <EOT>
job 5 at 2009-03-17 23:00
# 您瞧瞧！ at 还可以在一个工作内输入多个命令呢！不错吧！

###  at 工作的管理 

那么万一我下达了 at 之后，才发现命令输入错误，该如何是好？就将他移除啊！利用 atq 与 atrm 吧！

[root@www ~]# atq
[root@www ~]# atrm (jobnumber)

范例一：查询目前主机上面有多少的 at 工作排程？
[root@www ~]# atq
5       2009-03-17 23:00 a root
# 上面说的是：『在 2009/03/17 的 23:00 有一项工作，该项工作命令下达者为 
# root』而且，该项工作的工作号码 (jobnumber) 为 5 号喔！

范例二：将上述的第 5 个工作移除！
[root@www ~]# atrm 5
[root@www ~]# atq
# 没有任何资讯，表示该工作被移除了！
##  循环运行的例行性工作排程 
相对於 at 是仅运行一次的工作，循环运行的例行性工作排程则是由 cron (crond) 这个系统服务来控制的。 Linux 也提供使用者控制例行性工作排程的命令 (crontab)。 
我们可以限制使用 crontab 的使用者帐号喔！使用的限制数据有：

/etc/cron.allow：
将可以使用 crontab 的帐号写入其中，若不在这个文件内的使用者则不可使用 crontab；

/etc/cron.deny：
将不可以使用 crontab 的帐号写入其中，若未记录到这个文件当中的使用者，就可以使用 crontab 。
当使用者使用 crontab 这个命令来创建工作排程之后，该项工作就会被纪录到 /var/spool/cron/ 里面去了，而且是以帐号来作为判别的喔！举例来说， dmtsai 使用 crontab 后， 他的工作会被纪录到 /var/spool/cron/dmtsai 里头去！
默认情况下，任何使用者只要不被列入 /etc/cron.deny 当中，那么他就可以直接下达『 crontab -e 』去编辑自己的例行性命令了！



[root@www ~]# crontab [-u username] [-l|-e|-r]
选项与参数：
-u  ：只有 root 才能进行这个任务，亦即帮其他使用者创建/移除 crontab 工作排程；
-e  ：编辑 crontab 的工作内容
-l  ：查阅 crontab 的工作内容
-r  ：移除所有的 crontab 的工作内容，若仅要移除一项，请用 -e 去编辑。

范例一：用 dmtsai 的身份在每天的 12:00 发信给自己
[dmtsai@www ~]$ crontab -e
# 此时会进入 vi 的编辑画面让您编辑工作！注意到，每项工作都是一行。
0   12  *  *  * mail dmtsai -s "at 12:00" < /home/dmtsai/.bashrc
#分 时 日 月 周 |<# ==命令串# # >|
![](/data/dokuwiki/linux/pasted/20150628-220612.png)
![](/data/dokuwiki/linux/pasted/20150628-220632.png)


例题：
假若你的女朋友生日是 5 月 2 日，你想要在 5 月 1 日的 23:59 发一封信给他，这封信的内容已经写在 /home/dmtsai/lover.txt 内了，该如何进行？
答：
直接下达 crontab -e 之后，编辑成为：
59 23 1 5 * mail kiki < /home/dmtsai/lover.txt
那样的话，每年 kiki 都会收到你的这封信喔！（当然罗，信的内容就要每年变一变啦！）

例题：
假如每五分钟需要运行 /home/dmtsai/test.sh 一次，又该如何？
答：
同样使用 crontab -e 进入编辑：
*/5 * * * * /home/dmtsai/test.sh
那个 crontab 每个人都只有一个文件存在，就是在 /var/spool/cron 里面啊！ 还有建议您：『命令下达时，最好使用绝对路径，这样比较不会找不到运行档喔！』

例题：
假如你每星期六都与朋友有约，那么想要每个星期五下午 4:30 告诉你朋友星期六的约会不要忘记，则：
答：
还是使用 crontab -e 啊！
30 16 * * 5 mail friend@his.server.name < /home/dmtsai/friend.txt


[dmtsai@www ~]$ crontab -l
59 23 1 5 * mail kiki < /home/dmtsai/lover.txt
*/5 * * * * /home/dmtsai/test.sh
30 16 * * 5 mail friend@his.server.name < /home/dmtsai/friend.txt

# 注意，若仅想要移除一项工作而已的话，必须要用 crontab -e 去编辑～
# 如果想要全部的工作都移除，才使用 crontab -r 喔！
[dmtsai@www ~]$ crontab -r
[dmtsai@www ~]$ crontab -l
no crontab for dmtsai

##  系统的配置档： /etc/crontab 

这个『 crontab -e 』是针对使用者的 cron 来设计的，如果是『系统的例行性任务』时， 该怎么办呢？是否还是需要以 crontab -e 来管理你的例行性工作排程呢？当然不需要，你只要编辑 /etc/crontab 这个文件就可以啦！有一点需要特别注意喔！那就是 crontab -e 这个 crontab 其实是 /usr/bin/crontab 这个运行档，但是 /etc/crontab 可是一个『纯文字档』喔！你可以 root 的身份编辑一下这个文件哩！
##  可唤醒停机期间的工作任务 
anacron 并不是用来取代 crontab 的，anacron 存在的目的就在於我们上头提到的，在处理非 24 小时一直启动的 Linux 系统的 crontab 的运行！

[root@www ~]# anacron [-sfn] [job]..
[root@www ~]# anacron -u [job]..
选项与参数：
-s  ：开始一连续的运行各项工作 (job)，会依据时间记录档的数据判断是否进行；
-f  ：强制进行，而不去判断时间记录档的时间戳记；
-n  ：立刻进行未进行的任务，而不延迟 (delay) 等待时间；
-u  ：仅升级时间记录档的时间戳记，不进行任何工作。
job ：由 /etc/anacrontab 定义的各项工作名称。

更多参考:http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html