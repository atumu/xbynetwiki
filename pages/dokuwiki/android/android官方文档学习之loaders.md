title: android官方文档学习之loaders 

#  android官方文档学习之Loaders 
从Android 3.0开始，Android引入loaders功能，loaders提供了在activity和fragment中异步载入数据以及监视数据源的变化的能力。Loaders的特性如下：
  * 在每个Activity和Fragment都可用；
  * 实现异步加载数据
  * 监控源数据的变化，当数据发生变化的时候获取新的数据；
  * 他们最后的装载机光标自动重新连接到配置更改后创建时。因此，他们并不需要重新查询自己的数据。

##  Loader API总结： 
![](/data/dokuwiki/android/pasted/20150606-183818.png)
##  在应用程序中使用Loaders 
通常使用装载器的应用程序包括以下内容：
  * 一个Activity或者Fragment
  * 一个` LoaderManager `的实例,通常通过` getLoaderManager() `获取
  * 一个` Cousorloader `通过ContentProvider加载备份数据。另外，你可以实现你自己` Loader或AsyncTaskLoader `的子类加载一些其他来源的数据。
  * ` LoaderManager.LoaderCallbacks `的实现，这是你创建一个新的Loader和管理已经存在的loader的参考
  * 显示loader数据的方式,例如，` SimpleCursorAdapter `。
  * 一个数据源，例如：` ContentProvider `，当使用CursorLoader时。
###  启动一个Loader 
LoaderManager通过Activity或者Fragment来管理一个或者多个Loader实例。在一个Activity或者Fragment中仅仅只有一个LoaderManager。
你通常的可以通过` Activity的onCreate() `方法来初始化一个Loader，或者通过` Fragment的onActivityCreated() `方法。你可以按照下面的做：
```

   // Prepare the loader.  Either re-connect with an existing one,
  // or start a new one.
  getLoaderManager().initLoader(0, null, this);

```
initLoader()方法需要以下几个参数：
一个唯一的Id标识，在这个例子中Id是0。
构造函数的可选参数（在这里为空）
LoaderManager.LoaderCallbacks的实现，LoaderManager调用来报告Loader事件。在这个例子中，本类实现` 了LoaderManager.LoaderCallbacks `接口，它传递自身的引用，this。
` initLoader() `调用，确保Loader被初始化，以及存在。它可能有两种结果：
  * 如果通过Id标识的Loader已经存在，上次被创建的Loader将会被重用。
  * 如果不存在的， 由ID标识的Loader，initLoader（）方法触发`  LoaderManager.LoaderCallbacks的onCreateLoader（） `方法 。**这是你实现的代码来实例化并返回一个新的Loader**。更多的讨论，请参阅节onCreateLoader。

###  重新启动一个Loader 
当你使用initLoader(),它将使用指定Id的已经存在的Loader。如果没有，将会创建一个。**但有时你要放弃你的旧数据，并重新开始。**
想要丢弃您的旧数据，您使用` restartLoader（） `方法。例如：当用户的查询状态改变时，实现SearchView.OnQueryTextListener的类，会重启Loader。Loader需要重新启动，以便它可以使用修改后的搜索过滤器，做一个新的查询：
`  getLoaderManager().restartLoader(0, null, this); `
###  使用LoaderManager回调 
` LoaderManager.LoaderCallbacks `是一个回调接口，可以让客户端和LoaderManager交互。
LoaderManager.LoaderCallbacks包括这些方法：
  * onCreateLoader() — Instantiate and return a new Loader for the given ID.
  * onLoadFinished() — Called when a previously created loader has finished its load.
  * onLoaderReset() — Called when a previously created loader is being reset, thus making its data unavailable.

```

 String mCurFilter;
  ...
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
   // This is the Adapter being used to display the list's data.
  SimpleCursorAdapter mAdapter;
  ...
  public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
    // Swap the new cursor in.  (The framework will take care of closing the
    // old cursor once we return.)
    mAdapter.swapCursor(data);
  }

  // This is the Adapter being used to display the list's data.
  SimpleCursorAdapter mAdapter;
  ...
  public void onLoaderReset(Loader<Cursor> loader) {
    // This is called when the last Cursor provided to onLoadFinished()
    // above is about to be closed.  We need to make sure we are no
    // longer using it.
    mAdapter.swapCursor(null);
  }

```
##  例子 
作为一个例子，这里完整的实现了一个Fragment，它显示了一个ListView，该ListView包含的查询结果与联系人内容提供者一致。它使用一个CursorLoader在提供者上管理查询。
对于应用程序访问用户的联系人，就像本例所示，它的清单中必须包含READ_CONTACTS许可。
```

public static class CursorLoaderListFragment extends ListFragment
        implements OnQueryTextListener, LoaderManager.LoaderCallbacks<Cursor> {
    // This is the Adapter being used to display the list's data.
    SimpleCursorAdapter mAdapter;
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
        // Prepare the loader.  Either re-connect with an existing one,
        // or start a new one.
        getLoaderManager().initLoader(0, null, this);
    }
    @Override public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
        // Place an action bar item for searching.
        MenuItem item = menu.add("Search");
        item.setIcon(android.R.drawable.ic_menu_search);
        item.setShowAsAction(MenuItem.SHOW_AS_ACTION_IF_ROOM);
        SearchView sv = new SearchView(getActivity());
        sv.setOnQueryTextListener(this);
        item.setActionView(sv);
    }
    public boolean onQueryTextChange(String newText) {
        // Called when the action bar search text has changed.  Update
        // the search filter, and restart the loader to do a new query
        // with this filter.
        mCurFilter = !TextUtils.isEmpty(newText) ? newText : null;
        getLoaderManager().restartLoader(0, null, this);
        return true;
    }
    @Override public boolean onQueryTextSubmit(String query) {
        // Don't care about this.
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
    }
    public void onLoaderReset(Loader<Cursor> loader) {
        // This is called when the last Cursor provided to onLoadFinished()
        // above is about to be closed.  We need to make sure we are no
        // longer using it.
        mAdapter.swapCursor(null);
    }
 }
 

```
在这个例子中，onCreateLoader（） 回调方法创建一个CursorLoader。你必须使用CursorLoader的构造方法构造它，需要ContentProvider执行查询时所需要的很多信息。具体来说，它需要：
Uri-内容检索的URI
projection — 要返回的列元素。如果为空的话，返回所有的列，但是这效率比较低。
selection — 声明返回行的过滤器，格式化为一个SQL WHERE子句（不含WHERE本身）。传递为空，将返回给定的URI的所有行。
selectionArgs —你可以包含？在selection中，将会被 selectionArgs的值替代，该值将被绑定为字符串。
SortOrder — 如何对列进行排序，格式化为SQL令（不含ORDER本身）BY子句。如果为空将使用默认的排序顺序，这可能是无序的。
更多的例子
有几个不同的例子在ApiDemos中，说明如何使用装载机：
  * LoaderCursor
  * LoaderThrottle