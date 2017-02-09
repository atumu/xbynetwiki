title: tortoisegit使用 

#  tortoiseGit报错 git did not exit cleanly (exit code 1) 
tortoiseGit报错 git did not exit cleanly (exit code 1)` 注意是exitcode 1而不是128 `
解决思路一：未成功
参考：http://stackoverflow.com/questions/22165953/tortoisegit-git-did-not-exit-cleanly-exit-code-1
[plain] view plaincopy
Right click -> TortoiseGit -> Settings -> Network  
SSH client was pointing to C:\Program Files\TortoiseGit\bin\TortoisePlink.exe  
Changed path to C:\Program Files (x86)\Git\bin\ssh.exe  
解决思路二：` 猜测是上传的内容太大，导致push失败！成功解决 `
参考：http://www.52au3.com/tools/tortoisegit-not-exit-cleanly.html
遂找到解决方法如下： ` 添加git的配置项：postBuffer = 524288000 `
![](/data/dokuwiki/tooluse/pasted/20150511-012837.png)

