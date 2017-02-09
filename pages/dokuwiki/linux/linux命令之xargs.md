title: linux命令之xargs 

#  Linux命令之xargs 
xargs通常与find命令结合使用。
之所以能用到这个命令，关键是**由于很多命令不支持|管道来传递参数**，而日常工作中有有这个必要，所以就有了xargs命令，例如：
```

find /sbin -perm +700 |ls -l       这个命令是错误的
find /sbin -perm +700 |xargs ls -l   这样才是正确的

```
xargs 可以读入 stdin 的资料，并且以空白字元或断行字元作为分辨，将 stdin 的资料分隔成为 arguments 。 因为是以空白字元作为分隔，所以，如果有一些档名或者是其他意义的名词内含有空白字元的时候， xargs 可能就会误判了

选项解释
-0 当sdtin含有特殊字元时候，将其当成一般字符，想/'空格等
```

例如：root@localhost:~/test#echo "//"|xargs  echo 
      root@localhost:~/test#echo "//"|xargs -0 echo

``` 
-a file 从文件中读入作为sdtin，（看例一）
-e flag ，注意有的时候可能会是-E，flag必须是一个以空格分隔的标志，当xargs分析到含有flag这个标志的时候就停止。（例二）
-p 当每次执行一个argument的时候询问一次用户。（例三）
-n num 后面加次数，表示命令在执行的时候一次用的argument的个数，默认是用所有的。（例四）
-t 表示先打印命令，然后再执行。（例五）
-i 或者是-I，这得看linux支持了，将xargs的每项名称，一般是一行一行赋值给{}，可以用{}代替。（例六）
-r no-run-if-empty 当xargs的输入为空的时候则停止xargs，不用再去执行了。（例七）
-s num 命令行的最好字符数，指的是xargs后面那个命令的最大命令行字符数。（例八）
 
-L  num Use at most max-lines nonblank input lines per command line.-s是含有空格的。
-l  同-L
-d delim 分隔符，默认的xargs分隔符是回车，argument的分隔符是空格，这里修改的是xargs的分隔符（例九）
-x exit的意思，主要是配合-s使用。
-P 修改最大的进程数，默认是1，为0时候为as many as it can ，这个例子我没有想到，应该平时都用不到的吧。

----
假如你有一个文件包含了很多你希望下载的URL, 你能够使用xargs 下载所有链接
# cat url-list.txt | xargs wget –c
查找所有的jpg 文件，并且压缩它
# find / -name *.jpg -type f -print | xargs tar -cvzf images.tar.gz
拷贝所有的图片文件到一个外部的硬盘驱动 
# ls *.jpg | xargs -n1 -i cp {} /external-hard-drive/directory
快速重命名目录中的文件。
$ ls | xargs -t -i mv {} {}.bak
-i 选项告诉 xargs 用每项的名称替换 {}。-t 选项指示 xargs 先打印命令，然后再执行。
删除数量比较多的文件
ls | xargs -n 20 rm -fr
ls当然是输出所有的文件名(用空格分割)
xargs就是将ls的输出，每20个为一组(以空格为分隔符)，作为rm -rf的参数

参考：http://blog.csdn.net/andy572633/article/details/7214534