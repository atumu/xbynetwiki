title: linux时间命名文件名备份 

#  linux时间命名文件名备份 
```

#! /bin/sh
subfix=`date +%Y%m%d`
tar -zcvf data$subfix.tar.gz dokuwiki/data --exclude=dokuwiki/data/cache

```