title: android官方文档学习之supporting_swipe-to-refresh 

#  android官方文档学习之Supporting Swipe-to-Refresh 
` 前提需要android.support.v4支持库。 `
官方文档地址:http://developer.android.com/training/swipe/index.html
##  Adding Swipe-to-Refresh To Your App 
 SwipeRefreshLayout字面意思就是下拉刷新的布局，继承自ViewGroup，在support v4兼容包下.**（android.support.v4.widget.SwipeRefreshLayout），但必须把你的support library的版本升级到19.1**。 提到下拉刷新大家一定对ActionBarPullToRefresh比较熟悉，而如今google推出了更官方的下拉刷新组件，这无疑是对开发者来说比较好的消息。利用这个组件可以很方便的实现Google Now的刷新效果，见下图：
![](/data/dokuwiki/android/pasted/20150608-155912.png)
##  Add the SwipeRefreshLayout Widget 

xml布局文件
只要在需要刷新的控件最外层加上SwipeRefreshLayout，然后他的child首先是可滚动的view，如` ListView、GridView `。
```

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:id="@+id/container"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    tools:ignore="MergeRootFrame" >  
  
    <android.support.v4.widget.SwipeRefreshLayout  
        android:id="@+id/swipe_container"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent" >  
  
        <ListView  
            android:id="@+id/list"  
            android:layout_width="match_parent"  
            android:layout_height="match_parent" >  
        </ListView>  
  
    </android.support.v4.widget.SwipeRefreshLayout>  
  
</FrameLayout>  

```
```

public class SwipRefreshLayoutActivity extends Activity implements  
        SwipeRefreshLayout.OnRefreshListener {  
    private SwipeRefreshLayout swipeLayout;  
    private ListView listView;  
    private ListViewAdapter adapter;  
    private ArrayList<SoftwareClassificationInfo> list; private boolean isRefresh = false;//是否刷新中  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.swipe_refresh_layout);  
        swipeLayout = (SwipeRefreshLayout) findViewById(R.id.swipe_container);  
        swipeLayout.setOnRefreshListener(this);  
        //加载颜色是循环播放的，只要没有完成刷新就会一直循环，color1>color2>color3>color4  
        swipeLayout.setColorScheme(android.R.color.white,  
                android.R.color.holo_green_light,  
                android.R.color.holo_orange_light, android.R.color.holo_red_light);  
  
        list = new ArrayList<SoftwareClassificationInfo>();  
        list.add(new SoftwareClassificationInfo(1, "asdas"));  
        listView = (ListView) findViewById(R.id.list);  
        adapter = new ListViewAdapter(this, list);  
        listView.setAdapter(adapter);  
    }  
  
    public void onRefresh() { if(!isRefresh){ isRefresh = true;  
        new Handler().postDelayed(new Runnable() {  
            public void run() {  
                swipeLayout.setRefreshing(false);  
                list.add(new SoftwareClassificationInfo(2, "ass"));  
                adapter.notifyDataSetChanged(); isRefresh= false;  
            }  
        }, 3000); }  
    }  
}  

```
主要方法：
  * setOnRefreshListener(OnRefreshListener): 为布局添加一个Listener
  * setRefreshing(boolean): 显示或隐藏刷新进度条
  * isRefreshing(): 检查是否处于刷新状态
  * setColorScheme(): 设置进度条的颜色主题，最多能设置四种
##  Add a Refresh Action to the Action Bar 
You should add the **refresh action as a menu item, rather than as a button,** by setting the attribute`  android:showAsAction=never `
```

<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:id="@+id/menu_refresh"
        android:showAsAction="never"
        android:title="@string/menu_refresh"/>
</menu>

```
##  Responding to a Refresh Request  
```

/*
 * Sets up a SwipeRefreshLayout.OnRefreshListener that is invoked when the user
 * performs a swipe-to-refresh gesture.
 */
mySwipeRefreshLayout.setOnRefreshListener(
    new SwipeRefreshLayout.OnRefreshListener() {
        @Override
        public void onRefresh() {
            Log.i(LOG_TAG, "onRefresh called from SwipeRefreshLayout");

            // This method performs the actual data-refresh operation.
            // The method calls setRefreshing(false) when it's finished.
            myUpdateOperation();
        }
    }
);

``` 
###  Respond to the Refresh Action 
```

/*
 * Listen for option item selections so that we receive a notification
 * when the user requests a refresh by selecting the refresh action bar item.
 */
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {

        // Check if user triggered a refresh:
        case R.id.menu_refresh:
            Log.i(LOG_TAG, "Refresh menu item selected");

            // Signal SwipeRefreshLayout to start the progress indicator
            mySwipeRefreshLayout.setRefreshing(true);

            // Start the refresh background task.
            // This method calls setRefreshing(false) when it's finished.
            myUpdateOperation();

            return true;
    }

    // User didn't trigger a refresh, let the superclass handle this action
    return super.onOptionsItemSelected(item);

}

```

参考：
http://blog.csdn.net/jabony/article/details/22890793