title: android_gravity 

#  android:gravit与android:layout_gravity的区别 

LinearLayout有两个非常相似的属性：
android:gravity与android:layout_gravity。
 
他们的区别在于：
 
android:gravity　属性是对该view中内容的限定．比如一个button 上面的text. 你可以设置该text 相对于view的靠左，靠右等位置．


android:layout_gravity是用来设置该view相对与父view 的位置．比如一个button 在linearlayout里，你想把该button放在linearlayout里靠左、靠右等位置就可以通过该属性设置． 
 
即android:gravity用于设置View中内容相对于View组件的对齐方式，而android:layout_gravity用于设置View组件相对于Container的对齐方式。


 我们可以通过设置android:gravity="center"来让EditText中的文字在EditText组件中居中显示；同时我们设置EditText的android:layout_gravity="right"来让EditText组件在LinearLayout中居右显示。看下效果：


 
正如我们所看到的，在EditText中，其中的文字已经居中显示了，而EditText组件自己也对齐到了LinearLayout的右侧。
 
附上布局文件：
 
[xhtml] view plaincopyprint?
<LinearLayout  
   xmlns:android="http://schemas.android.com/apk/res/android"  
    android:orientation="vertical"  
    android:layout_width="fill_parent"  
    android:layout_height="fill_parent">  
    <EditText  
        android:layout_width="wrap_content"  
        android:gravity="center"  
        android:layout_height="wrap_content"  
        android:text="one"  
        android:layout_gravity="right"/>  
</LinearLayout>  
 
 
 
 
那么上面是通过布局文件的方式来设置的。，相信大家都曾写过，那么如何通过Java代码来设置组件的位置呢？
 
依然考虑实现上述效果。
 
通过查看SDK，发现有一个setGravity方法， 顾名思义， 这个应该就是用来设置Button组件中文字的对齐方式的方法了。
仔细找了一圈，没有发现setLayoutgravity方法， 有点失望。 不过想想也对， 如果这边有了这个方法， 将Button放在不支持Layout_Gravity属性的Container中如何是好！ 
 
于是想到， 这个属性有可能在Layout中 ， 于是仔细看了看LinearLayout 的 LayoutParams， 果然有所发现， 里面有一个 gravity 属性，相信这个就是用来设置组件相对于容器本身的位置了，没错，应该就是他了。
 
实践后发现，如果如此， 附上代码，各位自己看下。
 
 
 
代码比较简单，但是发现它们还是花了我一点时间的。
 
[java] view plaincopyprint?
Button button  = new Button(this);  
button.setText("One");  
LinearLayout.LayoutParams lp = new LinearLayout.LayoutParams(LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT);  
//此处相当于布局文件中的Android:layout_gravity属性  
lp.gravity = Gravity.RIGHT;  
button.setLayoutParams(lp);  
//此处相当于布局文件中的Android：gravity属性  
button.setGravity(Gravity.CENTER);  
  
LinearLayout linear = new LinearLayout(this);  
//注意，对于LinearLayout布局来说，设置横向还是纵向是必须的！否则就看不到效果了。  
linear.setOrientation(LinearLayout.VERTICAL);  
linear.addView(button);  
setContentView(linear);  
 
 
或者这样也可以：
 
[java] view plaincopyprint?
Button button  = new Button(this);  
button.setText("One");  
//此处相当于布局文件中的Android：gravity属性  
button.setGravity(Gravity.CENTER);  
  
LinearLayout linear = new LinearLayout(this);  
//注意，对于LinearLayout布局来说，设置横向还是纵向是必须的！否则就看不到效果了。  
linear.setOrientation(LinearLayout.VERTICAL);  
  
LinearLayout.LayoutParams lp = new LinearLayout.LayoutParams(LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT);  
//此处相当于布局文件中的Android:layout_gravity属性  
lp.gravity = Gravity.RIGHT;  
  
linear.addView(button, lp);  
setContentView(linear);  

原文：http://blog.csdn.net/feng88724/article/details/6333809