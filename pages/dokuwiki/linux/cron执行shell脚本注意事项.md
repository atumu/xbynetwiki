title: cron执行shell脚本注意事项 

#  cron执行shell脚本注意事项 
#crontab -e打开编辑页面
45 15 * * * /opt/htdocs/backdata.sh 即每天的15点45分执行这个脚本
分 时 日 月 星期 年（可选)

编辑之后执行
#service cron restart

**编辑shell脚本注意2点：**
第一、cron不会提供**环境变量**，所以我们需要
export PATH=/opt/jre7/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
第二、使用**绝对路径**
basedir="/opt/htdocs/"
tar -zcvf ${basedir}"data$subfix.tar.gz" ${basedir}"path/data"
第三、shell语法**字符串合并**为 
basedir="/opt/htdocs/"
path=${basedir}"data$subfix.tar.gz"

一个完整脚本
```

#! /bin/sh
export PATH=/opt/jre7/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/opt/jre7/bin
subfix=`date +%Y%m%d` #格式20150919
basedir="/opt/htdocs/"
tar -zcvf ${basedir}"data$subfix.tar.gz" ${basedir}"path/data" --exclude=${basedir}"path/data/cache"

```