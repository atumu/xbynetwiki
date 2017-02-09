title: linux命令之locate_which_whereis 

#  Linux命令之locate,which,whereis 
我们经常在linux要查找某个文件，但不知道放在哪里了，可以使用下面的一些命令来搜索： 
  * which  查看可执行文件的位置。
  * whereis 查看文件的位置。 
  * locate   配合数据库查看文件位置。
  * find   实际搜寻硬盘查询文件名称。
which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。 

locate命令可以在搜寻数据库时快速找到档案，数据库由updatedb程序来更新，updatedb是由cron daemon周期性建立的，locate命令在搜寻数据库时比由整个由硬盘资料来搜寻资料来得快，但较差劲的是locate所找到的档案若是最近才建立或 刚更名的，可能会找不到，在内定值中，updatedb每天会跑一次，可以由修改crontab来更新设定值。(etc/crontab)

whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。
