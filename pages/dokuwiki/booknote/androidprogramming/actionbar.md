title: actionbar 

#  ActionBar 
更多请参考：[Actionbar学习](/pages/dokuwiki/android/actionbar)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150605-035803.png)
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150605-035656.png)
##  OptionsMenu 
系统自带图标位置：D:\android-sdk-windows\platforms\android-21\data\res
创建选项菜单，覆盖Activity的` onCreateOptionsMenu(Menu) `,响应菜单事件` onOptionsItemSelected(MenuItem item) `
挡在fragment中创建OptionsMenu时应该注意:
覆盖` onCreateOptionsMenu(Menu) `和` onOptionsItemSelected(MenuItem item) `
在` onCreate中添加setHasOptionsMenu(true) `;
##  实现层级式导航 
actiobar.setDisplayHomeAsUpEnabled(true);
```

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            getActivity().getActionBar().setDisplayHomeAsUpEnabled(true);
        }   

```
但是这样只会让应用图标变为按钮，并没有做具体的事情，我们需要编码实现。
###  响应向上按钮 
**通知fragmentManager:**
`  setHasOptionsMenu(true); `
```

  @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setHasOptionsMenu(true);
    }

```
**响应应用图标(Home)菜单项:**
` android.R.id.home `
配置NavUtils与manifest文件中的元数据。
```

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case android.R.id.home:
          	//如果在元数据中指定了Parent Activity.则向上导航。
         	 if( NavUtils..getParentActivityName(getActivity())!=null)
                NavUtils.navigateUpFromSameTask(getActivity());
       		 }
                return true;
            default:
                return super.onOptionsItemSelected(item);
        } 
    }

```
```

 <activity
      android:name=".CrimeActivity"
      android:label="@string/app_name" >
      <meta-data android:name="android.support.PARENT_ACTIVITY"
        android:value=".CrimeListActivity"/>
    </activity>

```
##  处理设备旋转问题 
fragment setRetainInstance(true);