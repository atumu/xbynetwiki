title: android官方文档学习之fragment 

#  android官方文档学习之Fragment 
一个fragment(碎片)在一个活动中代表一个行为或用户界面的一部分。 你可以在一个单一的活动中组合使用多个碎片以建立一个多窗格的UI，并且可以在多个活动中重用一个碎片。你可以认为是一个拥有独立生命周期、能够独立接受输入事件、并且可以在活动运行时添加或移除的碎片作为一个活动的模块化部分.
##  创建一个Fragment 
Fragment类的代码很像Activity.它还有和activity相似的回调方法,比如onCreate(), onStart(), onPause(), 和 onStop().实际上,如果你在使用Fragment来转换一个现成的应用,你可能只是简单的从你的activity回调方法中移动代码到fragment相应的回调方法中.
一般的,你至少应该实现下面的生命周期方法:
` onCreate() `
` onCreateView() `:在fragment第一次绘制他的用户界面的时候系统会调用这个方法.如果你想为你的fragment绘制界面,你必须从这个方法中返回一个View,这个View是你fragment布局的基础.如果这个Fragment不提供UI,你可以返回空.
` onPause() `:

##  Fragment子类 

` DialogFragment `:**显示一个浮动的对话框**.使用这个类来创建一个对话框是和使用对话Helper方法在Activity类中创建对话框都是很好的方法,因为**你可以把Fragment对话框包含在activity管理的Fragment返回栈中,允许用户返回到关闭的Fragment中.**
` ListFragment `:展示一列被adapter(比如` SimpleCursorAdapter `)管理的项,和ListActivity很相似.它提供了一些管理一个列表视图的方法,比如处理点击事件的` onListItemClick() `方法.
` PreferenceFragment `：用一个列表来显示一组偏好设置对象,类似于PreferenceActivity. 在创建设置型的activity时会用到.
##  Adding a user interface 
```

public static class ExampleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.example_fragment, container, false);
    }
}

```
传递给onCreateView()的container参数是fragment要插入的activity的父ViewGroup(来自对应的activity布局)。
inflate()方法接收三个参数:
  * 你想要添加的layout的资源ID.
  * 将作为填充布局的父容器的ViewGroup.传递容器参数是非常重要的,只用这样才能使系统应用布局参数到填充视图的根视图,从而被它的父视图所确定.
  * 一个boolean类型的参数,用于在填充时指明填充的布局是否应该附加在ViewGroup(第二个参数)上.(如果系统已经插入这个填充布局到容器了就返回false,如果将要在最终布局中创建一个多余的viewgroup,那就返回true)**一般返回false.**
##  给Activity添加一个Fragment 
```

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <fragment android:name="com.example.news.ArticleListFragment"
            android:id="@+id/list"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
    <fragment android:name="com.example.news.ArticleReaderFragment"
            android:id="@+id/viewer"
            android:layout_weight="2"
            android:layout_width="0dp"
            android:layout_height="match_parent" />
</LinearLayout>

```
当系统生成这个activity布局时，会把布局中每个特定的片段实例化，然后依次调用onCreateView（）方法以便检索每个片段布局。**系统直接插入片段返回的视图来代替<fragment>元素。**
` 在任何你Activity运行的时候,你都可以动态地把fragment添加到Activity的视图中. `你只需要**指定一个用于盛放Fragment的ViewGroup**.为了让fragment可以被管理(比如添加,删除,替换fragment),你必须使用来自` FragmentTransaction `的API.你可以像下面这样在Activity中获取一个FragmentTransaction的实例:
```

FragmentManager fragmentManager = getFragmentManager()
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
ExampleFragment fragment = new ExampleFragment();
fragmentTransaction.add(R.id.fragment_container, fragment);//必须制定用于盛放Fragment的ViewGroup容器ID
fragmentTransaction.commit();

```
add()方法中的第一个参数是Fragment**所要放置的目标ViewGroup**,通过资源ID指定,第二个参数是要添加的Fragment.只要你使用FragmentTransaction做了修改,你必须调用` commit() `方法来使修改生效.
##  添加一个没有UI的Fragment 
为了添加一个没有UI的Fragment.需要使用` add(Fragment, String) ` 方法,其中,你需要为Fragment提供一个**字符串的标志**而不是一个视图ID.这样增加的Fragment,由于没有涉及到Activity的视图,**所以不会调用onCreateView()方法**.所以你不需要实现这个方法.
为Fragment提供一个字符串标志不一定只局限于没有UI的Fragment,你也可以为有UI的Fragment指定一个字符串标志,**但是如果这个Fragment真的没有UI,那这个字符串标志是确定它的唯一标志**.如果你想在后面从Activity中获取到这个fragment,你需要使用` findFragmentByTag() `方法.
##  管理fragment 
为了管理你Activity中的fragment,你需要使用` FragmentManager `.你可以通过你Activity中的` getFragmentManager() `来获取它.
使用FragmentManager你可以:
  * 使用` findFragmentById() `(提供UI的Fragment)或者` findFragmentByTag()( `没有提供UI的Fragment) 获取你Activity存在的Fragment,
  * 使用` popBackStack() `把Fragment从返回栈中弹出(**模拟用户的返回命令**).
  * 使用` addOnBackStackChangedListener() `方法为返回栈的变化注册监听器.
你也可以使用FragmentManager来打开FragmentTransaction,FragmentTransaction允许你执行添加,删除Fragment的事务.
##  执行Fragment事务. 
在你的Activity中使用Fragment的最大好处就是可以针对用户的操作,进行对Fragment的添加,移除,替换等等其他操作.你提交给Activity的每个变化称为一个事务,这些事务你可以使用FragmenTransaction的API来实现.你也可以在Activity管理的返回栈中保存每个事务,使用户可以在Fragmen的变化后返回之前的状态(类似于在Activity跳转后的返回).
每个事务是一系列你想要同时执行的Fragmen的变化.你可以使用像add(),remove(),replace()这样的方法来为一个事务设定你想要执行的操作.为了使Activity的事务生效,**你必须执行commit()方法.**
在你调用commit()方法的之前,为了添加这个事务到一个Fragmen事务的返回栈,你可能想要调用` addToBackStack() `方法.这个返回栈被Activity管理,允许用户通过按下返回按键返回之前的Fragmen状态.
这里展示了怎么使用一个Fragmen替换另一个,然后在返回栈中返回到之前的状态:
```

//创建一个新的Fragmen和事务
Fragment newFragment = new ExampleFragment();
FragmentTransaction transaction = getFragmentManager().beginTransaction();
 
// Replace whatever is in the fragment_container view with this fragment,
//使用这个Fragment替换在Fragmen容器中的Fragmet
// and add the transaction to the back stack
//添加这个事务到返回栈
transaction.replace(R.id.fragment_container, newFragment);
//addToBackStack(name)An optional name for this back stack state, or null.
transaction.addToBackStack(null);
 
// Commit the transaction
//提交这个事务.
transaction.commit();

```
在这个例子中,新的Fragmen**替换**了R.id.fragment_container ID指定的布局容器中当前存在的fragment(如果存在的话).通过调用addToBackStack()方法,` 替换事务 `(**注意只是这个替换事务而非fragment**)**被保存在了返回栈中**,这样` 用户可以回退这个事务 `,通过**按下返回键返回到以前的fragment**.
如果你在事务中添加了多个变化(比如另一个add()方法或者remove()方法),然后调用了addToBackStack()方法,那在你调用commit()方法之前的所有变化都会作为单独的事务被添加到返回栈中,返回键将会把他们全部回退.
<note tip>小贴士:对于每个fragment事务,你可以在提交之前通过调用` setTransition() `来应用一个fragment动画.</note>
调用commit()方法不能立即执行事务而是安排它运行在Activity的UI线程中("主"线程)---如果这线程可以这么做的话.
<note>注意:你只可以在Activity保存他状态之前(在用户离开这个Actvity的时候)使用commit()方法来提交一个事务.如果你在这个时间点之后提交,系统会抛出一个异常.这是因为如果Activity需要恢复,在提交之后的状态可能会丢失.对于允许丢失提交的情况,请使用commitAllowingStateLoss()方法.</note>
##  与Activity的通讯 
一个fragment实例还是和它所在的容器有直接的关系.特别的,fragment可以通过getActivity()方法来访问Activity实例并可以轻易的执行像在activity视图中查找View的任务.
View listView = getActivity().findViewById(R.id.list);
同样的,使用` findFragmentById()或findFragmentByTag() `通过从FragmentManager获取一个对这个Fragment的引用,你的Activity可以调用fragment中的方法
ExampleFragment fragment = (ExampleFragment) getFragmentManager().findFragmentById(R.id.example_fragment);
##  为Activity创建回调接口 
在一些情况下,**你可能需要一个Fragment和Activity共享事件**.一个好的方法是在` Fragment中定义一个回调接口 `然后` 让承载他的Activity实现它 `.当Activity通过接口接收到调用时,必要时他可以和视图中的其他Fragment共享信息.
举个例子,如果一个新的应用在一个Activity中有两个Fragment,一个显示一列文章标题(FragmentA),另一列显示文章内容(FragmentB),那么在一列被选中的时候,FragmentA必须告诉Actvity那一列被选中了,这样Actvity就可以告诉FragmentB显示哪一篇文章.在这种情况下,OnArticleSelectedListener 接口会在FragmentA中声明.
```

public static class FragmentA extends ListFragment {
    ...
    // Container Activity must implement this interface
    public interface OnArticleSelectedListener {
        public void onArticleSelected(Uri articleUri);
    }
    ...
}
 

```
然后承载Fragment的Activity实现OnArticleSelectedListener接口并重写onArticleSelected()方法来通知FragmentB响应FragmentA的事件.**为了保证这个Activity实现了这个接口**,FragmentA的` onAttach()方法 `(系统在添加Fragment到这个Activity的时候调用)**通过把Activity参数传递到onAttach()方法传递实例化一个OnArticleSelectedListener实例.**
```

public static class FragmentA extends ListFragment {
    OnArticleSelectedListener mListener;
    ...
    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        try {
            mListener = (OnArticleSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString() + " must implement OnArticleSelectedListener");
        }
    }
    ...
}



```
##  在Action Bar上添加项. 
你的Fragment可以通过实现` onCreateOptionsMenu() `来为Activity的Options Menu创建菜单项(结果就是形成ActionBar).为了让这个方法接收到调用,你必须` 在onCreate()方法中调用setHasOptionsMenu() `方法来表明这个Fragment允许在Options Menu中增加项(**否则,Fragment将不能接收onCreateOptionsMenu()的调用**).
你从Fragment 添加到 Options Menu的任何项**都是现存菜单项的附加项**.在一个菜单项选中的时候,Fragment也接收响应` onOptionsItemSelected() `方法的调用.

你也可以在你的Fragment视图中通过调用` registerForContextMenu() `方法来注册一个视图,从而提供一个上下文菜单.当用户打开上下文菜单时,Fragment会接收一个` onCreateContextMenu() `的调用,当用户选择一项的时候,Fragment接收一个` onContextItemSelected() `的调用.

<note tip>注意.即使你的Fragment在每个添加的菜单项接收了一个on-item-selected调用,在用户选择一个菜单项的时候,Activity是第一个接受各自调用的组件.如果Activity实现的on-item-selected调用没有处理选择项后的事件,那这个事件会传递到Fragment的回调中.这对 Options Menu 和上下文菜单都是适用的.</note>

##  处理Fragment的生命周期 
![](/data/dokuwiki/android/pasted/20150606-175625.png)
和Activity一样,在这个Activity所在的进程被杀死或者你需要在Activity重新创建的时候保存Fragment的状态,你可以用Bundle来做这个工作.你可以` 在Fragment执行onSaveInstanceState()方法 `的时候保存它的状态,然后` 在onCreate()或者onCreateView(),onActivityCreated()方法的时候恢复这些状态. `更多关于保存状态的内容参考Activity文档.
Activity和Fragment最大的不同是他们在返回栈中的存在形式.默认的,Activity在停止的时候,是放在一个被系统管理的返回栈中(这样用户可以使用back按钮返回,就像在Tasks and Back Stack一章中谈论的那样).**然而在一个移除Fragment的事务中,****只有在你通过调用addToBackStack()明确的指明这个Fragment不要被保存,这个Fragment才会被放在被宿主Activity管理的返回栈中.**
Fragment有一些额外的生命周期,用来处理和Activity的特殊交换,从而可以执行形如创建和销毁FragmentUI的事情.这些额外的回调方法有:
` onAttach() `:当Fragment和Activity链接起来的时候调用(**Activity实例在这里传送过来**).
` onCreateView() `：创建Fragment的视图层.
` onActivityCreated() `：当Activity的onCreate返回的时候执行.
` onDestroyView() `：当Fragment的试图层被移除的时候执行.
` onDetach() `：当Fragment和Activity分离的时候执行.

##  Example-例子 
为了把上面介绍的知识汇总,这里有个使用两个Fragment组成两个视图布局的例子.下面的activity包含两个Fragment,一个用来显示Shakespeare话剧的标题,另一个用来显示选中话剧的简介.也演示了怎么根据屏幕的不同为这两个Fragment提供不同的配置.注意:完整代码在Api14中的ApiDemos的FragmentLayout.java中.
```

public class FragmentLayout extends Activity {


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        setContentView(R.layout.fragment_layout);
    }

```
res/layout-land/fragment_layout.xml:横屏布局
```

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent" android:layout_height="match_parent">

    <fragment class="com.example.android.apis.app.FragmentLayout$TitlesFragment"
            android:id="@+id/titles" android:layout_weight="1"
            android:layout_width="0px" android:layout_height="match_parent" />

    <FrameLayout android:id="@+id/details" android:layout_weight="1"
            android:layout_width="0px" android:layout_height="match_parent"
            android:background="?android:attr/detailsElementBackground" />
    
</LinearLayout>

```
res/layout-land/fragment.xml：竖屏布局
```

    android:layout_width="match_parent" android:layout_height="match_parent">
    <fragment class="com.example.android.apis.app.FragmentLayout$TitlesFragment"
            android:id="@+id/titles"
            android:layout_width="match_parent" android:layout_height="match_parent" />
</FrameLayout>

```
正如你看到的那样,**注意在用户点击列表响的时候,有两个可能的行为:如果这两个视图存在,将在这个activity中创建并显示一个新的Fragment(把Fragment添加到FragmentLayout中);如果只有一个视图(竖屏),那会启动一个新的activity(Fragment在这个activity中显示).**
```

public static class TitlesFragment extends ListFragment {
        boolean mDualPane;
        int mCurCheckPosition = 0;

        @Override
        public void onActivityCreated(Bundle savedInstanceState) {
            super.onActivityCreated(savedInstanceState);

            // Populate list with our static array of titles.
            setListAdapter(new ArrayAdapter<String>(getActivity(),
                    android.R.layout.simple_list_item_activated_1, Shakespeare.TITLES));

            // Check to see if we have a frame in which to embed the details
            // fragment directly in the containing UI.
            View detailsFrame = getActivity().findViewById(R.id.details);
            mDualPane = detailsFrame != null && detailsFrame.getVisibility() == View.VISIBLE;

            if (savedInstanceState != null) {
                // Restore last state for checked position.
                mCurCheckPosition = savedInstanceState.getInt("curChoice", 0);
            }

            if (mDualPane) {
                // In dual-pane mode, the list view highlights the selected item.
                getListView().setChoiceMode(ListView.CHOICE_MODE_SINGLE);
                // Make sure our UI is in the correct state.
                showDetails(mCurCheckPosition);
            }
        }

        @Override
        public void onSaveInstanceState(Bundle outState) {
            super.onSaveInstanceState(outState);
            outState.putInt("curChoice", mCurCheckPosition);
        }

        @Override
        public void onListItemClick(ListView l, View v, int position, long id) {
            showDetails(position);
        }

        /**
         * Helper function to show the details of a selected item, either by
         * displaying a fragment in-place in the current UI, or starting a
         * whole new activity in which it is displayed.
         */
        void showDetails(int index) {
            mCurCheckPosition = index;

            if (mDualPane) {
                // We can display everything in-place with fragments, so update
                // the list to highlight the selected item and show the data.
                getListView().setItemChecked(index, true);

                // Check what fragment is currently shown, replace if needed.
                DetailsFragment details = (DetailsFragment)
                        getFragmentManager().findFragmentById(R.id.details);
                if (details == null || details.getShownIndex() != index) {
                    // Make new fragment to show this selection.
                    details = DetailsFragment.newInstance(index);

                    // Execute a transaction, replacing any existing fragment
                    // with this one inside the frame.
                    FragmentTransaction ft = getFragmentManager().beginTransaction();
                    ft.replace(R.id.details, details);
                    ft.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_FADE);
                    ft.commit();
                }

            } else {
                // Otherwise we need to launch a new activity to display
                // the dialog fragment with selected text.
                Intent intent = new Intent();
                intent.setClass(getActivity(), DetailsActivity.class);
                intent.putExtra("index", index);
                startActivity(intent);
            }
        }
    }

```
第二个Fragment,DetailsFragment显示了在TitleFragment中选中的话剧简介.
```

public static class DetailsFragment extends Fragment {
    /**
     * Create a new instance of DetailsFragment, initialized to
     * show the text at 'index'.
     */
    public static DetailsFragment newInstance(int index) {
        DetailsFragment f = new DetailsFragment();
 
        // Supply index input as an argument.
        Bundle args = new Bundle();
        args.putInt("index", index);
        f.setArguments(args);
 
        return f;
    }
 
    public int getShownIndex() {
        return getArguments().getInt("index", 0);
    }
 
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        if (container == null) {
            // We have different layouts, and in one of them this
            // fragment's containing frame doesn't exist.  The fragment
            // may still be created from its saved state, but there is
            // no reason to try to create its view hierarchy because it
            // won't be displayed.  Note this is not needed -- we could
            // just run the code below, where we would create and return
            // the view hierarchy; it would just never be used.
            return null;
        }
 
        ScrollView scroller = new ScrollView(getActivity());
        TextView text = new TextView(getActivity());
        int padding = (int)TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,
                4, getActivity().getResources().getDisplayMetrics());
        text.setPadding(padding, padding, padding, padding);
        scroller.addView(text);
        text.setText(Shakespeare.DIALOGUE[getShownIndex()]);
        return scroller;
    }
}

```
来自TitleFragment的调用,如果用户点击列表项的时候当前布局不包含R.id.details视图(DetailsFragment 所在的视图),那应用将会启动DetailsActivity 来显示选中项的内容简介.
```

public static class DetailsActivity extends Activity {
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
 
        if (getResources().getConfiguration().orientation
                == Configuration.ORIENTATION_LANDSCAPE) {
            // If the screen is now in landscape mode, we can show the
            // dialog in-line with the list so we don't need this activity.
            finish();
            return;
        }
 
        if (savedInstanceState == null) {
            // During initial setup, plug in the details fragment.
            DetailsFragment details = new DetailsFragment();
            details.setArguments(getIntent().getExtras());
            getFragmentManager().beginTransaction().add(android.R.id.content, details).commit();
        }
    }
}

```
附实例代码：![](/data/dokuwiki/android/fragmentlayout.zip|)
注意activity会在横屏的时候结束自己,这样主activity可以接管并显示DetailsFragment旁边的TitlesFragment.如果用户在竖屏的时候启动DetailsActivity,然后把设备转到横屏(将会重启当前的activity).
更多使用Fragment的例子(包括这个例子的全部代码),请参考API14 ApiDemo(可以在SDK例子那里下载).