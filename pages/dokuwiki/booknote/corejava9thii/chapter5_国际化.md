title: chapter5_国际化 

#  Chapter5 国际化 
Created Friday 27 March 2015

#  Locale 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075412.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075423.png)
#  数字格式 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075430.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075434.png)

#  日期和时间 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075438.png)

##  SimpleDateFormat 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075450.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075458.png)

```

SimpleDateFormat sdfmt=new SimpleDateFormat("yyyy:MM:dd HH:mm:ss",locale);
System.out.println(sdfmt.format(new Date()));

```

#  排序 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075528.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075532.png)

#  消息格式化 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075551.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075605.png)
#  文本文件和字符集 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075620.png)

#  资源包 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075627.png)
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075630.png)

##  定位资源包 
![](/data/dokuwiki/booknote/corejava9thii/pasted/20150521-075646.png)
##  使用ResourceBundle查找资源时注意 
在IDE中将properties文件放在src/目录下。
在jar中直接放到jar根目录下即可。否则找不到。
