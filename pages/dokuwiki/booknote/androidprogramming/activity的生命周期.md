title: activity的生命周期 

#  Chapter3 Activity的生命周期 
Created Wednesday 04 March 2015

##  Activity生命周期状态图 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084253.png)

##  设备旋转与Activity生命周期 
设备配置与备选资源
旋转设备会改变设备运行时配置（runtime  configuration change ）。其他的设备配置包括：屏幕方向，键盘类型等。

###  设备旋转前保存数据 
protected void onSaveInstanceState(Bundle bundle)
之后在onCreate中检查该bundle，注意开始时需要判断非空：
if(savedInstanceState !=null){
...
}
bundle中保存的其他对象需要实现Serializable接口
bundle保存在系统的Activity record中。

##  Activity Record理解与暂存(stashed)状态 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084304.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084310.png)

##  Android日志记录的级别与方法 
android.util.Log
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084314.png)
