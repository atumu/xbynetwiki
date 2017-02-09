title: android_禁止屏幕转向 

#  android_禁止屏幕转向 

Android中屏幕旋转会导致Activity被销毁然后重新创建，这会导致很麻烦的状态存储与恢复问题。虽然有方法可以解决。但是当我们的应用无需旋转屏幕时，主动禁用屏幕旋转是一个不错的选择。

描述
方式一：
步骤：
在Android中要让一个程序的界面始终保持一个方向，不随手机方向转动而变化的办法： 只要在AndroidManifest.xml里面配置一下就可以了。 
在AndroidManifest.xml的activity(需要禁止转向的activity)配置中加入 android:screenOrientation=”landscape”属性即可(landscape是横向，portrait是纵向)。 
要避免在转屏时重启activity，可以通过在androidmanifest.xml文件中重新定义方向(给每个activity加上 android:configChanges=”keyboardHidden|orientation”属性)，并根据Activity的重写 onConfigurationChanged(Configuration newConfig)方法来控制，这样在转屏时就不会重启activity了，而是会去调用 onConfigurationChanged(Configuration newConfig)这个钩子方法。
例如
```

@Override public void onConfigurationChanged(Configuration newConfig) {   
super.onConfigurationChanged(newConfig);   
if(newConfig.orientation==Configuration.ORIENTATION_LANDSCAPE) {  
//横向 setContentView(R.layout.file_list_landscape);  
} else {  
//竖向 setContentView(R.layout.file_list);  
}  
}  

```
在模拟器中可以按 CTL+F11 模拟做屏幕旋转。

方式二
另外一种方式就是禁用方向传感器：从Android 1.5开始系统可以设置Sensor旋转屏幕，如果你的应用在部分方面没有处理好横屏和竖屏的切换，可能需要强制禁用方向感应器Sensor，相关的方法可以在androidmanifest.xml的相关activity中加入android:screenOrientation="nosensor"属性。