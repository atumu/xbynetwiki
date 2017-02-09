title: androidsdk版本与兼容性 

#  Chapter6 AndroidSDK版本与兼容性 
Created Saturday 07 March 2015

![](/data/dokuwiki/booknote/androidprogramming/pasted/20150521-084528.png)
举例：ActionBar是在Android3.0中才添加进来的，为了使用它，我们需要提前判断设备的编译版本。
**if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.HONEYCOMB){**
	**ActionBar actionBar=getActionBar();**
**	actionBar.setSubtitle("xby");**
**}**
禁止AndroidLint提示兼容性问题：在前段代码之前添加注解。
@TargetApi(11)
