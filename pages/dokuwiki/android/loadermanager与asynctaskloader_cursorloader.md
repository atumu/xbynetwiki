title: loadermanager与asynctaskloader_cursorloader 

#  android:LoaderManager与AsyncTaskLoader、CursorLoader 

#  概述 
Loaders，装载机，适用于Android3.0以及更高的版本，它提供了一套在UI的主线程中` 异步加载数据 `的框架。使用Loaders可以非常简单的在Activity或者Fragment中异步加载数据，一般适用于大量的数据查询，或者需要经常修改并及时展示的数据显示到UI上，这样可以避免查询数据的时候，造成UI主线程的卡顿。
Loaders有以下特点：
  * 可以适用于Activity和Fragment。
  * 可以提供异步的方式加载数据。
  * 监听数据源，当数据改变的时候，将新的数据发布到UI上。
  * Loaders使用Cursor加载数据，在更改Cursor的时候，会自动重新连接到最后配置的Cursor中读取数据，因此不需要重新查询数据。

在Android中使用Loaders机制，需要多个类和接口的配合，以下是它们大致的关系图，
![](/data/dokuwiki/android/pasted/20150516-150655.png)
##  接口说明： 

**LoaderManager**，装载机管理器。用于在Activity或者Fragment中管理一个或多个Loader实例。在Activity或者Fragment中，可以通过getLoaderManager()方法获取LoaderManager对象，它是一个单例模式。
**LoaderManager.LoaderCallbacks**
　　LoaderCallbacks是LoaderManager和Loader之间的回调接口。它是一个回调接口，所以我们需要实现其定义的三个方法：
**Loader**，一个抽象的类，用于执行异步加载数据，这个Loader对象可以监视数据源的改变和在内容改变后，以新数据的内容改变UI的展示。它是一个Loader的抽象接口，所有需要实现的Loader功能的类都需要实现这个接口，但是如果需要自己开发一个装载机的话，一般并不推荐继承Loader接口，而是继承它的子类AsyncTaskLoader，这是一个以AsyncTask框架执行的异步加载。
**SimpleCursorAdapter**
　在Android中，数据的展示都需要使用一个Adapter适配器，而Loader一般返回的就是一个Cursor的数据，可以使用BaseAdapter的一个子类SimpleCursorAdapter，它可以使用XML资源文件自定义一个布局在展示数据。它有两个构造函数，但是有一个构造函数在API Level11之后就不推荐使用。下面是构造函数的签名：
　　SimpleCursorAdapter(Context context,int layout,Cursor c,String[] from,int[] to,int flags).
　　**最后一个参数flags是一个标识，标识当数据改变调用onContentChanged()的时候，是否通知ContentProvider数据的改变，如果无需监听ContentProvider的改变，则可以传0。**` 对于SimpleCursorAdapter适配器的Cursor的改变，可以使用SimpleCursorAdapter.swapCursor(Cursor)方法，它会与旧的Cursor互换，并且返回旧的Cursor。 `
**CursorLoader由LoaderManager管理，用于异步从ContentProvider加载Cursor.**默认的CursorLoader实现是从ContentProvider加载Cursor.
Loader体系：android.content.Loader<AsyncTaskLoader<CursorLoader.
那么问题来了，如果我们想从数据库直接异步加载Cursor呢？那么最好的思路就是继承AsyncTaskLoader.


# 步骤 
步骤一：先自定义Loader
步骤二：在Activity或者Fragment中实现LoaderManager.LoadCallbacks接口。
```

 @Override
    public Loader<RSSModel> onCreateLoader(int id, Bundle args) {
        return new MyRssModelLoader(this, (long) nowId);
    }

    @Override
    public void onLoadFinished(Loader<RSSModel> loader, RSSModel data) {
        rssPageAdapter.chageModel(data);
        rssPageAdapter.notifyDataSetChanged();
    }
    @Override
    public void onLoaderReset(Loader<RSSModel> loader) {
      //Loader Reset时取消与旧数据的绑定。你也可以注释这句
	rssPageAdapter.chageModel("");
    }

```
步骤三：调用getSupportLoaderManager().initLoader()方法进行初始化Loader
```

 getSupportLoaderManager().initLoader(1, null, this);//注意这个方法的调用顺序。

```
步骤四，更新Loader:调用getSupportLoaderManager().restartLoader()方法重新加载Loader用于Loader的更新。很重要，或者调用getSupportLoaderManager().destroyLoader()然后再调用initLoader也行。
  * 方式一：
```

getSupportLoaderManager().restartLoader(1);

```
  * 方式二：
```

getSupportLoaderManager().destroyLoader(1);
getSupportLoaderManager().initLoader(1, null, this);

```

##  API文档中android.content.AsyncTaskLoader有个自定义AsyncTaskLoader的示例 
如下：
```

public static class AppListLoader extends AsyncTaskLoader<List<AppEntry>> {
    final InterestingConfigChanges mLastConfig = new InterestingConfigChanges();
    final PackageManager mPm;

    List<AppEntry> mApps;
    PackageIntentReceiver mPackageObserver;

    public AppListLoader(Context context) {
        super(context);

        // Retrieve the package manager for later use; note we don't
        // use 'context' directly but instead the save global application
        // context returned by getContext().
        mPm = getContext().getPackageManager();
    }

    /**
     * This is where the bulk of our work is done.  This function is
     * called in a background thread and should generate a new set of
     * data to be published by the loader.
     */
    @Override public List<AppEntry> loadInBackground() {
        // Retrieve all known applications.
        List<ApplicationInfo> apps = mPm.getInstalledApplications(
                PackageManager.GET_UNINSTALLED_PACKAGES |
                PackageManager.GET_DISABLED_COMPONENTS);
        if (apps == null) {
            apps = new ArrayList<ApplicationInfo>();
        }

        final Context context = getContext();

        // Create corresponding array of entries and load their labels.
        List<AppEntry> entries = new ArrayList<AppEntry>(apps.size());
        for (int i=0; i<apps.size(); i++) {
            AppEntry entry = new AppEntry(this, apps.get(i));
            entry.loadLabel(context);
            entries.add(entry);
        }

        // Sort the list.
        Collections.sort(entries, ALPHA_COMPARATOR);

        // Done!
        return entries;
    }

    /**
     * Called when there is new data to deliver to the client.  The
     * super class will take care of delivering it; the implementation
     * here just adds a little more logic.
     */
    @Override public void deliverResult(List<AppEntry> apps) {
        if (isReset()) {
            // An async query came in while the loader is stopped.  We
            // don't need the result.
            if (apps != null) {
                onReleaseResources(apps);
            }
        }
        List<AppEntry> oldApps = apps;
        mApps = apps;

        if (isStarted()) {
            // If the Loader is currently started, we can immediately
            // deliver its results.
            super.deliverResult(apps);
        }

        // At this point we can release the resources associated with
        // 'oldApps' if needed; now that the new result is delivered we
        // know that it is no longer in use.
        if (oldApps != null) {
            onReleaseResources(oldApps);
        }
    }

    /**
     * Handles a request to start the Loader.
     */
    @Override protected void onStartLoading() {
        if (mApps != null) {
            // If we currently have a result available, deliver it
            // immediately.
            deliverResult(mApps);
        }

        // Start watching for changes in the app data.
        if (mPackageObserver == null) {
            mPackageObserver = new PackageIntentReceiver(this);
        }

        // Has something interesting in the configuration changed since we
        // last built the app list?
        boolean configChange = mLastConfig.applyNewConfig(getContext().getResources());

        if (takeContentChanged() || mApps == null || configChange) {
            // If the data has changed since the last time it was loaded
            // or is not currently available, start a load.
            forceLoad();
        }
    }

    /**
     * Handles a request to stop the Loader.
     */
    @Override protected void onStopLoading() {
        // Attempt to cancel the current load task if possible.
        cancelLoad();
    }

    /**
     * Handles a request to cancel a load.
     */
    @Override public void onCanceled(List<AppEntry> apps) {
        super.onCanceled(apps);

        // At this point we can release the resources associated with 'apps'
        // if needed.
        onReleaseResources(apps);
    }

    /**
     * Handles a request to completely reset the Loader.
     */
    @Override protected void onReset() {
        super.onReset();

        // Ensure the loader is stopped
        onStopLoading();

        // At this point we can release the resources associated with 'apps'
        // if needed.
        if (mApps != null) {
            onReleaseResources(mApps);
            mApps = null;
        }

        // Stop monitoring for changes.
        if (mPackageObserver != null) {
            getContext().unregisterReceiver(mPackageObserver);
            mPackageObserver = null;
        }
    }

    /**
     * Helper function to take care of releasing resources associated
     * with an actively loaded data set.
     */
    protected void onReleaseResources(List<AppEntry> apps) {
        // For a simple List<> there is nothing to do.  For something
        // like a Cursor, we would close it here.
    }
}

```
##  关于SimpleCursorAdapter更新的问题： 

不能调用notifyDataSetChanged()，而是调用 
```
adapter.changeCursor(data);</code>

##  API文档中android.app.LoaderManager有个关于LoaderManager结合CursorLoader、ListFragment、SimpleCursorAdapter的例子 
，如下：
<code java>
public static class CursorLoaderListFragment extends ListFragment
        implements OnQueryTextListener, OnCloseListener,
        LoaderManager.LoaderCallbacks<Cursor> {
    SimpleCursorAdapter mAdapter;
    SearchView mSearchView;
    // If non-null, this is the current filter the user has provided.
    String mCurFilter;

    @Override public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        // Give some text to display if there is no data.  In a real
        // application this would come from a resource.
        setEmptyText("No phone numbers");

        // We have a menu item to show in action bar.
        setHasOptionsMenu(true);

        // Create an empty adapter we will use to display the loaded data.
        mAdapter = new SimpleCursorAdapter(getActivity(),
                android.R.layout.simple_list_item_2, null,
                new String[] { Contacts.DISPLAY_NAME, Contacts.CONTACT_STATUS },
                new int[] { android.R.id.text1, android.R.id.text2 }, 0);
        setListAdapter(mAdapter);

        // Start out with a progress indicator.
        setListShown(false);

        // Prepare the loader.  Either re-connect with an existing one,
        // or start a new one.
        getLoaderManager().initLoader(0, null, this);
    }

    public static class MySearchView extends SearchView {
        public MySearchView(Context context) {
            super(context);
        }

        // The normal SearchView doesn't clear its search text when
        // collapsed, so we will do this for it.
        @Override
        public void onActionViewCollapsed() {
            setQuery("", false);
            super.onActionViewCollapsed();
        }
    }

    @Override public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        // Place an action bar item for searching.
        MenuItem item = menu.add("Search");
        item.setIcon(android.R.drawable.ic_menu_search);
        item.setShowAsAction(MenuItem.SHOW_AS_ACTION_IF_ROOM
                | MenuItem.SHOW_AS_ACTION_COLLAPSE_ACTION_VIEW);
        mSearchView = new MySearchView(getActivity());
        mSearchView.setOnQueryTextListener(this);
        mSearchView.setOnCloseListener(this);
        mSearchView.setIconifiedByDefault(true);
        item.setActionView(mSearchView);
    }

    public boolean onQueryTextChange(String newText) {
        // Called when the action bar search text has changed.  Update
        // the search filter, and restart the loader to do a new query
        // with this filter.
        String newFilter = !TextUtils.isEmpty(newText) ? newText : null;
        // Don't do anything if the filter hasn't actually changed.
        // Prevents restarting the loader when restoring state.
        if (mCurFilter == null && newFilter == null) {
            return true;
        }
        if (mCurFilter != null && mCurFilter.equals(newFilter)) {
            return true;
        }
        mCurFilter = newFilter;
        getLoaderManager().restartLoader(0, null, this);
        return true;
    }

    @Override public boolean onQueryTextSubmit(String query) {
        // Don't care about this.
        return true;
    }

    @Override
    public boolean onClose() {
        if (!TextUtils.isEmpty(mSearchView.getQuery())) {
            mSearchView.setQuery(null, true);
        }
        return true;
    }

    @Override public void onListItemClick(ListView l, View v, int position, long id) {
        // Insert desired behavior here.
        Log.i("FragmentComplexList", "Item clicked: " + id);
    }

    // These are the Contacts rows that we will retrieve.
    static final String[] CONTACTS_SUMMARY_PROJECTION = new String[] {
        Contacts._ID,
        Contacts.DISPLAY_NAME,
        Contacts.CONTACT_STATUS,
        Contacts.CONTACT_PRESENCE,
        Contacts.PHOTO_ID,
        Contacts.LOOKUP_KEY,
    };

    public Loader<Cursor> onCreateLoader(int id, Bundle args) {
        // This is called when a new Loader needs to be created.  This
        // sample only has one Loader, so we don't care about the ID.
        // First, pick the base URI to use depending on whether we are
        // currently filtering.
        Uri baseUri;
        if (mCurFilter != null) {
            baseUri = Uri.withAppendedPath(Contacts.CONTENT_FILTER_URI,
                    Uri.encode(mCurFilter));
        } else {
            baseUri = Contacts.CONTENT_URI;
        }

        // Now create and return a CursorLoader that will take care of
        // creating a Cursor for the data being displayed.
        String select = "((" + Contacts.DISPLAY_NAME + " NOTNULL) AND ("
                + Contacts.HAS_PHONE_NUMBER + "=1) AND ("
                + Contacts.DISPLAY_NAME + " != '' ))";
        return new CursorLoader(getActivity(), baseUri,
                CONTACTS_SUMMARY_PROJECTION, select, null,
                Contacts.DISPLAY_NAME + " COLLATE LOCALIZED ASC");
    }

    public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
        // Swap the new cursor in.  (The framework will take care of closing the
        // old cursor once we return.)
        mAdapter.swapCursor(data);

        // The list should now be shown.
        if (isResumed()) {
            setListShown(true);
        } else {
            setListShownNoAnimation(true);
        }
    }

    public void onLoaderReset(Loader<Cursor> loader) {
        // This is called when the last Cursor provided to onLoadFinished()
        // above is about to be closed.  We need to make sure we are no
        // longer using it.
        mAdapter.swapCursor(null);
    }
}

```
#  自定义Loader从数据库直接异步加载Cursor或者其他数据。 
先定义一个抽象类，继承了AsyncTaskLoader
```

public abstract class SQLiteCursorLoader extends AsyncTaskLoader<Cursor> {
    private Cursor mCursor;

    public SQLiteCursorLoader(Context context) {
        super(context);
    }
//定义抽象方法，子类可以利用此方法定义Cursor加载。
    protected abstract Cursor loadCursor();

    @Override
    public Cursor loadInBackground() {
        Cursor cursor = loadCursor();
        if (cursor != null) {
            // ensure that the content window is filled
            //通过调用一个方法预加载内容。
            cursor.getCount();
        }
        return cursor;
    }
    //数据更新时
    @Override
    public void deliverResult(Cursor data) {
        Cursor oldCursor = mCursor;//保存旧的cursor引用。
        mCursor = data;

        if (isStarted()) {
            super.deliverResult(data);
        }
        //为防止cursor缓存，所以检测新旧cursor引用的一致性。然后判断是否关闭旧的cursor
        if (oldCursor != null && oldCursor != data && !oldCursor.isClosed()) {
            oldCursor.close();
        }
    }
    @Override
    protected void onStartLoading() {
        if (mCursor != null) {
            deliverResult(mCursor);
        }
        if (takeContentChanged() || mCursor == null) {
            forceLoad();
        }
    }
    @Override
    protected void onStopLoading() {
        // Attempt to cancel the current load task if possible.
        cancelLoad();
    }
    @Override
    public void onCanceled(Cursor cursor) {
        if (cursor != null && !cursor.isClosed()) {
            cursor.close();
        }
    }
    @Override
    protected void onReset() {
        super.onReset();
        // Ensure the loader is stopped
        onStopLoading();
        if (mCursor != null && !mCursor.isClosed()) {
            mCursor.close();
        }
        mCursor = null;
    }
}

```
然后定义它的子类：
```

public class MyRssCursorLoader  extends SQLiteCursorLoader {
    private final SQLiteOpenHelper helper;
    public MyRssCursorLoader(Context context) {
        super(context);
        helper = RssUtils.getInstance().getSQLiteOpenHelper();
    }

    @Override
    protected Cursor loadCursor() {
        // query the list of runs
       SQLiteDatabase db=helper.getWritableDatabase();
        RSSModelDao dao=RssUtils.getInstance().getRssDao(db);
       return db.query(dao.getTablename(),dao.getAllColumns(),null,null,null,null,"_id ASC");
    }
}

```
#  自定义Loader异步加载Model数据模型 
```

public abstract class RSSModelLoader extends AsyncTaskLoader<RSSModel> {
    private RSSModel mModel;

    public RSSModelLoader(Context context) {
        super(context);
    }

    protected abstract RSSModel loadRSSModel();

    @Override
    public RSSModel loadInBackground() {
        RSSModel model = loadRSSModel();
        if (model != null) {
            // ensure that the content window is filled
            //通过调用一个方法预加载内容。
//        cursor.getCount();
            model.getId();
        }
        return model;
    }

    //数据更新时
    @Override
    public void deliverResult(RSSModel data) {
        RSSModel oldModel = mModel;//保存旧的cursor引用。
        mModel = data;

        if (isStarted()) {
            super.deliverResult(data);
        }
//        为防止cursor缓存，所以检测新旧cursor引用的一致性。然后判断是否关闭旧的cursor
        if (oldModel != null && oldModel != data) {
            oldModel = null;
        }
    }

    @Override
    protected void onStartLoading() {
        if (mModel != null) {
            deliverResult(mModel);
        }
        if (takeContentChanged() || mModel == null) {
            forceLoad();
        }
    }

    @Override
    protected void onStopLoading() {
        // Attempt to cancel the current load task if possible.
        cancelLoad();
    }
    @Override
    public void onCanceled(RSSModel model) {
        if (model != null) {
            model = null;
        }
    }
    @Override
    protected void onReset() {
        super.onReset();

        // Ensure the loader is stopped
        onStopLoading();

        if (mModel != null) {
            mModel=null;
        }
        mModel = null;
    }
}

```
```

public class MyRssModelLoader extends RSSModelLoader {
    private final SQLiteOpenHelper helper;
    private long id;
    private final SQLiteDatabase db;
    private final RSSModelDao dao;
    private RSSModel cacheModel;

    public MyRssModelLoader(Context ctx,long id){
        super(ctx);
        helper = RssUtils.getInstance().getSQLiteOpenHelper();
        db = helper.getWritableDatabase();
        dao = RssUtils.getInstance().getRssDao(db);
        this.id=id+1;

    }
    @Override
    protected RSSModel loadRSSModel() {
//        return null;

//        return db.query(dao.getTablename(),dao.getAllColumns(),null,null,null,null,"_id ASC");
        Log.d("haha", id + ":RSSModelLoader");
        return dao.load(id);
    }
}

```
##  提醒： 

不要忘记在Activity中或者Fragment中实现LoaderManager.LoaderCallbacks回调接口

#  Demo示例（重要） 
下面通过一个Demo来讲解一下Loaders的使用。在这个Demo中，数据使用SQLite数据库保存，而使用ContentProvider进行数据的请求与访问。在SQLite数据库中，存在一个Student表，它近有两个字段：_id,name。在Demo中，使用一个ListView展示数据，使用LoaderManager管理一个Loader，并通过这个Loader的回调接口进行刷新ListView的数据显示。进行对SQLite数据库中的数据进行增加与删除。下面不提供SQLiteOpenHelper和ContentProvider相关实现类的代码。如有需要可以下载源码查看，
```

package com.example.loadermanagerdemo;

import android.net.Uri;
import android.os.Bundle;
import android.app.Activity;
import android.app.AlertDialog;
import android.app.LoaderManager;
import android.app.LoaderManager.LoaderCallbacks;
import android.content.ContentResolver;
import android.content.ContentValues;
import android.content.CursorLoader;
import android.content.Loader;
import android.database.Cursor;
import android.util.Log;
import android.view.ContextMenu;
import android.view.LayoutInflater;
import android.view.Menu;
import android.view.MenuInflater;
import android.view.MenuItem;
import android.view.View;
import android.view.ContextMenu.ContextMenuInfo;
import android.widget.AdapterView.AdapterContextMenuInfo;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.SimpleCursorAdapter;
import android.widget.TextView;

public class MainActivity extends Activity {
    private LoaderManager manager;
    private ListView listview;
    private AlertDialog alertDialog;
    private SimpleCursorAdapter mAdapter;
    private final String TAG="main";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        listview = (ListView) findViewById(R.id.listView1);
        //使用一个SimpleCursorAdapter，布局使用android自带的布局资源simple_list_item_1， android.R.id.text1 为simple_list_item_1中TextView的Id
        mAdapter = new SimpleCursorAdapter(MainActivity.this,
                android.R.layout.simple_list_item_1, null,
                new String[] { "name" }, new int[] { android.R.id.text1 },0);
        
        // 获取Loader管理器。
        manager = getLoaderManager();
        // 初始化并启动一个Loader，设定标识为1000，并制定一个回调函数。
        manager.initLoader(1000, null, callbacks);

        // 为ListView注册一个上下文菜单
        registerForContextMenu(listview);
    }

    @Override
    public void onCreateContextMenu(ContextMenu menu, View v,
            ContextMenuInfo menuInfo) {
        super.onCreateContextMenu(menu, v, menuInfo);
        // 声明一个上下文菜单，contentmenu中声明了两个菜单，添加和删除
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.contentmenu, menu);
    }

    @Override
    public boolean onContextItemSelected(MenuItem item) {
        
        switch (item.getItemId()) {
        case R.id.menu_add:
            // 声明一个对话框
            AlertDialog.Builder builder = new AlertDialog.Builder(
                    MainActivity.this);
            // 加载一个自定义布局，add_name中有一个EditText和Button控件。
            final View view = LayoutInflater.from(MainActivity.this).inflate(
                    R.layout.add_name, null);
            Button btnAdd = (Button) view.findViewById(R.id.btnAdd);
            btnAdd.setOnClickListener(new View.OnClickListener() {

                @Override
                public void onClick(View v) {
                    EditText etAdd = (EditText) view
                            .findViewById(R.id.username);
                    String name = etAdd.getText().toString();
                    // 使用ContentResolver进行删除操作，根据name字段。
                    ContentResolver contentResolver = getContentResolver();
                    ContentValues contentValues = new ContentValues();
                    contentValues.put("name", name);
                    Uri uri = Uri
                            .parse("content://com.example.loadermanagerdemo.StudentContentProvider/student");
                    Uri result = contentResolver.insert(uri, contentValues);
                    if (result != null) {
                        //result不为空证明删除成功，重新启动Loader，注意标识需要和之前init的标识一致。
                        manager.restartLoader(1000, null, callbacks);
                    }
                    // 关闭对话框
                    alertDialog.dismiss();
                    
                    Log.i(TAG, "添加数据成功，name="+name);
                }
            });
            builder.setView(view);
            alertDialog = builder.show();
            return true;
        case R.id.menu_delete:
            // 获取菜单选项的信息
            AdapterContextMenuInfo info = (AdapterContextMenuInfo) item
                    .getMenuInfo();
            // 获取到选项的TextView控件，并得到选中项的那么
            TextView tv = (TextView) info.targetView;
            String name = tv.getText().toString();
            // 使用ContentResolver进行删除操作
            Uri url = Uri
                    .parse("content://com.example.loadermanagerdemo.StudentContentProvider/student");
            ContentResolver contentResolver = getContentResolver();
            String where = "name=?";
            String[] selectionArgs = { name };
            int count = contentResolver.delete(url, where, selectionArgs);
            if (count == 1) {
                //这个操作仅删除单挑记录，如果删除行为1 ，则重新启动Loader
                manager.restartLoader(1000, null, callbacks);
            }
            Log.i(TAG, "删除数据成功，name="+name);
            return true;
        default:
            return super.onContextItemSelected(item);
        }

    }

    // Loader的回调接口
    private LoaderManager.LoaderCallbacks<Cursor> callbacks = new LoaderCallbacks<Cursor>() {

        @Override
        public Loader<Cursor> onCreateLoader(int id, Bundle bundle) {
            // 在Loader创建的时候被调用，这里使用一个ContentProvider获取数据，所以使用CursorLoader返回数据
            Uri uri = Uri
                    .parse("content://com.example.loadermanagerdemo.StudentContentProvider/student");
            CursorLoader loader = new CursorLoader(MainActivity.this, uri,
                    null, null, null, null);
            Log.i(TAG, "onCreateLoader被执行。");
            return loader;
        }

        @Override
        public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
            //刷新SimpleCursorAdapter的数据
            mAdapter.swapCursor(cursor);
            // 重新设定适配器
            listview.setAdapter(mAdapter);
            Log.i(TAG, "onLoadFinished被执行。");
        }

        @Override
        public void onLoaderReset(Loader<Cursor> loader) {
            // 当Loader被从LoaderManager中移除的时候，被执行，清空SimpleCursorAdapter适配器的Cursor
            mAdapter.swapCursor(null);
            Log.i(TAG, "onLoaderReset被执行。");
        }
    };

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

}

```
效果：
![](/data/dokuwiki/android/pasted/20150516-151641.png)
源码：
![](/data/dokuwiki/android/loadermanagerdemo.zip|)

参考：
[Android--Loaders](http://www.cnblogs.com/plokmju/p/android_Loaders.html)