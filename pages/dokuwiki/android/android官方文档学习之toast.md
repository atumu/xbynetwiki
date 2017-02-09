title: android官方文档学习之toast 

#  android官方文档学习之Toast 
```

Toast.makeText(context, text, duration).show();//记得调用show()哦，很多时候容易遗忘

```
##  Toast显示位置定位 
标准的Toast通知水平居中显示在屏幕底部附近，可以通过` setGravity(int, int, int) `方法来重新设置显示位置。
这个方法有三个参数： **1.Gravity常量（详细参照Gravity类）； 2.X轴偏移量； 3.Y轴偏移量。**
例如：如果你想让Toast通知显示在屏幕的左上角，可以这样设置setGravity(int ,int ,int )方法：
` toast.setGravity(Gravity.TOP|Gravity.LEFT, 0, 0); `
如果想让位置向右移，可以增加第二个参数的值，要向下移动，可以增加最后一个参数的值。
##  创建自定义Toast视图 
你可以给Toast通知创建一个自定义的布局(layout)。要创建一个自定义的布局(layout)，可以在XML文件或程序代码中定义一个View布局，然后把(根)View对象传递给` setView(View) `
```

<LinearLayout xmlns:android=
             "http://schemas.android.com/apk/res/android"
             android:id="@+id/toast_layout_root"
             android:orientation="@+"horizontal"
             android:layout_width="fill_parent"
             android:layout_height="fill_parent"
             android:padding="10dp"
             android:background="#DAAA"
             >
   <ImageView android:id="@+id/image"
              android:layout_width="wrap_content"
              android:layout_height="fill_parent"
              android:layout_marginRight="10dp"
              />
   <TextView android:id="@+id/text"
             android:layout_width="wrap_content"
             android:layout_height="fill_parent"
             android:textColor="#FFF"
             />
</LinearLayout>

```
```

LayoutInflater inflater = getLayoutInflater();
View layout = inflater.inflate(R.layout.toast_layout,
                               (ViewGroup) findViewById(R.id.toast_layout_root));
 
ImageView image = (ImageView) layout.findViewById(R.id.image);
image.setImageResource(R.drawable.android);
TextView text = (TextView) layout.findViewById(R.id.text);
text.setText("Hello! This is a custom toast!");
 
Toast toast = new Toast(getApplicationContext());
toast.setGravity(Gravity.CENTER_VERTICAL, 0, 0);
toast.setDuration(Toast.LENGTH_LONG);
toast.setView(layout);
toast.show();

``` 
` 除非你要用setView(View)方法定义布局(layout)，否则不要使用公共的Toast类构造器。如果不使用自定义的布局(layout)，必须使用makeText(Context, int, int)方法来创建Toast `