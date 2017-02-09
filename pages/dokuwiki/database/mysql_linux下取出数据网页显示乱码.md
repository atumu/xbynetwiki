title: mysql_linux下取出数据网页显示乱码 

#  mysql linux下取出数据网页显示乱码 
添加jdbc url参数useUnicode=true&characterEncoding=UTF-8 
添加的作用是：指定字符的编码、解码格式。
例如：mysql数据库用的是gbk编码，而项目数据库用的是utf-8编码。这时候如果添加了useUnicode=true&characterEncoding=UTF-8 ，那么作用有如下两个方面：
1. 存数据时：
数据库在存放项目数据的时候会先用UTF-8格式将数据解码成字节码，然后再将解码后的字节码重新使用GBK编码存放到数据库中。

2.取数据时：
在从数据库中取数据的时候，数据库会先将数据库中的数据按GBK格式解码成字节码，然后再将解码后的字节码重新按UTF-8格式编码数据，最后再将数据返回给客户端。

注意：在xml配置文件中配置数据库utl时，要使用&的转义字符也就是` &amp; `
例如：```

<property name="url" value="jdbc:mysql://localhost:3306/email?useUnicode=true&amp;characterEncoding=UTF-8" />

```
HTML中常用的特殊字符：
最常用的字符实体(Character Entities)
```

 	显示一个空格	&nbsp;	&#160;
<	小于	&lt;	&#60;
>	大于	&gt;	&#62;
&	&符号	&amp;	&#38;
"	双引号	&quot;	&#34;

```
其他常用的字符实体(Character Entities)
显示结果	说明	Entity Name	Entity Number
```

©	版权	&copy;	&#169;
®	注册商标	&reg;	&#174;
×	乘号	&times;	&#215;
÷	除号	&divide;	&#247;

```
