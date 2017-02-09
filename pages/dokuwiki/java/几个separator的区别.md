title: 几个separator的区别 

#  line.separator、path.separator、file.separator的区别 

在windows下通过System.getProperties()可以看到
line.separator=\r\n （行分隔符，linux下为\n，windows下为\r\n）
file.separator=\  (指定是文件路径的分隔符)
path.separator=; (指的是PATH变量的分隔符)
