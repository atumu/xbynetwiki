title: 创建第二个activity 

#  Chapter5 创建第二个Activity 
Created Saturday 07 March 2015

注意:为每个Activity都在Manifest中声明。

##  启动Activity 
Activity.startActivity();该方法调用请求发送给了操作系统管理的ActivityManager。ActivityManager负责创建Activity实例并调用其onCreate(..)方法。Activity通过传入的Intent参数决定启动哪个Activity。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084414.png)

###  基于Intent的通信 
intent对象是组件（如Activity,Service,BroadcastReceiver,ContentProvider）用于与操作系统通信的一种媒介工具。
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084418.png)

####  显式与隐式Intent 
本应用中各组件之间一般用显式的，调用其他程序的组件一般用隐式的。

##  Activity之间的数据传递 

###  使用intent extra 
intent.putExtra();
一般地，extra的标志采用包名作为前缀："net.xby1993.myapp.my_extra";这样可以避免来自不同应用程序intent extra提取的冲突。
之后便可以在Activity的onCreate()方法中getIntent().getXxxExtra()获取相关值
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084424.png)

###  从之Activity获取返回结果 
```

startActivityForResult(Intent intent,int requestCode);//当一个Activity启动了多个Activity并且要求返回值时，请求码是用于识别返回值源的。
之后我们便可以在子Activity中设置返回结果，这些返回结果会在子Activity调用**Activity.finish()**或者用户按下返回键后被返回。
setResult(int resultCode,Intent data);
通常来说返回结果代码为：Activity.RESULT_OK;Activity.RESULT_CANCELED
但返回结果真正返回时ActivityManager会调用父Activity的onActivityResult(int requestCode,int resultCode,Intent data);

```
**所以父Activity还需要实现该方法以便对返回结果进行处理。一般需要判断请求码和结果码。并且需要判断data!=null**
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084450.png)
注意：ActivityManager维护着一个非特定用户独享的回退栈，所有应用的activity都共享该回退栈。所以需要将ActivityManager设计成系统管理的对象。




