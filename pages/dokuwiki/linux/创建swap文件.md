title: 创建swap文件 

#  Linux创建swap文件 

创建swap文件方法
 1) 创建一个足够大的文件
 dd if=/dev/zero of=/localdisk/swapfile bs=1024 count=4096000
 (count的值等于1024 x 你想要的文件大小, 4096000是4G)

 2) 把这个文件变成swap文件.
 mkswap /localdisk/swapfile

 3) 启用这个swap文件
 swapon /localdisk/swapfile

 4) 在每次开机的时候自动加载swap文件, 需要在 /etc/fstab 文件中增加一行
 /localdisk/swapfile swap swap defaults 0 0