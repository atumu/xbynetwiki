title: actionbar 

#  Android ActionBar完全解析，使用官方推荐的最佳导航栏 
http://blog.csdn.net/guolin_blog/article/details/18234477
本篇文章主要内容来自于Android Doc，我翻译之后又做了些加工，英文好的朋友也可以直接去读原文。
http://developer.android.com/guide/topics/ui/actionbar.html
Action Bar是一种新増的导航栏功能，在Android 3.0之后加入到系统的API当中，它标识了用户当前操作界面的位置，并提供了额外的用户动作、界面导航等功能。使用ActionBar的好处是，它可以给提供一种全局统一的UI界面，使得用户在使用任何一款软件时都懂得该如何操作，并且ActionBar还可以自动适应各种不同大小的屏幕。下面是一张使用ActionBar的界面截图：
![](/data/dokuwiki/android/pasted/20150528-081403.png)
其中，[1]是ActionBar的图标，[2]是两个action按钮，[3]是overflow按钮。
由于Action Bar是在3.0以后的版本中加入的，如果想在2.x的版本里使用ActionBar的话则需要引入Support Library，不过3.0之前版本的市场占有率已经非常小了，这里简单起见我们就不再考虑去做向下兼容，而是只考虑4.0以上版本的用法。
##  添加和移除Action Bar 

ActionBar的添加非常简单，只需要在AndroidManifest.xml中` 指定Application或Activity的theme是Theme.Holo或其子类就可以了 `，而使用Eclipse创建的项目自动就会将Application的theme指定成Theme.Holo，所以ActionBar默认都是显示出来的。新建一个空项目并运行，效果如下图所示：
![](/data/dokuwiki/android/pasted/20150528-081510.png)
而如果想要移除ActionBar的话通常有两种方式
  * 一是` 将theme指定成Theme.Holo.NoActionBar `，表示使用一个不包含ActionBar的主题，
  * 二是在Activity中调用以下方法：
```

ActionBar actionBar = getActionBar();  
actionBar.hide(); 

```
##  修改Action Bar的图标和标题 

默认情况下，系统会使用<application>或者<activity>中icon属性指定的图片来作为ActionBar的图标，但是我们也可以改变这一默认行为。**如果我们想要使用另外一张图片来作为ActionBar的图标，可以在<application>或者<activity>中通过` logo属性 `来进行指定。**比如项目的res/drawable目录下有一张weather.png图片，就可以在AndroidManifest.xml中这样指定：
```

<activity  
    android:name="com.example.actionbartest.MainActivity"  
    android:logo="@drawable/weather" >  
</activity>

```
![](/data/dokuwiki/android/pasted/20150528-081706.png)
ActionBar的图标已经修改成功了，那么标题中的内容该怎样修改呢？其实也很简单，使用label属性来指定一个字符串就可以了，如下所示：
```

<activity  
    android:name="com.example.actionbartest.MainActivity"  
    android:label="天气"  
    android:logo="@drawable/weather" >  
</activity>  

```
![](/data/dokuwiki/android/pasted/20150528-081737.png)
##  添加Action按钮 

ActionBar还可以根据应用程序当前的功能来提供与其相关的Action按钮，这些按钮都会以图标或文字的形式直接显示在ActionBar上。当然，如果按钮过多，ActionBar上显示不完，多出的一些按钮可以隐藏在overflow里面（最右边的三个点就是overflow按钮），点击一下overflow按钮就可以看到全部的Action按钮了。
当Activity启动的时候，系统会调用Activity的**onCreateOptionsMenu()**方法来取出所有的Action按钮，**我们只需要在这个方法中去加载一个menu资源**，并把所有的Action按钮都定义在资源文件里面就可以了。
那么我们先来看下menu资源文件该如何定义，代码如下所示：
```

<menu xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    tools:context="com.example.actionbartest.MainActivity" >  
  
    <item  
        android:id="@+id/action_compose"  
        android:icon="@drawable/ic_action_compose"  
        android:showAsAction="always"  
        android:title="@string/action_compose"/>  
    <item  
        android:id="@+id/action_delete"  
        android:icon="@drawable/ic_action_delete"  
        android:showAsAction="always"  
        android:title="@string/action_delete"/>  
    <item  
        android:id="@+id/action_settings"  
        android:icon="@drawable/ic_launcher"  
        android:showAsAction="never"  
        android:title="@string/action_settings"/>  
  
</menu> 

```
**showAsAction**则指定了该按钮显示的位置，主要有以下几种值可选：
  * **always**表示永远显示在ActionBar中，如果屏幕空间不够则无法显示，
  * **ifRoom**表示屏幕空间够的情况下显示在ActionBar中，不够的话就显示在overflow中，
  * **never**则表示永远显示在overflow中。

接着，重写Activity的**onCreateOptionsMenu()**方法，代码如下所示：
```

@Override  
public boolean onCreateOptionsMenu(Menu menu) {  
    MenuInflater inflater = getMenuInflater();  
    inflater.inflate(R.menu.main, menu);  
    return super.onCreateOptionsMenu(menu);  
}  

```
![](/data/dokuwiki/android/pasted/20150528-082103.png)
##  响应Action按钮的点击事件 

当用户点击Action按钮的时候，系统会调用Activity的**onOptionsItemSelected()**方法，通过方法传入的MenuItem参数，我们可以调用它的getItemId()方法和menu资源中的id进行比较，从而辨别出用户点击的是哪一个Action按钮，比如：
```

@Override  
public boolean onOptionsItemSelected(MenuItem item) {  
    switch (item.getItemId()) {  
    case R.id.action_compose:  
        Toast.makeText(this, "Compose", Toast.LENGTH_SHORT).show();  
        return true;  
    case R.id.action_delete:  
        Toast.makeText(this, "Delete", Toast.LENGTH_SHORT).show();  
        return true;  
    case R.id.action_settings:  
        Toast.makeText(this, "Settings", Toast.LENGTH_SHORT).show();  
        return true;  
    default:  
        return super.onOptionsItemSelected(item);  
    }  
}  

```
##  fragment中添加OptionsMenu 
在fragment中创建OptionsMenu时应该注意:
覆盖` onCreateOptionsMenu(Menu) `和` onOptionsItemSelected(MenuItem item) `
在` onCreate中添加setHasOptionsMenu(true) `;通知FragmentManager
##  通过Action Bar图标进行导航 

启用ActionBar图标导航的功能，可以允许用户根据当前应用的位置来在不同界面之间切换。比如，A界面展示了一个列表，点击某一项之后进入了B界面，这时B界面就应该启用ActionBar图标导航功能，这样就可以回到A界面。
我们可以通过调用**setDisplayHomeAsUpEnabled()**方法来启用ActionBar图标导航功能且此时的itemId是**android.R.id.home**，比如：
![](/data/dokuwiki/android/pasted/20150528-082229.png)
```

@Override  
protected void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  
    setTitle("天气");  
    setContentView(R.layout.activity_main);  
    ActionBar actionBar = getActionBar();  
    actionBar.setDisplayHomeAsUpEnabled(true);  
}  
@Override  
public boolean onOptionsItemSelected(MenuItem item) {  
    switch (item.getItemId()) {  
    case android.R.id.home:  
        finish();  
        return true;  
    ……  
    }  
}  

```
现在看上去，ActionBar导航和Back键的功能貌似是一样的。没错，如果我们只是简单地finish了一下，ActionBar导航和Back键的功能是完全一样的，但ActionBar导航的设计初衷并不是这样的，它和Back键的功能还是有一些区别的，举个例子吧。
![](/data/dokuwiki/android/pasted/20150528-082411.png)
上图中的Conversation List是收件箱的主界面，现在我们点击第一封邮件会进入到Conversation1 details界面，然后点击下一封邮件会进入到Conversation 2 details界面，再点击下一封邮箱会进入到Conversation3 details界面。好的，这个时候如果我们按下Back键，应该会回到Conversation 2 details界面，再按一次Back键应该回到Conversation1 details界面，再按一次Back键才会回到Conversation List。而ActionBar导航则不应该表现出这种行为，无论我们当前在哪一个Conversation details界面，点击一下导航按钮都应该回到Conversation List界面才对。
这就是ActionBar导航和Back键在设计上的区别，那么该怎样才能实现这样的功能呢？其实并不复杂，实现标准的ActionBar导航功能只需三步走。
第一步我们已经实现了，就是调用setDisplayHomeAsUpEnabled()方法，并传入true。
第二步需要在AndroidManifest.xml中配置父Activity，如下所示：
```

<activity  
    android:name="com.example.actionbartest.MainActivity"  
    android:logo="@drawable/weather" >  
    <meta-data  
        android:name="android.support.PARENT_ACTIVITY"  
        android:value="com.example.actionbartest.LaunchActivity" />  
</activity>  

```
可以看到，这里通过meta-data标签**指定了MainActivity的父Activity是LaunchActivity**，**在Android 4.1版本之后，也可以直接使用` android:parentActivityName `这个属性来进行指定**，如下所示：
```

<activity  
    android:name="com.example.actionbartest.MainActivity"  
    android:logo="@drawable/weather"  
    android:parentActivityName="com.example.actionbartest.LaunchActivity" >  
</activity>  

```
第三步则需要对android.R.id.home这个事件进行一些特殊处理，如下所示：
```

@Override  
public boolean onOptionsItemSelected(MenuItem item) {  
    switch (item.getItemId()) {  
    case android.R.id.home:  
        Intent upIntent = NavUtils.getParentActivityIntent(this);  
        if (NavUtils.shouldUpRecreateTask(this, upIntent)) {  
            TaskStackBuilder.create(this)  
                    .addNextIntentWithParentStack(upIntent)  
                    .startActivities();  
        } else {  
            upIntent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);  
            NavUtils.navigateUpTo(this, upIntent);  
        }  
        return true;  
        ......  
    }  
}  

```
其中，调用NavUtils.getParentActivityIntent()方法可以获取到跳转至父Activity的Intent，然后如果父Activity和当前Activity是在同一个Task中的，则直接调用navigateUpTo()方法进行跳转，如果不是在同一个Task中的，则需要借助TaskStackBuilder来创建一个新的Task。
这样，就按照标准的规范成功实现ActionBar导航的功能了。
##  添加Action View 

ActionView是一种可以在ActionBar中替换Action按钮的控件，它可以允许用户在不切换界面的情况下通过ActionBar完成一些较为丰富的操作。比如说，你需要完成一个搜索功能，就可以将SeachView这个控件添加到ActionBar中。
为了声明一个ActionView，我们可以在menu资源中通过actionViewClass属性来指定一个控件，例如可以使用如下方式添加SearchView：
```

<menu xmlns:android="http://schemas.android.com/apk/res/android" >  
  
    <item  
        android:id="@+id/action_search"  
        android:icon="@drawable/ic_action_search"  
        android:actionViewClass="android.widget.SearchView"  
        android:showAsAction="ifRoom|collapseActionView"  
        android:title="@string/action_search" />  
    ......  
  
</menu>  

```
**注意在showAsAction属性中我们还声明了一个collapseActionView，这个值表示该控件可以被合并成一个Action按钮。**
![](/data/dokuwiki/android/pasted/20150528-082759.png)
![](/data/dokuwiki/android/pasted/20150528-082804.png)
可以看到，这时SearchView就会展开占满整个ActionBar，而其它的Action按钮由于将showAsAction属性设置成了ifRoom，此时都会隐藏到overflow当中。
如果你还希望在代码中对SearchView的属性进行配置（比如添加监听事件等），完全没有问题，只需要在**onCreateOptionsMenu()**方法中获取该ActionView的实例就可以了，代码如下所示：
```

@Override  
public boolean onCreateOptionsMenu(Menu menu) {  
    MenuInflater inflater = getMenuInflater();  
    inflater.inflate(R.menu.main, menu);  
    MenuItem searchItem = menu.findItem(R.id.action_search);  
    SearchView searchView = (SearchView) searchItem.getActionView();  
    // 配置SearchView的属性  
    ......  
    return super.onCreateOptionsMenu(menu);  
}  

```
##  Overflow按钮不显示的情况 
，overflow按钮的显示情况和手机的硬件情况是有关系的，如果手机没有物理Menu键的话，overflow按钮就可以显示，如果有物理Menu键的话，overflow按钮就不会显示出来。
实际上，在ViewConfiguration这个类中有一个叫做sHasPermanentMenuKey的静态变量，系统就是根据这个变量的值来判断手机有没有物理Menu键的。当然这是一个内部变量，我们无法直接访问它，但是可以通过反射的方式修改它的值，让它永远为false就可以了，代码如下所示：
```

@Override  
protected void onCreate(Bundle savedInstanceState) {  
    ......  
    setOverflowShowingAlways();  
}  
  
private void setOverflowShowingAlways() {  
    try {  
        ViewConfiguration config = ViewConfiguration.get(this);  
        Field menuKeyField = ViewConfiguration.class.getDeclaredField("sHasPermanentMenuKey");  
        menuKeyField.setAccessible(true);  
        menuKeyField.setBoolean(config, false);  
    } catch (Exception e) {  
        e.printStackTrace();  
    }  
}  

```
##  让Overflow中的选项显示图标 
略
##  添加Action Provider 
和Action View有点类似，Action Provider也可以将一个Action按钮替换成一个自定义的布局。但不同的是，Action Provider能够完全控制事件的所有行为，并且还可以在点击的时候显示子菜单。
为了添加一个Action Provider，我们需要在<item>标签中指定一个actionViewClass属性，在里面填入Action Provider的完整类名。我们可以通过**继承ActionProvider**类的方式来创建一个自己的Action Provider，同时，Android也提供好了几个内置的Action Provider，比如说ShareActionProvider。

由于每个Action Provider都可以自由地控制事件响应，所以它们不需要在onOptionsItemSelected()方法中再去监听点击事件**，而是应该在onPerformDefaultAction()方法中去执行相应的逻辑。**
那么我们就先来看一下ShareActionProvider的简单用法吧，编辑menu资源文件，在里面加入ShareActionProvider的声明，如下所示：
```

<menu xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    tools:context="com.example.actionbartest.MainActivity" >  
  
    <item  
        android:id="@+id/action_share"  
        android:actionProviderClass="android.widget.ShareActionProvider"  
        android:showAsAction="ifRoom"  
        android:title="@string/action_share" />  
    ......  
  
</menu>  

```
<note tip>注意，ShareActionProvider会自己处理它的显示和事件，但我们仍然要记得给它添加一个title，以防止它会在overflow当中出现。</note>
接着剩下的事情就是通过Intent来定义出你想分享哪些东西了，我们只需要在onCreateOptionsMenu()中调用MenuItem的getActionProvider()方法来得到该ShareActionProvider对象，再通过setShareIntent()方法去选择构建出什么样的一个Intent就可以了。代码如下所示：
```

@Override  
public boolean onCreateOptionsMenu(Menu menu) {  
    MenuInflater inflater = getMenuInflater();  
    inflater.inflate(R.menu.main, menu);  
    MenuItem shareItem = menu.findItem(R.id.action_share);  
    ShareActionProvider provider = (ShareActionProvider) shareItem.getActionProvider();  
    provider.setShareIntent(getDefaultIntent());  
    ......  
    return super.onCreateOptionsMenu(menu);  
}  
  
private Intent getDefaultIntent() {  
    Intent intent = new Intent(Intent.ACTION_SEND);  
    intent.setType("image/*");  
    return intent;  
}  

```
![](/data/dokuwiki/android/pasted/20150528-084040.png)
细心的你一定观察到了，这个ShareActionProvider点击之后是可以展开的，有点类似于overflow的效果，这就是Action Provider的子菜单。除了使用ShareActionProvider之外，我们也可以自定义一个Action Provider，比如说如果想要建立一个拥有两项子菜单的Action Provider，就可以这样写：
```

public class MyActionProvider extends ActionProvider {  
      
    public MyActionProvider(Context context) {  
        super(context);  
    }  
  
    @Override  
    public View onCreateActionView() {  
        return null;  
    }  
  
    @Override  
    public void onPrepareSubMenu(SubMenu subMenu) {  
        subMenu.clear();  
        subMenu.add("sub item 1").setIcon(R.drawable.ic_launcher)  
                .setOnMenuItemClickListener(new OnMenuItemClickListener() {  
                    @Override  
                    public boolean onMenuItemClick(MenuItem item) {  
                        return true;  
                    }  
                });  
        subMenu.add("sub item 2").setIcon(R.drawable.ic_launcher)  
                .setOnMenuItemClickListener(new OnMenuItemClickListener() {  
                    @Override  
                    public boolean onMenuItemClick(MenuItem item) {  
                        return false;  
                    }  
                });  
    }  
  
    @Override  
    public boolean hasSubMenu() {  
        return true;  
    }  
  
}  

```
这里我们新建了一个MyActionProvider继承自ActionProvider，为了表示这个Action Provider是有子菜单的，` 需要重写hasSubMenu()方法并返回true `，然后在onPrepareSubMenu通过调用SubMenu的add()方法添加子菜单。
接着修改menu资源，在里面加入MyActionProvider的声明：
```

<menu xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:tools="http://schemas.android.com/tools"  
    tools:context="com.example.actionbartest.MainActivity" >  
  
    <item  
        android:id="@+id/action_share"  
        android:actionProviderClass="com.example.actionbartest.MyActionProvider"  
        android:icon="@drawable/ic_launcher"  
        android:showAsAction="ifRoom"  
        android:title="@string/action_share" />  
    ......  
  
</menu> 

```
![](/data/dokuwiki/android/pasted/20150528-084140.png)
##  添加导航Tabs 
Tabs的应用可以算是非常广泛了，它可以使得用户非常轻松地在你的应用程序中切换不同的视图。而Android官方更加推荐使用ActionBar中提供的Tabs功能，因为它更加的智能，可以自动适配各种屏幕的大小。比如说，在平板上屏幕的空间非常充足，Tabs会和Action按钮在同一行显示，如下图所示：
![](/data/dokuwiki/android/pasted/20150528-084215.png)![](/data/dokuwiki/android/pasted/20150528-084220.png)
下面我们就来看一下如何使用ActionBar提供的Tab功能，大致可以分为以下几步：

  * 1. 实现**ActionBar.TabListener**接口，这个接口提供了Tab事件的各种回调，比如当用户点击了一个Tab时，你就可以进行切换Tab的操作。
  * 2.为每一个你想添加的Tab创建一个**ActionBar.Tab**的实例，并且调用**setTabListener()方法来设置ActionBar.TabListener**。除此之外，还需要调用**setText()方法来给当前Tab设置标题**。
  * 3.最后调用ActionBar的**addTab()**方法将创建好的Tab添加到ActionBar中。

看起来并不复杂，总共就只有三步，那么我们现在就来尝试一下吧。首先第一步需要创建一个实现ActionBar.TabListener接口的类，代码如下所示：
```

public class TabListener<T extends Fragment> implements ActionBar.TabListener {  
    private Fragment mFragment;  
    private final Activity mActivity;  
    private final String mTag;  
    private final Class<T> mClass;  
  
    public TabListener(Activity activity, String tag, Class<T> clz) {  
        mActivity = activity;  
        mTag = tag;  
        mClass = clz;  
    }  
  
    public void onTabSelected(Tab tab, FragmentTransaction ft) {  
        if (mFragment == null) {  
            mFragment = Fragment.instantiate(mActivity, mClass.getName());  
            ft.add(android.R.id.content, mFragment, mTag);  
        } else {  
            ft.attach(mFragment);  
        }  
    }  
  
    public void onTabUnselected(Tab tab, FragmentTransaction ft) {  
        if (mFragment != null) {  
            ft.detach(mFragment);  
        }  
    }  
  
    public void onTabReselected(Tab tab, FragmentTransaction ft) {  
    }  
}  

```
当Tab被选中的时候会调用onTabSelected()方法，在这里我们先判断mFragment是否为空，如果为空的话就创建Fragment的实例并调用FragmentTransaction的add()方法，如果不会空的话就调用FragmentTransaction的attach()方法。
而当Tab没有被选中的时候，则调用FragmentTransaction的detach()方法，将UI资源释放掉。

当Tab被重新选中的时候会调用onTabReselected()方法，如果没有特殊需求的话，通常是不需要进行处理的。

接下来第二步要给每一个Tab创建一个ActionBar.Tab的实例，在此之前要先准备好每个Tab页对应的Fragment。比如说这里我们想创建两个Tab页，Artist和Album，那就要先准备好这两个Tab页对应的Fragment。首先新建ArtistFragment，代码如下所示：
```

public class ArtistFragment extends Fragment {  
  
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        TextView textView = new TextView(getActivity());  
        textView.setText("Artist Fragment");  
        textView.setGravity(Gravity.CENTER_HORIZONTAL);  
        LinearLayout layout = new LinearLayout(getActivity());  
        LayoutParams params = new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.WRAP_CONTENT);  
        layout.addView(textView, params);  
        return layout;  
    }  
  
}  

```
没有什么实质性的代码，只是在TextView中显示了Artist Fragment这个字符串。
然后如法炮制，新建AlbumFragment，代码如下所示：
```

public class AlbumFragment extends Fragment {  
      
    @Override  
    public View onCreateView(LayoutInflater inflater, ViewGroup container,  
            Bundle savedInstanceState) {  
        TextView textView = new TextView(getActivity());  
        textView.setText("Album Fragment");  
        textView.setGravity(Gravity.CENTER_HORIZONTAL);  
        LinearLayout layout = new LinearLayout(getActivity());  
        LayoutParams params = new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.WRAP_CONTENT);  
        layout.addView(textView, params);  
        return layout;  
    }  
  
}  

```
Fragment都准备好了之后，接下来就可以开始创建Tab实例了，创建好了之后则再调用addTab()方法添加到ActionBar当中，这两步通常都是在Activity的onCreate()方法中执行的，代码如下：
```

@Override  
protected void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  
    setTitle("天气");  
    ActionBar actionBar = getActionBar();  
    actionBar.setDisplayHomeAsUpEnabled(true);  
    setOverflowShowingAlways();  
    actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);  
    Tab tab = actionBar  
            .newTab()  
            .setText(R.string.artist)  
            .setTabListener(  
                    new TabListener<ArtistFragment>(this, "artist",  
                            ArtistFragment.class));  
    actionBar.addTab(tab);  
    tab = actionBar  
            .newTab()  
            .setText(R.string.album)  
            .setTabListener(  
                    new TabListener<AlbumFragment>(this, "album",  
                            AlbumFragment.class));  
    actionBar.addTab(tab);  
}  

```
![](/data/dokuwiki/android/pasted/20150528-084757.png)
##  自定义ActionBar样式 
###  1. 使用主题 

Android中有两个最基本的Activity主题可以用于指定ActionBar的颜色，分别是：
  * Theme.Holo，这是一个深色系的主题。
  * Theme.Holo.Light，这是一个浅色系的主题。
你可以将这些主题应用到你的整个应用程序，也可以只应用于某个Activity。通过在AndroidManifest.xml文件中给<application>或<activity>标签指定android:theme属性就可以实现了。比如：
```

<application android:theme="@android:style/Theme.Holo.Light" ... /> 

```
如果你只想让ActionBar使用深色系的主题，而Activity的内容部分仍然使用浅色系的主题，可以通过声明Theme.Holo.Light.DarkActionBar这个主题来实现，

虽说ActionBar给用户提供了一种全局统一的界面风格和操作方式，但这并不意味着所有应用程序的ActionBar都必须要长得一模一样。如果你需要修改ActionBar的样式来更加好地适配你的应用，可以非常简单地通过Android样式和主题来实现。

其实Android内置的几个Activity主题中就已经包含了"dark"或"light"这样的ActionBar样式了，同时你也可以继承这些主题，然后进行更深一步的定制。
###  2. 自定义背景 
如果想要修改ActionBar的背景，我们可以通过创建一个自定义主题并重写actionBarStyle属性来实现。这个属性可以指向另外一个样式，然后我们在这个样式中重写background这个属性就可以指定一个drawable资源或颜色，从而实现自定义背景的功能。
编辑styles.xml文件，在里面加入一个自定义的主题，如下所示：
```

<resources>  
  
    <style name="CustomActionBarTheme" parent="@android:style/Theme.Holo.Light">  
        <item name="android:actionBarStyle">@style/MyActionBar</item>  
    </style>  
  
    <style name="MyActionBar" parent="@android:style/Widget.Holo.Light.ActionBar">  
        <item name="android:background">#f4842d</item>  
    </style>  
  
</resources>  

```
![](/data/dokuwiki/android/pasted/20150528-085255.png)
这样我们就成功修改ActionBar的背景色了。不过现在看上去还有点怪怪的，因为只是ActionBar的背景色改变了，Tabs的背景色还是原来的样子，这样就感觉不太协调。那么下面我们马上就来修改一下Tabs的背景色，编辑styles.xml文件，如下所示：
```

<resources>  
  
    <style name="CustomActionBarTheme" parent="@android:style/Theme.Holo.Light">  
        <item name="android:actionBarStyle">@style/MyActionBar</item>  
    </style>  
  
    <style name="MyActionBar" parent="@android:style/Widget.Holo.Light.ActionBar">  
        <item name="android:background">#f4842d</item>  
        <item name="android:backgroundStacked">#d27026</item>  
    </style>  
  
</resources>  

```
可以看到，这里又重写了**backgroundStacked**属性，这个属性就是用于指定Tabs背景色的。那么再次重新运行程序，效果如下图所示：
![](/data/dokuwiki/android/pasted/20150528-085440.png)
###  3. 自定义文字颜色 

现在整个ActionBar的颜色是属于偏暗系的，而ActionBar中文字的颜色又偏偏是黑色的，所以看起来并不舒服，那么接下来我们就学习一下如果自定义文字颜色，将文字颜色改成白色。
修改styles.xml文件，如下所示：
```

<resources>  
  
    ......  
  
    <style name="MyActionBar" parent="@android:style/Widget.Holo.Light.ActionBar">  
        ......  
        <item name="android:titleTextStyle">@style/MyActionBarTitleText</item>  
    </style>  
  
    <style name="MyActionBarTitleText" parent="@android:style/TextAppearance.Holo.Widget.ActionBar.Title">  
        <item name="android:textColor">#fff</item>  
    </style>  
  
</resources>  

```
OK，ActionBar标题文字的颜色已经成功改成白色了，那Tab标题的文字又该怎么修改呢？继续编辑styles.xml文件，如下所示：
```

<resources>  
  
    <style name="CustomActionBarTheme" parent="@android:style/Theme.Holo.Light">  
        <item name="android:actionBarStyle">@style/MyActionBar</item>  
        <item name="android:actionBarTabTextStyle">@style/MyActionBarTabText</item>  
    </style>  
      
    <style name="MyActionBarTabText"  
           parent="@android:style/Widget.Holo.ActionBar.TabText">  
        <item name="android:textColor">#fff</item>  
    </style>  
  
</resources>  

```
![](/data/dokuwiki/android/pasted/20150528-085635.png)
###  4. 自定义Tab Indicator 
为了可以明确分辨出我们当前选中的是哪一个Tab项，通常情况下都会在选中Tab的下面加上一条横线作为标识，这被称作Tab Indicator。那么上图中的Tab Indicator是蓝色的，明显和整体风格不相符，所以我们接下来就学习一下如何自定义Tab Indicator。
首先我们需要重写**actionBarTabStyle**这个属性，然后将它指向一个新建的Tab样式，然后**重写background**这个属性即可。需要注意的是，background必须要指定一个state-list drawable文件，这样在各种不同状态下才能显示出不同的效果。
那么在开始之前，首先我们需要准备四张图片，分别用于表示Tab的四种状态，如下所示：
![](/data/dokuwiki/android/pasted/20150528-085757.png)
这四张图片分别表示Tab选中未按下，选中且按下，未选中未按下，未选中且按下这四种状态，那么接着新建**res/drawable/actionbar_tab_indicator.xml**文件，代码如下所示：
```

<?xml version="1.0" encoding="utf-8"?>  
<selector xmlns:android="http://schemas.android.com/apk/res/android">  
  
    <item  android:state_selected="false"  
          android:state_pressed="false"  
          android:drawable="@drawable/tab_unselected" />  
    <item android:state_selected="true"  
          android:state_pressed="false"  
          android:drawable="@drawable/tab_selected" />  
    <item android:state_selected="false"  
          android:state_pressed="true"  
          android:drawable="@drawable/tab_unselected_pressed" />  
    <item android:state_selected="true"  
          android:state_pressed="true"  
          android:drawable="@drawable/tab_selected_pressed" />  
  
</selector>  

```
接着修改style.xml文件，代码如下所示
```

<resources>  
  
    <style name="CustomActionBarTheme" parent="@android:style/Theme.Holo.Light">  
        ......  
        <item name="android:actionBarTabStyle">@style/MyActionBarTabs</item>  
    </style>  
      
    <style name="MyActionBarTabs" parent="@android:style/Widget.Holo.ActionBar.TabView">  
        <item name="android:background">@drawable/actionbar_tab_indicator</item>  
    </style>  
  
</resources>  

```
![](/data/dokuwiki/android/pasted/20150528-085943.png)
可以看到，Tab Indicator的颜色已经变成了白色，这样看上去就协调得多了。
除此之外，Action Bar还有许许多多的属性可以进行自定义，这里我们无法一一涵盖到本篇文章中，更多的自定义属性请参考官方文档进行学习。
#  对android中ActionBar中setDisplayHomeAsUpEnabled和setHomeButtonEnabled和setDisplayShowHomeEnabled方法的理解 
原文:http://blog.csdn.net/lovexieyuan520/article/details/9974929
setHomeButtonEnabled该方法的作用：决定左上角的图标是否可以点击。没有向左的小图标。 true 图标可以点击  false 不可以点击。
actionBar.setDisplayHomeAsUpEnabled(true)    / / 给左上角图标的左边加上一个返回的图标 。对应ActionBar.DISPLAY_HOME_AS_UP
actionBar.setDisplayShowHomeEnabled(true)   / /使左上角图标是否显示，如果设成false，则没有程序图标，仅仅就个标题，否则，显示应用程序图标，` 对应id为android.R.id.home `，对应ActionBar.DISPLAY_SHOW_HOME
actionBar.setDisplayShowCustomEnabled(true)  / / 使自定义的普通View能在title栏显示，即` actionBar.setCustomView能起作用 `，对应ActionBar.DISPLAY_SHOW_CUSTOM
actionBar.setDisplayShowTitleEnabled(true)   / / 对应ActionBar.DISPLAY_SHOW_TITLE。
其中setHomeButtonEnabled和setDisplayShowHomeEnabled共同起作用，如果setHomeButtonEnabled设成false，即使setDisplayShowHomeEnabled设成true，图标也不能点击


而如果想要对home图标进行点击时间处理，则需要在为这个icon“使能”：
setHomeButtonEnabled=true；
如果还想要一个回退箭头的话，再加上一句setDisplayHomeAsUpEnabled(true);

此home图标的id便是androi.R.id.home;
#  Android ActionBar隐藏修改图标和标题 
原文：http://blog.csdn.net/jdsjlzx/article/details/41353029
有时候在一些子页面或者内容页面，不需要显示ActionBar的标题栏图标。可用如下方式进行设置。首先获取到ActionBar对象
ActionBar actionBar=getActionBar();
使用android:logo属性。不像方方正正的icon，logo的图像不会有任何宽度限制。logo图像典型的给你的APP提供品牌。当你有Logo的时候，你可以隐藏label。默认的，ActionBar使用Activity的android:icon属性，还有一致的android:label属性。
  * 隐藏Label标签：actionBar.setDisplayShowTitleEnabled(false);
  * 隐藏logo和icon：actionBar.setDisplayShowHomeEnabled(false);
设置标题，一个主标题，一个子标题
actionBar.setSubtitle(“Inbox”); 
actionBar.setTitle(“Label:important”);

actionBar.hide()   //  影藏标题栏
actionBar.show()  // 显示标题栏
#  ActionBar 背景色修改 
重点是修改：values/styles.xml中的样式。

添加如下代码：
```

<!-- <style name="AppTheme" parent="android:Theme.Light" /> -->
    <style name="my_theme" parent="android:Theme.Holo.Light">
        <item name="android:actionMenuTextColor">#FF0000</item>
        <item name="android:actionBarStyle">@style/my_actionbar_style</item>
    </style>
 
    <style name="my_actionbar_style" parent="@android:style/Widget.Holo.Light.ActionBar">
        <item name="android:background">#FFFF00</item>
    </style>

```

另外还可以参考这篇系列文章：
http://blog.csdn.net/flowingflying/article/details/14001855