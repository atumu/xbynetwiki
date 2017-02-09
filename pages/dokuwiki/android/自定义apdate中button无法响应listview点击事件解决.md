title: 自定义apdate中button无法响应listview点击事件解决 

#  android:自定义Apdate中Button无法响应listView点击事件解决 
在一个list的item中添加button需要注意防止Button捕获点击事件。
所以需要在xml中给Button设置:
```

android:clickable="false"

```
如果不这样设置，Button就会抢夺该item的焦点，这样你的item就点不动了。

另外可以参考：http://www.360doc.com/content/12/1023/16/6541311_243302469.shtml