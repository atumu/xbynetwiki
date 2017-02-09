title: android_toolbar 

#  Android ToolBar 
2014的 google i/o 发表令多数人为之一亮的 material design
toolbar，这是用来取代过去 actionbar 的控件，而现在于 material design 中也对之有一个统一名称：app bar，在未来的 android app 中，就以 toolbar 这个元件来实作之。
概述
Android 3.0  Android 推了 ActionBar 这个控件，而到了2013 年 Google 开始大力地推动所谓的 android style，想要逐渐改善过去 android 纷乱的界面设计，希望让终端使用者尽可能在 android 手机有个一致的操作体验。ActionBar 过去最多人使用的两大套件就是 ActionBarSherlock 以及官方提供在 support library v 7 里的 AppCompat。
Toolbar的诞生，也意味着官方在某些程度上认为 ActionBar 限制了 android app 的开发与设计的弹性，而在 material design 也对之做了名称的定义：App bar。

Material Design的Theme
  * @android:style/Theme.Material (dark version)
  * @android:style/Theme.Material.Light (light version)
  * @android:style/Theme.Material.Light.DarkActionBar
与之对应的Compat Theme:
  * Theme.AppCompat
  * Theme.AppCompat.Light
  * Theme.AppCompat.Light.DarkActionBar

示例：
https://github.com/tekinarslan/AndroidMaterialDesignToolbar
https://github.com/mengdd/HelloActivityAndFragment
https://github.com/hongyangAndroid/Android_Blog_Demos/tree/master/blogcodes/src/main/java/com/zhy/blogcodes/toolbar

https://github.com/mosil/Android-Mosil-Sample-Toolbar 
https://github.com/hanks-zyh/ToolBar


##  基础 
1.首先项目gradle中添加:appcompat-v7依赖
2.确保Activity继承**AppCompatActivity**
3.在application设置中使用NoActionBar的主题:
```

<application
    android:theme="@style/Theme.AppCompat.Light.NoActionBar"
/>

```
4.把Toolbar写在布局中
```

<android.support.v7.widget.Toolbar
   android:id="@+id/toolbar"
   android:layout_height="wrap_content"
   android:minHeight="?attr/actionBarSize"
   android:layout_width="match_parent"
   android:background="?attr/colorPrimary"
   android:elevation="4dp"
   android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
   app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>

```

请记得用 support v7 里的 toolbar，不然然只有 API Level 21 也就是 Android 5.0 以上的版本才能使用。
程序 (Java)
在 MainActivity.java 中onCreate中加入 Toolbar 的声明：
```

Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
setSupportActionBar(toolbar);//声明后，再将之用 setSupportActionBar 设定，Toolbar即能取代原本的 actionbar 了

```

定义menu:
```

<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_android"
        android:icon="@drawable/ic_android_black_24dp"
        android:title="@string/action_android"
        app:showAsAction="always" />
    <item
        android:id="@+id/action_favourite"
        android:icon="@drawable/ic_favorite_black_24dp"
        android:title="@string/action_favourite"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/action_settings"
        android:title="@string/action_settings"
        app:showAsAction="never" />
</menu>

```
然后在代码中inflate和处理它的点击事件:
```

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    Log.i(TAG, "onCreateOptionsMenu()");
    getMenuInflater().inflate(R.menu.menu_activity_main, menu);
    return super.onCreateOptionsMenu(menu);
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {
        case R.id.action_android:
            Log.i(TAG, "action android selected");
            return true;
        case R.id.action_favourite:
            Log.i(TAG, "action favourite selected");
            return true;
        case R.id.action_settings:
            Log.i(TAG, "action settings selected");
            return true;
        default:
            return super.onOptionsItemSelected(item);
    }
}

```
Menu Item，依然支持在menu/menu_main.xml去声明，然后复写onCreateOptionsMenu和**onOptionsItemSelected**即可。
**可选方案**也可以通过**toolbar.setOnMenuItemClickListener**实现点击MenuItem的回调。
```

  toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                return false;
            }
        });

```

**添加向上返回的action**
添加向上返回parent的action:
```

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_toolbar_demo);
    setSupportActionBar(toolbar);

    // add a left arrow to back to parent activity,
    // no need to handle action selected event, this is handled by super
    getSupportActionBar().setDisplayHomeAsUpEnabled(true);
}

```
然后只需要在manifest中指定parent:
```

<activity
    android:name=".toolbar.ToolbarDemoActivity"
    android:parentActivityName=".MainActivity">
    <!-- Parent activity meta-data to support 4.0 and lower -->
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value=".MainActivity" />
      </activity>

```

##  自定义颜色(Customization color) 
![](/data/dokuwiki/android/pasted/20161009-233139.png)
![](/data/dokuwiki/android/pasted/20161009-001338.png)
  * colorPrimaryDark（状态栏底色）：在风格 (styles) 或是主题 (themes) 里进行设定。
  * App bar 底色。这个设定分为二，若你的 android app 仍是使用 actionbar ，则直接在风格 (styles) 或是主题 (themes) 里进行设定 colorPrimary 参数即可；
可若是采用 toolbar 的话，则要在界面 (layout) 里面设定 toolbar 控件的 **background** 属性。
  * navigationBarColor（导航栏底色）：仅能在 API v21 也就是 Android 5 以后的版本中使用， 因此要将之设定在 res/values-v21/styles.xml 里面。
也因此在这个阶段，我们需要设定的地方有三，一是 style中(res/values/styles.xml)
```

<style name="AppTheme.Base" parent="Theme.AppCompat.Light.NoActionBar">
 <!-- <item name="windowActionBar">false</item> -->
 <!-- <item name="android:windowNoTitle">true</item> -->
  <!-- Actionbar color -->
  <item name="colorPrimary">@color/accent_material_dark</item>
  <!--Status bar color-->
  <item name="colorPrimaryDark">@color/accent_material_light</item>
  <!--Window color-->
  <item name="android:windowBackground">@color/dim_foreground_material_dark</item>
</style>

```
再来是 v21 的style中 (res/values-v21/styles.xml)
```

<style name="AppTheme" parent="AppTheme.Base">
  <!--Navigation bar color-->
  <item name="android:navigationBarColor">@color/accent_material_light</item>
</style>

```
最后，就是为了本篇的主角 – Toolbar 的 background 进行设定。
```

<android.support.v7.widget.Toolbar
  android:id="@+id/toolbar"
  android:layout_height="?attr/actionBarSize"
  android:layout_width="match_parent"
  android:background="?attr/colorPrimary" >
</android.support.v7.widget.Toolbar>

```
##  控件 (component) 
在还未于 <android.support.v7.widget.Toolbar/>  标签中，自行添加元件的 toolbar 有几个大家常用的元素可以使用，请先见下图：
![](/data/dokuwiki/android/pasted/20161009-001757.png)
```

Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
 
// App Logo
toolbar.setLogo(R.drawable.ic_launcher);
// Title
toolbar.setTitle("My Title");
// Sub Title
toolbar.setSubtitle("Sub title");
 
setSupportActionBar(toolbar);
 
// Navigation Icon 要設定在 setSupoortActionBar之后才有作用
// 否則會出現 back button
toolbar.setNavigationIcon(R.drawable.ab_android);

```
` 这边要留意的是setNavigationIcon需要放在 setSupportActionBar之后才会生效。 `
**可选方案当然如果你喜欢，也可以在布局文件中去设置部分属性：**
```

 <android.support.v7.widget.Toolbar
        android:id="@+id/id_toolbar"
        app:title="App Title"
        app:subtitle="Sub Title"
        app:navigationIcon="@drawable/ic_toc_white_24dp"
        android:layout_height="wrap_content"
        android:minHeight="?attr/actionBarSize"
        android:layout_width="match_parent"
        android:background="?attr/colorPrimary"/>

```



Action Views and Action Providers
To add an action view, create an <item> element in the toolbar's menu resource, as Add Action Buttons describes. Add one of the following two attributes to the <item> element:
  * actionViewClass: The class of a widget that implements the action.
  * actionLayout: A layout resource describing the action's components.
```

<item android:id="@+id/action_search"
     android:title="@string/action_search"
     android:icon="@drawable/ic_search"
      <!-- app: -->
     app:showAsAction="ifRoom|collapseActionView"
     app:actionViewClass="android.support.v7.widget.SearchView" />

```
getActionView的正确操作方式：
```

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.main_activity_actions, menu);

    MenuItem searchItem = menu.findItem(R.id.action_search);
    SearchView searchView =
            (SearchView) MenuItemCompat.getActionView(searchItem);

    // Configure the search info and add any event listeners...

    return super.onCreateOptionsMenu(menu);
}

```

Add an Action Provider
```

<item android:id="@+id/action_share"
    android:title="@string/share"
    app:showAsAction="ifRoom"
    app:actionProviderClass="android.support.v7.widget.ShareActionProvider"/>

```

在这样的架构设计下，**ToolBar直接成了Layout中可以控制的东西，相对于过去的actionbar来说，设计与可操控性大幅提升。**
最后再附上一个界面上常用的属性说明图：
![](/data/dokuwiki/android/pasted/20161009-002225.png)
这里按照图中从上到下的顺序做个简单的说明：

colorPrimaryDark

状态栏背景色。
在 style 的属性中设置。
textColorPrimary

App bar 上的标题与更多菜单中的文字颜色。
在 style 的属性中设置。
App bar 的背景色

Actionbar 的背景色设定在 style 中的 colorPrimary。
Toolbar 的背景色在layout文件中设置background属性。
colorAccent

各控制元件(如：check box、switch 或是 radoi) 被勾选 (checked) 或是选定 (selected) 的颜色。
在 style 的属性中设置。
colorControlNormal

各控制元件的预设颜色。
在 style 的属性中设置
windowBackground

App 的背景色。
在 style 的属性中设置
navigationBarColor

导航栏的背景色，但只能用在 API Level 21 (Android 5) 以上的版本
在 style 的属性中设置


最后需要注意的是：**使用material主题的时候，必须设定targetSdkVersion = 21，否则界面看起来是模糊的 **      


##  在Fragment中使用Toolbar: 
在Fragment中使用Toolbar的步骤和Activity差不多.
在Fragment布局中添加一个Toolbar, 然后find它, 然后调用Activity的方法来把它设置成ActionBar:
```

((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);

```
注意此处有一个强转, 必须是AppCompatActivity才有这个方法.
但是此时运行到Fragment之后, 发现Toolbar上的文字和按钮全是Activity传过来的, 这是因为只有Activity的onCreateOptionsMenu()被调用了, 但是Fragment的并没有被调用.
在Fragment中加上这句:

` setHasOptionsMenu(true); `

此时Fragment的onCreateOptionsMenu()回调会被调到了, 但是inflate出的按钮和Activity中的actions加在一起显示出来了.
因为Activity的onCreateOptionsMenu()会在之前调用到.
于是Fragment中的写成这样:
```

@Override
public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
    Log.e(TAG, "onCreateOptionsMenu()");
    menu.clear();
    inflater.inflate(R.menu.menu_parent_fragment, menu);
}

```
即先clear()一下, 这样按钮就只有Fragment中设置的自己的了, 不会有Activity中的按钮.

##  在嵌套的子Fragment中使用Toolbar 
前面已经介绍过, Fragment可以嵌套使用: [Android Fragment使用(二) 嵌套Fragments (Nested Fragments) 的使用及常见错误](http://www.cnblogs.com/mengdd/p/5552721.html)
那么在前面的Fragment中再显示一个子Fragment, 并且又带有一个不一样的Toolbar, 还需要哪些处理呢?
首先, java代码中还是需要有:
```

setHasOptionsMenu(true)
((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);

```
然后根据是否需要菜单按钮, 覆写onCreateOptionsMenu()方法来inflate自己的menu文件即可.
感觉和在普通的Fragment中使用Toolbar作为ActionBar并没有什么区别.
但是如果你的多个Fragment有不同的Toolbar菜单选项, 如果你没有懂得其中的原理, 可能就会出现一些混乱.
下面来解说一下相关的方法.
**onCreateOptionsMenu()方法的调用
一旦调用((AppCompatActivity) getActivity()).setSupportActionBar(toolbar);
就会导致Activity.onCreateOptionsMenu()方法的调用, 而Activity会根据其中Fragment是否设置了setHasOptionsMenu(true)来调用Fragment的onCreateOptionsMenu()方法, 调用顺序是树形的, 按层级调用, 中间如果有false则跳过.**

假设当前Activity, Parent Fragment和Child Fragment中都设置了自己的Toolbar为ActionBar.
在打开Child fragment的时候, onCreateOptionsMenu()的调用顺序是.Activity -> Parent -> Child. 此时parent和child fragment都设置了setHasOptionsMenu(true).

关于这个, 还有以下几种情况:
- 如果Parent的`setHasOptionsMenu(false)`, Child为true, 则Parent的`onCreateOptionsMenu()`不会调用, 打开Child的时候Activity -> Child.
- 如果Child的`setHasOptionsMenu(false)`, Parent为true, 则打开Child的时候仍然会调用Activity和Parent的onCreateOptionsMenu()方法.
- 如果Parent和Child都置为false, 打开Parent和Child Fragment的时候都会调用Activity的onCreateOptionsMenu()方法.
仅仅是child Fragment的show() hide()的切换, activity和parent Fragment的onCreateOptionsMenu()也会重新进入.

**上面的机制常常是导致Toolbar上面的按钮混淆错乱的原因.**
举个例子:
如果我们现在Activity和Parent Fragment有不同的Toolbar按钮, 但是Child只有文字, 没有按钮.
很显然我们不需要给child写menu文件, 也不需要覆写child里的onCreateOptionsMenu()方法.
但是此时不管怎样, parent的onCreateOptionsMenu()方法都会被调用, 这样我们打开child的时候, toolbar上就神奇地出现了parent里的按钮.
**这种情况如何解决呢?
可以在parent中加一个条件, 当没有child fragment的时候才做inflate的工作:**

```

@Override
public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
    Log.e(TAG, "onCreateOptionsMenu()");
    menu.clear();
    if (getChildFragmentManager().getBackStackEntryCount() == 0) {
        inflater.inflate(R.menu.menu_parent_fragment, menu);
    }
}

```
另外, 除了setSupportActionBar()之外, 如果我们想主动触发 onCreateOptionsMenu()方法的调用, 可以用**invalidateOptionsMenu()**方法.

**onOptionsItemSelected()方法的调用
在Activity和其中的Fragment都有options menu的时候, 需要注意menu item的id不要重复.
因为点击事件的分发也是从Activity开始分发下去的, 如果child fragment中有个选项的id和Activity中一个选项的id重复了, 则在Activity中就会将其处理, 不会继续分发.**

**有嵌套Fragment时 Back键处理
之前没有嵌套Fragment的情况下, 只要将Fragment加入到Back Stack中, 那么按下Back键的时候pop动作是系统自动做好的.
虽然在添加child fragment的时候将其加入到back stack中, 但是按back键的时候仍然是将parent fragment弹出, 只剩下Activity.这是因为back键只检查第一层Fragment的back stack, 对于child fragment, 需要在其parent中自己处理.**
比如这样处理:
在Activity中
```

@Override
public void onBackPressed() {
    Fragment fragment = getSupportFragmentManager().findFragmentById(android.R.id.content);
    if (fragment instanceof ToolbarFragment) {
        if (((ToolbarFragment) fragment).onBackPressed()) {
            return;
        }
    }
    super.onBackPressed();
}

```
其中ToolbarFragment是直接加在Activity中作为parent fragment的.
**在parent fragment中(即ToolbarFragment中):**
```

public boolean onBackPressed() {
    return getChildFragmentManager().popBackStackImmediate();

```
##  Fragment单独使用Toolbar而不与ActionBar进行关联 
对于Fragment使用Toolbar思维被局限了，Menu的操作也不用在onCreateOptionsMenu方法，**直接使用ToolBar的inflateMenu方法**，Menu的事件也是独立的，**需要通过设置ToolBar的setOnMenuItemClickListener来实现**。
```

Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
 
// App Logo
toolbar.setLogo(R.drawable.ic_launcher);
// Title
toolbar.setTitle("My Title");
// Sub Title
toolbar.setSubtitle("Sub title");
 
//setSupportActionBar(toolbar);
 toolbar.inflateMenu(R.menu.activity_main);
// Navigation Icon 要設定在 setSupoortActionBar后 才有作用
// 否則會出現 back button 
toolbar.setNavigationIcon(R.drawable.ab_android);
toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                
            }
        });
// Menu item click 设定在 setSupportActionBar 之后才有作用
toolbar.setOnMenuItemClickListener(onMenuItemClick);
private Toolbar.OnMenuItemClickListener onMenuItemClick = new Toolbar.OnMenuItemClickListener() {
  @Override
  public boolean onMenuItemClick(MenuItem menuItem) {
    String msg = "";
    switch (menuItem.getItemId()) {
      case R.id.action_edit:  
        break;
    }
    return true;
  }
};

```
自定义布局
title修改为居中
```

<android.support.v7.widget.Toolbar
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
	android:layout_height="wrap_content"
    android:background="@mipmap/bg_title"
    android:minHeight="?actionBarSize">
    <TextView
        android:id="@+id/toolbar_title"
        style="@style/TextAppearance.AppCompat.Widget.ActionBar.Title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center" />
</android.support.v7.widget.Toolbar>

```
禁用系统的title显示，即setDisplayShowTitleEnabled方法
实现将布局的内容延伸到状态栏
styles.xml
**<item name="android:windowTranslucentStatus" tools:targetApi="19">true</item>**

##  整合ToolBar，DrawerLayout，ActionBarDrawerToggle 
![](/data/dokuwiki/android/pasted/20161009-233750.png)
大致思路
整体实现还是比较容易的，首先需要引入DrawerLayout（如果你对DrawerLayout不了解，可以参考  
[Android DrawerLayout 高仿QQ5.2双向侧滑菜单](http://blog.csdn.net/lmj623565791/article/details/41531475))，然后去初始化mActionBarDrawerToggle，mActionBarDrawerToggle实际上是个DrawerListener，设置mDrawerLayout.setDrawerListener(mActionBarDrawerToggle);就已经能够实现上面点击Nav Icon切换效果了。当然了细节还是挺多的。
我们的效果图，左侧菜单为Fragment，内容区域为Fragment，点击左侧菜单切换内容区域的Fragment即可。
activity_main.xml
```

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent"
    android:background="#ffffffff"
    xmlns:app="http://schemas.android.com/apk/res-auto">
 
    <!--app:subtitle="Sub Title"-->
    <android.support.v7.widget.Toolbar
        android:id="@+id/id_toolbar"
        app:title="App Title"
        app:navigationIcon="@drawable/ic_toc_white_24dp"
        android:layout_height="wrap_content"
        android:minHeight="?attr/actionBarSize"
        android:layout_width="match_parent"
        android:background="?attr/colorPrimary" />
 
    <android.support.v4.widget.DrawerLayout
        android:id="@+id/id_drawerlayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
 
        <FrameLayout
            android:id="@+id/id_content_container"
            android:layout_width="match_parent"
            android:layout_height="match_parent"></FrameLayout>
 
        <FrameLayout
            android:id="@+id/id_left_menu_container"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="left"
            android:background="#ffffffff"></FrameLayout>
 
 
    </android.support.v4.widget.DrawerLayout>
 
 
</LinearLayout>

```
DrawerLayout中包含两个FrameLayout，分别放内容区域和左侧菜单的Fragment。
LeftMenuFragment
```

package com.zhy.toolbar;
 
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.ListFragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ListView;
 
/**
 * Created by zhy on 15/4/26.
 */
public class LeftMenuFragment extends ListFragment {
 
    private static final int SIZE_MENU_ITEM = 3;
 
    private MenuItem[] mItems = new MenuItem[SIZE_MENU_ITEM];
 
    private LeftMenuAdapter mAdapter;
 
    private LayoutInflater mInflater;
 
    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
 
        mInflater = LayoutInflater.from(getActivity());
 
        MenuItem menuItem = null;
        for (int i = 0; i < SIZE_MENU_ITEM; i++) {
            menuItem = new MenuItem(getResources().getStringArray(R.array.array_left_menu)[i], false, R.drawable.music_36px, R.drawable.music_36px_light);
            mItems[i] = menuItem;
        }
    }
 
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return super.onCreateView(inflater, container, savedInstanceState);
    }
 
    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
 
        setListAdapter(mAdapter = new LeftMenuAdapter(getActivity(), mItems));
 
    }
 
    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
        super.onListItemClick(l, v, position, id);
 
        if (mMenuItemSelectedListener != null) {
            mMenuItemSelectedListener.menuItemSelected(((MenuItem) getListAdapter().getItem(position)).text);
        }
 
        mAdapter.setSelected(position);
 
    }
 
 
    //选择回调的接口
    public interface OnMenuItemSelectedListener {
        void menuItemSelected(String title);
    }
    private OnMenuItemSelectedListener mMenuItemSelectedListener;
 
    public void setOnMenuItemSelectedListener(OnMenuItemSelectedListener menuItemSelectedListener) {
        this.mMenuItemSelectedListener = menuItemSelectedListener;
    }
 
 
 
}

```
继承自ListFragment，主要用于展示各个Item，提供了一个选择Item的回调，这个需要在Activity中去注册处理。
LeftMenuAdapter
```


package com.zhy.toolbar;
 
import android.content.Context;
import android.graphics.Color;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.ImageView;
import android.widget.TextView;
 
/**
 * Created by zhy on 15/4/26.
 */
public class LeftMenuAdapter extends ArrayAdapter<MenuItem> {
 
 
    private LayoutInflater mInflater;
 
    private int mSelected;
 
 
    public LeftMenuAdapter(Context context, MenuItem[] objects) {
        super(context, -1, objects);
 
        mInflater = LayoutInflater.from(context);
 
    }
 
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
 
 
        if (convertView == null) {
            convertView = mInflater.inflate(R.layout.item_left_menu, parent, false);
        }
 
        ImageView iv = (ImageView) convertView.findViewById(R.id.id_item_icon);
        TextView title = (TextView) convertView.findViewById(R.id.id_item_title);
        title.setText(getItem(position).text);
        iv.setImageResource(getItem(position).icon);
        convertView.setBackgroundColor(Color.TRANSPARENT);
 
        if (position == mSelected) {
            iv.setImageResource(getItem(position).iconSelected);
            convertView.setBackgroundColor(getContext().getResources().getColor(R.color.state_menu_item_selected));
        }
 
        return convertView;
    }
 
    public void setSelected(int position) {
        this.mSelected = position;
        notifyDataSetChanged();
    }
 
 
}
 
 
package com.zhy.toolbar;
 
public class MenuItem {
 
    public MenuItem(String text, boolean isSelected, int icon, int iconSelected) {
        this.text = text;
        this.isSelected = isSelected;
        this.icon = icon;
        this.iconSelected = iconSelected;
    }
 
    boolean isSelected;
    String text;
    int icon;
    int iconSelected;
}

```
Adapter没撒说的~~提供了一个setSection方法用于设置选中Item的样式什么的。 
接下来看ContentFragment，仅仅只是一个TextView而已，所以代码也比较easy。
```

package com.zhy.toolbar;
 
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.text.TextUtils;
import android.view.Gravity;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
 
/**
 * Created by zhy on 15/4/26.
 */
public class ContentFragment extends Fragment {
 
    public static final String KEY_TITLE = "key_title";
    private String mTitle;
 
    public static ContentFragment newInstance(String title) {
        ContentFragment fragment = new ContentFragment();
        Bundle bundle = new Bundle();
        bundle.putString(KEY_TITLE, title);
        fragment.setArguments(bundle);
        return fragment;
    }
 
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
 
        TextView tv = new TextView(getActivity());
        String title = (String) getArguments().get(KEY_TITLE);
        if (!TextUtils.isEmpty(title))
        {
            tv.setGravity(Gravity.CENTER);
            tv.setTextSize(40);
            tv.setText(title);
        }
 
        return tv;
    }
}

```
提供newInstance接收一个title参数去实例化它。
最后就是我们的MainActivity了，负责管理各种Fragment。
```

MainActivity
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentTransaction;
import android.support.v4.widget.DrawerLayout;
import android.support.v7.app.ActionBarDrawerToggle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.text.TextUtils;import android.view.Gravity;
import java.util.List;
public class MainActivity extends AppCompatActivity {
 
    private ActionBarDrawerToggle mActionBarDrawerToggle;    
  private DrawerLayout mDrawerLayout;    
  private Toolbar mToolbar;    
  private LeftMenuFragment mLeftMenuFragment;    
  private ContentFragment mCurrentFragment;    
  private String mTitle;    
  private static final String TAG = "com.zhy.toolbar";    
  private static final String KEY_TITLLE = "key_title";    
  @Override
    protected void onCreate(Bundle savedInstanceState) {        
      super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        initToolBar();
        initViews();        //恢复title
        restoreTitle(savedInstanceState);
 
        FragmentManager fm = getSupportFragmentManager();        //查找当前显示的Fragment
        mCurrentFragment = (ContentFragment) fm.findFragmentByTag(mTitle);        
      if (mCurrentFragment == null) {
            mCurrentFragment = ContentFragment.newInstance(mTitle);
            fm.beginTransaction().add(R.id.id_content_container, mCurrentFragment, mTitle).commit();
        }
 
        mLeftMenuFragment = (LeftMenuFragment) fm.findFragmentById(R.id.id_left_menu_container);        
      if (mLeftMenuFragment == null) {
            mLeftMenuFragment = new LeftMenuFragment();
            fm.beginTransaction().add(R.id.id_left_menu_container, mLeftMenuFragment).commit();
        }        //隐藏别的Fragment，如果存在的话
        List<Fragment> fragments = fm.getFragments();        
      if (fragments != null)            
          for (Fragment fragment : fragments) {                
            if (fragment == mCurrentFragment || fragment == mLeftMenuFragment) continue;
                fm.beginTransaction().hide(fragment).commit();
            }        //设置MenuItem的选择回调
        mLeftMenuFragment.setOnMenuItemSelectedListener(new LeftMenuFragment.OnMenuItemSelectedListener() {            
          @Override
            public void menuItemSelected(String title) {
 
                FragmentManager fm = getSupportFragmentManager();
                ContentFragment fragment = (ContentFragment) getSupportFragmentManager().findFragmentByTag(title);                
              if (fragment == mCurrentFragment) {
                    mDrawerLayout.closeDrawer(Gravity.LEFT);                    
                return;
                }
 
                FragmentTransaction transaction = fm.beginTransaction();
                transaction.hide(mCurrentFragment);                
              if (fragment == null) {
                    fragment = ContentFragment.newInstance(title);
                    transaction.add(R.id.id_content_container, fragment, title);
                } else {
                    transaction.show(fragment);
                }
                transaction.commit();
 
                mCurrentFragment = fragment;
                mTitle = title;
                mToolbar.setTitle(mTitle);
                mDrawerLayout.closeDrawer(Gravity.LEFT);
 
 
            }
        });
 
    }    private void restoreTitle(Bundle savedInstanceState) {        
      if (savedInstanceState != null)
            mTitle = savedInstanceState.getString(KEY_TITLLE);        
      if (TextUtils.isEmpty(mTitle)) {
            mTitle = getResources().getStringArray(
                    R.array.array_left_menu)[0];
        }
 
        mToolbar.setTitle(mTitle);
    }    @Override
    protected void onSaveInstanceState(Bundle outState) {        
      super.onSaveInstanceState(outState);
        outState.putString(KEY_TITLLE, mTitle);
    }    private void initToolBar() {
 
        Toolbar toolbar = mToolbar = (Toolbar) findViewById(R.id.id_toolbar);        // App Logo
        // toolbar.setLogo(R.mipmap.ic_launcher);
        // Title
        toolbar.setTitle(getResources().getStringArray(R.array.array_left_menu)[0]);        // Sub Title
        // toolbar.setSubtitle("Sub title");//        toolbar.setTitleTextAppearance();
 
 
        setSupportActionBar(toolbar);        //Navigation Icon
        toolbar.setNavigationIcon(R.drawable.ic_toc_white_24dp);        /*
        toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                return false;
            }
        });*/
 
    }    private void initViews() {
 
        mDrawerLayout = (DrawerLayout) findViewById(R.id.id_drawerlayout);
 
        mActionBarDrawerToggle = new ActionBarDrawerToggle(this,
                mDrawerLayout, mToolbar, R.string.open, R.string.close);
        mActionBarDrawerToggle.syncState();
        mDrawerLayout.setDrawerListener(mActionBarDrawerToggle);
 
 
    }
}

```
内容区域的切换是通过Fragment hide和show实现的，毕竟如果用replace，如果Fragment的view结构比较复杂，可能会有卡顿。当然了，注意每个Fragment占据的内存情况，如果内存不足，可能需要改变实现方式。 
对于旋转屏幕或者应用长时间置于后台，Activity重建的问题，做了简单的处理。



参考：
https://developer.android.com/training/appbar/setting-up.html
http://www.cnblogs.com/mengdd/p/5590634.html
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0430/2814.html
http://wuxiaolong.me/2015/12/21/fragmentToolbar/
http://wuxiaolong.me/2015/11/10/toolbar/


http://blog.csdn.net/a553181867/article/details/51336899
http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1118/2006.html
http://m.blog.csdn.net/article/details?id=51965925