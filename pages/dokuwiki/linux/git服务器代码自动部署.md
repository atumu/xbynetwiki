title: git服务器代码自动部署 

#  git代码自动部署方案 

传统代码部署方式：
  * windows 远程桌面
  * FTP/SFTP
  * 登录服务器pull github代码
  * Phing（PHP专业部署工具）

git 自动部署流程图
![](/data/dokuwiki/linux/pasted/20150517-132427.png)

##  服务器端准备工作： 


 0. 这些工作都在root或有管理权限的帐号下进行，下面以root为用户，切换到其他用户的时候会提示
1. 确保安装了git
2. 为了安全起见，新建一个专门用于代码部署的无特权用户
 useradd -m deployuser
 passwd deployuser        #设置该用户的密码，也可根据喜好配置成免密码登陆
3. 新建一个目录作为要部署代码的根目录，如：
 mkdir /var/www/html/deploy.git
4. 将这个目录的属主和属组都改为上面新建的用户deployuser
 cd /var/www/html
 chown deployuser:deployuser deploy.git
 5. 切换到部署代码的专用用户
su deployuser 
6. 进入项目根目录，初始化为git仓库
cd deploy
git init
7. 【重要】让仓库接受代码提交
 git config receive.denyCurrentBranch ignore
git config core.worktree /opt/htdocs
 [可选]  git config --bool receive.denyNonFastForwards false  #禁止强制推送
至此，一个空的git仓库就在服务器上建好了，仓库的地址为：
ssh://deployuser@ipaddress:22/var/www/html/deploy/.git

回到服务器端：
    测试部署效果 更新服务端 git 仓库状态并检出文件
               cd /var/www/html/deploy
               git update-server-info

                git checkout -f
        OR：
                git checkout branch_name     # 如果push的不是master分支

    . 检查是不是将文件更新进来了
**  最后一步 设置服务器端更新钩子 post-update**
              cd .git/hooks
       新建 post-receive 或将 post-receive.sample 重命名为 post-receive
              touch post-receive
       OR:
              mv post-receive.sample post-receive
              vim post-receive
      然后设置可执行权限sudo chmod a+x post-recieve
       将如下内容复制到文件中
                 #!/bin/sh
                 unset GIT_DIR
                 cd ..
                 git checkout -f
注: 将post-receive 替换为 post-update也可以, 不过需要先将post-update中的exec git update-server-info这一行删掉
参考：
http://www.douban.com/note/407034249/