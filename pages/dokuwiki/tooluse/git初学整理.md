title: git初学整理 

#   1. Git概念 
 
1.1. Git库中由三部分组成 
      <blockquote> Git 仓库就是那个.git 目录，其中存放的是我们所提交的文档索引内容，Git 可基于文档索引内容对其所管理的文档进行内容追踪，从而实现文档的版本控制。.git目录位于工作目录内。</blockquote> 
1） 工作目录：用户本地的目录； 
2） Index（索引）：将工作目录下所有文件（包含子目录）生成快照，存放到一个临时的存储区域，Git 称该区域为索引。 
3） 仓库：将索引通过commit命令提交至仓库中，每一次提交都意味着版本在进行一次更新。 
clip_image002
我们首先使用ssh命令连接github.com的SSH服务，登录用户名为git（所有GitHub用户共享此SSH用户名，不要写成其他）。
ssh -T git@github.com
执行之后提示：Permission denied (publickey).
这说明我们还没在GitHub账户中正确设置公钥认证。下图为GitHub的SSH公钥设置界面:
GitHub的SSH服务支持OpenSSH格式的公钥认证，可以通过Ubuntu下的ssh-keygen命令创建公钥/私钥对。
ssh-keygen -C "yourname@yourcompany.com" -f ~/.ssh/github
也可以用ssh-keygen命令以不同的名称创建多个公钥，当拥有多个GitHub账号时，非常重要。这是因为虽然一个GitHub账号允许使用多个不同的SSH公钥，但反过来，一个SSH公钥只能对应于一个GitHub账号。
接下来就将~/.ssh/github.pub文件内容拷贝到剪切板。公钥是一行长长的字符串，若用编辑器打开公钥文件会折行显示，注意在copy时一定不要在其中插入多余的换行符、空格等，否则在公钥认证过程因为服务器端和客户端公钥不匹配而导致认证失败。然后将公钥文件中的内容粘贴到GitHub的SSH公钥管理的对话框，即上图key对话框中，并为这个SSH Key起个名字并保存。设置成功后，再用ssh命令访问GitHub，会显示一条认证成功信息并退出。在认证成功的信息中还会显示该公钥对应的用户名。
ssh -T git@github.com
执行后提示：Hi github! You've successfully authenticated, but GitHub does not provide shell access.
通过以上的设置之后，我们就能够通过SSH的方式，直接使用Git命令访问GitHub托管服务器了。那么，下面我们就开始使用Git进行版本控制：
1. 从服务器下载代码，准确的说应该是从GitHub服务器复制一个版本库到本地：
mkdir git
mkdir repos
cd git/repos
git clone git@github.com:"account context"/"repos name".git
2. 获取到源码之后，就可以进行开发了，代码开发完成就可以提交代码：
git add .    //往暂存区域添加已添加和修改的文件，不处理删除的文件
git status   //比较本地数据目录与暂存区域的变化
git commit -m "commit directions" //提到代码到本地数据目录，并添加提交说明
3. 有可能你和其他人改的是同一个文件，那么冲突的情况是在所难免的，那么在提交之后再获取一下代码，就会提示代码冲突的文件，我们需要做的就是处理这些冲突，并再次提交：
git pull     //更新代码
根据提示修改冲突文件中的代码
git add .
git commit -m "commit directions"
4. 当你做完以上的步骤的时候，你需要做的是把本地数据目录的版本库的数据同步到GitHub服务器上去，这样你的同事才能够看到你作出的修改：
git push
注意：""中的内容需要读者根据自己实际情况书写合适的内容。


#  使用eclipse-EGIT管理： 

##  ssh key生成 

Repository创建好以后需要提交自己的ssh key.一般来说,key的生成有两种方式:

使用官方指南提供的msysgit工具的ssh-keygen命令生成.
使用eclipse自带的ssh2工具生成.
我们这里选用第二方式,使用eclipse自带的ssh2工具,具体步骤:

如果你的ssh2已经有了需要使用的id key,请先备份,然后将目录清空.
点击Window->Preferences->General->Network->SSH2,点击Key Management tab页,点击Generate RSA Key,然后点击Save Private key,将key保存自定义目录.
点击Export Via SFTP,在弹出窗口填入git@github.com,此时你的ssh目录会多出一个known_hosts文件,此文件与id_rsa.pub一样重要.
将生成的id_rsa.pub打开,删除空行复制里面的内容,然后粘贴到github的ssh keys中.
重启eclipse,查看ssh选项卡中是否能load出RSA Key与known hosts,如果不能检查以上步骤,否则你是连不上github的.
##  PUSH配置 

创建一个应用,然后在应用上右键->Team->Share Project,选择git,点击next,点击use or create repository in parent folder,不用理会上面的警告,直接finish.
在应用根目录下创建一个README,随便写入内容,然后右键->team->commit,但此时文件仍然在你本地,并没有push到远程服务器上.
接着右键->team->remote->push,此处填写你的项目地址,协议,填写完后点击next,如果出现ssh://git@github.com:22 The authenticity of host “github.com” can’t be established. RSA key的错误信息请重启eclipse,重启完毕后继续此步骤.
如果没有异常,在弹出窗口直接点击add all branches spec按钮,最后点击finish,整个过程完毕,点击github你的主页就能看到你的代码.

