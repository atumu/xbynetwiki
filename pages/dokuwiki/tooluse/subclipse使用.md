title: subclipse使用 

#  eclipse SVN插件subclipse 
本文参考：http://blog.snsgou.com/blog/552.html
Eclipse 下连接 SVN 库有两种插件 Subclipse 与 Subversive，Subclipse 主页位于http://subclipse.tigris.org ，与SVN（http://subversion.apache.org ）联系紧密，我们可以称之为 SVN 官方的 eclipse 插件。而Subversive 则是 eclipse 官方的 SVN 插件，主页位于http://www.eclipse.org/subversive。Subclipse 是 SVN 直接支持的项目，在很早的时候就已经出现了。
Subclipse插件
使用详见：http://www.ibm.com/developerworks/cn/opensource/os-ecl-subversion/ 
Eclipse下SVN插件的使用，下去大家自己去研究一下，可以参考：
http://www.blogjava.net/gdhqs/archive/2009/07/03/285399.html
http://www.cnblogs.com/chencidi/archive/2010/12/13/1904781.html
##  SVN/Subclipse使用步骤 
第一个步骤：演示如何把项目放入svn进行管理
1).选中项目名称，右键，选择`  Team --> Share Project --> SVN ` ，输入svn地址，选择 finish，后进行同步视图，选中项目，右键commit。
第二个步骤：删除项目
SVN资源库 --> 右键 --> 选中要删除的对象
（注意：在客户端中删除方式为，选择要删除的项目，然后右键，选择 TortoiseSvn -->repo-brower，进入浏览模式，选择删除即可。）
第三个步骤：从服务器端check out(签出)项目
import --> svn --> 输入或选择svn地址，输入用户名和密码，在列表中选中要Check out的项目 --> finish
第四个步骤：提交源代码文件
先同步，在提交：选中src，或者webroot目录，或者两个目录一起，` 右键 --> Team --> 同步SVN `，系统会提示进入**同步视图**，
在同步视图里面选择**commit**(` Outgoing mode模式 `)，或**update**(` Incoming mode模式) `
` **在修改任何文件之前，都必须先同步。如果不同步会覆盖别个的东西或者不能提交。** `
![](/data/dokuwiki/tooluse/pasted/20150820-072713.png)![](/data/dokuwiki/tooluse/pasted/20150820-072727.png)
第五个步骤：更新
**先同步，再update**。选中src，或者webroot目录，或者两个目录一起，` 右键 --> Team --> 同步SVN `，系统会提示进入同步视图，在同步视图里面选择commit，或update
如果有些文件会比较多人用，那么在修改前，请先锁定，锁定后其他人将不能提交，
锁定的步骤是：选择要锁定的文件 --` > 右键 --> Team --> lock 。 `

**使用SVN的流程：**
  * 1、每天工作的第一件事情：更新，update；
  * 2、下班前的最后一件事情：提交，commit；

