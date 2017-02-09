title: css文字截断 

#  css文字截断 
方式一、不推荐，hover无法显示完整
 确定一个宽度(可选), 然后对于加载上的文字截断(如果文字过长),这种方法是:
<div style="width:300px; overflow:hidden; text-overflow:ellipsis; white-space:nowrap;"> 任意长度的字符串 </div>
<div style=" overflow:hidden; text-overflow:ellipsis; white-space:nowrap;"> 任意长度的字符串 </div>
也可以使用input来变相,实现,可以不不确定宽度,其实已经算是动态截断了.
<input type=”text” style=”width:100%; cursor:default; border-width:0; border-style:none; background-color:transparent;” value=”任意长度的字符串” readonly/>
参考：http://blog.csdn.net/norsd/article/details/3577250

方式二、推荐，hover可显示。
http://stackoverflow.com/questions/16871519/automatically-add-if-text-extends-certain-width-but-show-it-on-hover
```

#test {
    width: 50px;
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
}

#test:hover {
    overflow: visible;
}

```