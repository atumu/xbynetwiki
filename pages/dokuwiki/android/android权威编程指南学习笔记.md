title: android权威编程指南学习笔记 

#  Android权威编程指南学习笔记 
android.support.v7.app.AppCompatActivity：保证兼容性
Android Gradle编译工具：
 gradlew.bat tasks 显示一系列可用任务
gradlew.bat installDebug 将应用安装到连接的设备上。

AndroidStudio配置成员变量前缀为m，
首选项-Editor-CodeStyle-Java-Code Generation选项卡下的Naming表单中，选择Fields行，添加前缀。

Gradle更改同步 Tool-Android-Sync Project with Gradle Files

https://developer.android.com/samples/index.html
https://github.com/googlesamples?page=1


**字符串占位符形**如%1$s

**兼容性与SDK版本**
```

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) { ... }

```

**Activity生命周期状态图解**
![](/data/dokuwiki/android/pasted/20161008-203703.png)

**设备配置与备选资源**
设备配置(device configuration)是用来描述设备当前状态的一系列特征。如屏幕方向，屏幕密度，屏幕尺寸，键盘类型，语言等。有些是固定的，有些是可以运行时改变的。
如屏幕方向，键盘隐藏，语言等等是可以在运行时变更的。一旦发生变更，Activity就会销毁重建，并选择更合适的资源来匹配新的设备变更。
所有的资源文件夹后缀查看：file:/ / /F:/Android/sdk/docs/guide/topics/resources/providing-resources.html
**设备旋转前保存数据**
覆盖Activity的onSaveInstanceState(Bundle)方法。
如果是Fragment，在onCreate方法中调用setRetainInstance(true);设置为保留的Fragment.


Serializable，Parcelable接口。
**Context**
Activity,Service等等都是Context的子类。
Activity Context与` ApplicationContext `的区别：存活时间不同。
##  Activity之间的通信 
AppCompat-v7兼容库
AppCompat兼容库能将部分最新系统的特色功能移植到Android旧版本系统中。
第一个Activity:
```

boolean answerIsTrue = mQuestionBank[mCurrentIndex].isAnswerTrue();
Intent i = CheatActivity.newIntent(QuizActivity.this, answerIsTrue);
//startActivityForResult
startActivityForResult(i, REQUEST_CODE_CHEAT);
//onActivityResult
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (resultCode != Activity.RESULT_OK) {
            return;
        }

        if (requestCode == REQUEST_CODE_CHEAT) {
            if (data == null) {
                return;
            }
            mIsCheater = CheatActivity.wasAnswerShown(data);
        }
    }

```
第二个Activity：
```

	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_cheat);
	  //getIntent()获取启动参数
        mAnswerIsTrue = getIntent().getBooleanExtra(EXTRA_ANSWER_IS_TRUE, false);
    }
   
	   Intent data = new Intent();
        data.putExtra(EXTRA_ANSWER_SHOWN, isAnswerShown);
 	//setResult返回值
        setResult(RESULT_OK, data);

```


##  Fragment 
![](/data/dokuwiki/android/pasted/20161008-211501.png)
```

public abstract  class SingleFragmentActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(getLayoutId());
        FragmentManager fm=getFragmentManager();
        Fragment f=fm.findFragmentById(R.id.fragmentContainer);
        if(f==null){
            f=createFragment();
            fm.beginTransaction().add(R.id.fragmentContainer,f).commit();
        }
    }
    protected abstract Fragment createFragment();
    protected abstract int getLayoutId();
}

```
**fragment参数**
```

  public static Fragment newInstance(UUID id) {
        Bundle bundle = new Bundle();
        bundle.putSerializable(EXTRA_CRIME_ID, id);
        CrimeFragment f = new CrimeFragment();
        f.setArguments(bundle);
        return f;
    }


 UUID cid = (UUID) getArguments().getSerializable(EXTRA_CRIME_ID);

```


## RecyclerView代替ListView、GridView显示
RecyclerView可以控制按需创建视图对象。它只创建刚好充满屏幕的视图。而不是所有item的视图。当滑动屏幕切换视图时，上一个视图会被回收利用。
RecyclerView来自于一个叫` recyclerview-v7的支持库 `。使用时需要添加该依赖。

RecyclerView需要结合RecyclerView.ViewHolder子类,RecyclerView.Adapter子类一起使用。同时还需要配置LayoutManager来管理和定位视图布局。
ViewHolder只做一件事：容纳View视图。当然也可以添加绑定数据到视图，初始化视图监听器等功能。
RecyclerView.Adapter是个控制器对象，从模型层获取数据，然后提供给RecyclerView显示：
1、创建必要的ViewHolder.(onCreateViewHolder)
2、绑定ViewHolder至模型层数据.(onBindViewHolder)

示例：fragment_crime_list.xml RecyclerView作为根元素。
```

<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.RecyclerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/crime_recycler_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>

```
CrimeListFragment.java
```

@Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_crime_list, container, false);

        mCrimeRecyclerView = (RecyclerView) view.findViewById(R.id.crime_recycler_view);
      //必须配置LayoutManager
        mCrimeRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));

        updateUI();
        return view;
    }
    private void updateUI() {
        CrimeLab crimeLab = CrimeLab.get(getActivity());
        List<Crime> crimes = crimeLab.getCrimes();
	if(mAdapter==null){
        mAdapter = new CrimeAdapter(crimes);
      //setAdapter
        mCrimeRecyclerView.setAdapter(mAdapter);
      }else{
     	mAdapter.notifyDataSetChanged();   
      }
    }
	//内部类的形式实现RecyclerView.ViewHolder，并实现监听器接口。
	private class CrimeHolder extends RecyclerView.ViewHolder
            implements View.OnClickListener {

        private TextView mTitleTextView;
        private TextView mDateTextView;
        private CheckBox mSolvedCheckBox;

        private Crime mCrime;

        public CrimeHolder(View itemView) {
            super(itemView);
            itemView.setOnClickListener(this);

            mTitleTextView = (TextView) itemView.findViewById(R.id.list_item_crime_title_text_view);
            mDateTextView = (TextView) itemView.findViewById(R.id.list_item_crime_date_text_view);
            mSolvedCheckBox = (CheckBox) itemView.findViewById(R.id.list_item_crime_solved_check_box);
        }

        public void bindCrime(Crime crime) {
            mCrime = crime;
            mTitleTextView.setText(mCrime.getTitle());
            mDateTextView.setText(mCrime.getDate().toString());
            mSolvedCheckBox.setChecked(mCrime.isSolved());
        }

        @Override
        public void onClick(View v) {
            Toast.makeText(getActivity(),
                    mCrime.getTitle() + " clicked!", Toast.LENGTH_SHORT)
                    .show();
        }
    }
	//内部类的形式实现RecyclerView.Adapter
    private class CrimeAdapter extends RecyclerView.Adapter<CrimeHolder> {

        private List<Crime> mCrimes;

        public CrimeAdapter(List<Crime> crimes) {
            mCrimes = crimes;
        }
		//onCreateViewHolder
        @Override
        public CrimeHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        	// LayoutInflater.from(getActivity());
            LayoutInflater layoutInflater = LayoutInflater.from(getActivity());
            View view = layoutInflater.inflate(R.layout.list_item_crime, parent, false);
            return new CrimeHolder(view);
        }
		//onBindViewHolder
        @Override
        public void onBindViewHolder(CrimeHolder holder, int position) {
            Crime crime = mCrimes.get(position);
            holder.bindCrime(crime);
        }
		//getItemCount
        @Override
        public int getItemCount() {
            return mCrimes.size();
        }
    }

```
RecyclerView扩展性、列表动画效果的原生支持，强制的ViewHolder模式。优于ListView、GridView
例如:列表项要从位置0移到位置5.
```

mRecyclerView.getAdapter().notifyItemMoved(0,5);

```

单例在UI线程上共享数据的使用。

##  使用ViewPager 
ViewPager类来自于支持库中，所以使用时需要添加v4支持库
activity_crime_pager.xml
```

<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.view.ViewPager
    android:id="@+id/activity_crime_pager_view_pager"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>

```
ViewPager与PagerAdapter(FragmentStatePagerAdapter,FragmentPageAdapter)
FragmentStatePagerAdapter提供了两个有用的方法 getCount和getItem(int)
```

private ViewPager mViewPager;
private List<Crime> mCrimes;


   @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_crime_pager);

        UUID crimeId = (UUID) getIntent()
                .getSerializableExtra(EXTRA_CRIME_ID);

        mViewPager = (ViewPager) findViewById(R.id.activity_crime_pager_view_pager);

        mCrimes = CrimeLab.get(this).getCrimes();
        FragmentManager fragmentManager = getSupportFragmentManager();
        mViewPager.setAdapter(new FragmentStatePagerAdapter(fragmentManager) {
            @Override
            public Fragment getItem(int position) {
                Crime crime = mCrimes.get(position);
                return CrimeFragment.newInstance(crime.getId());
            }

            @Override
            public int getCount() {
                return mCrimes.size();
            }
        });

        for (int i = 0; i < mCrimes.size(); i++) {
            if (mCrimes.get(i).getId().equals(crimeId)) {
                mViewPager.setCurrentItem(i); //设置当前显示的Item
                break;
            }
        }
    }

```
FragmentStatePagerAdapter与FragmentPageAdapter的区别:体现在卸载不再需要的Fragment时采取的行为：
FragmentStatePagerAdapter:会销毁不再需要的Fragment。但是会保留该Fragment的状态。
FragmentPageAdapter：会选择调用事务的detach(Fragment)方法来处理。它只是销毁了视图，而fragment的实例还保留在FragmentManager中。
相比之下FragmentStatePagerAdapter会更加节省内存，但是当所需要的Pager数量很少，少到只有几个时，用FragmentPageAdapter反而体现出了优势。

ViewPager的工作原理：略。。。
##  对话框与Fragment间数据传递 
建议将AlertDialog封装在DialogFragment中。（这样设备旋转后也能显示对话框）
Fragment间数据传递涉及targetFragment的配置
```

mDateButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                FragmentManager manager = getFragmentManager();
                DatePickerFragment dialog = DatePickerFragment
                        .newInstance(mCrime.getDate());
              //为DialogFragment配置TargetFragment
                dialog.setTargetFragment(CrimeFragment.this, REQUEST_DATE);
              //调用其show方法显示对话框
                dialog.show(manager, DIALOG_DATE_CODE);
            }
        });

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (resultCode != Activity.RESULT_OK) {
            return;
        }

        if (requestCode == REQUEST_DATE_CODE) {
            Date date = (Date) data
                    .getSerializableExtra(DatePickerFragment.EXTRA_DATE);
            mCrime.setDate(date);
            updateDate();
        }
    }

```
```

import android.support.v4.app.DialogFragment;
import android.support.v7.app.AlertDialog;

public class DatePickerFragment extends DialogFragment {

    public static final String EXTRA_DATE =
            "com.bignerdranch.android.criminalintent.date";
    private static final String ARG_DATE = "date";
    private DatePicker mDatePicker;

    public static DatePickerFragment newInstance(Date date) {
        Bundle args = new Bundle();
        args.putSerializable(ARG_DATE, date);
        DatePickerFragment fragment = new DatePickerFragment();
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        Date date = (Date) getArguments().getSerializable(ARG_DATE);
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        int year = calendar.get(Calendar.YEAR);
        int month = calendar.get(Calendar.MONTH);
        int day = calendar.get(Calendar.DAY_OF_MONTH);
        View v = LayoutInflater.from(getActivity())
                .inflate(R.layout.dialog_date, null);
        mDatePicker = (DatePicker) v.findViewById(R.id.dialog_date_date_picker);
        mDatePicker.init(year, month, day, null);
        return new AlertDialog.Builder(getActivity())
                .setView(v) //setView
                .setTitle(R.string.date_picker_title)
                .setPositiveButton(android.R.string.ok,
                        new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                int year = mDatePicker.getYear();
                                int month = mDatePicker.getMonth();
                                int day = mDatePicker.getDayOfMonth();
                                Date date = new GregorianCalendar(year, month, day).getTime();
                                sendResult(Activity.RESULT_OK, date); //设置返回值
                            }
                })
                .create();
    }

    private void sendResult(int resultCode, Date date) {
        if (getTargetFragment() == null) {
            return;
        }

        Intent intent = new Intent();
        intent.putExtra(EXTRA_DATE, date);
	
      	//直接调用targetFragment的onActivityResult方法。getTargetRequestCode()获取请求码
        getTargetFragment()
                .onActivityResult(getTargetRequestCode(), resultCode, intent);
    }
}


```

##  Toolbar(Actionbar已经过时) 
详细请参考：https://wiki.xby1993.net/doku.php?id=android:android_toolbar
Android5.0以后开始采用Toolbar来取代Actionbar，但是在appcompat-v7的兼容库中也有。
**工具栏**还能支持` 内嵌视图和调整高度、位置，还可以为每个fragment定制工具栏，可以配置多个工具栏。 `
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

获取MenuItem的actionView方法：` MenuItemCompat `.getActionView()
![](/data/dokuwiki/android/pasted/20161010-204705.png?200x300)
控件 (component)
![](/data/dokuwiki/android/pasted/20161010-204752.png)![](/data/dokuwiki/android/pasted/20161010-204854.png)
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
在Fragment中使用Toolbar，额外注意：
` setHasOptionsMenu(true); `

##  数据存储-SQLite与SharedPreferences、Internal Storage、External Storage 
**SharedPreferences**，存储路径/data/data/<your pkg name>/下
```

默认的
PreferenceManager.getDefaultSharedPreferences(getActivity()).edit().putString(QUERY_KEY,query).commit();
String query=PreferenceManager.getDefaultSharedPreferences(getActivity()).getString(QUERY_KEY,null);
非默认的
spf=ctx.getSharedPreferences(PREFS_FILE,Context.MODE_PRIVATE);

```
**Internal Storage**：存储路径/data/data/<your pkg name>/files/xxx
面向接口Context:
  * openFileOutput()
  * openFileInput(),还有个**openRawResource()**用于读取/res/raw/下的文件，传入资源ID:R.raw.<filename>
  * getCacheDir()  /data/data/<your pkg name>/cache目录，` 要记得及时清理 `
  * getFilesDir() /data/data/<your pkg name>/files目录
  * getDir()     /data/data/<your pkg name>/目录
  * deleteFile()
  * fileList()  /data/data/<your pkg name>/files目录下文件列表

```

String FILENAME = "hello_file";
String string = "hello world!";
//MODE_PRIVATE,MODE_APPEND,(MODE_WORLD_READABLE和MODE_WORLD_WRITEABLE已废弃)
FileOutputStream fos = ctx.openFileOutput(FILENAME, Context.MODE_PRIVATE);
fos.write(string.getBytes());
fos.close();

```

**External Storage**
权限：
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
面向接口：Context(ContextCompat)，Environment
  * Environment.getExternalStorageState()
  * Environment.getExternalStorageDirectory()
  * Environment.getExternalStoragePublicDirectory() 传入参数为 DIRECTORY_MUSIC, DIRECTORY_PICTURES, DIRECTORY_RINGTONES
  * ContextCompat.getExternalFilesDir(string) 参数为DIRECTORY_前缀的Environment常量
  * ContextCompat.getExternalFilesDirs(string)
  * ContextCompat.getExternalCacheDir()
  * ContextCompat.getExternalCacheDirs()
` 如果要支持4.3及以下低版本可使用ContextCompat来代替Contxt的使用 `，如ContextCompat.getExternalFilesDirs()

```

public File getAlbumStorageDir(String albumName) {
    // Get the directory for the user's public pictures directory.
    File file = new File(Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES), albumName);
    if (!file.mkdirs()) {
        Log.e(LOG_TAG, "Directory not created");
    }
    return file;
}

```
```

/* Checks if external storage is available for read and write */
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}

/* Checks if external storage is available to at least read */
public boolean isExternalStorageReadable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state) ||
        Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
        return true;
    }
    return false;
}

```



**SQLite**： 数据路径/data/data/<your pkg name>/databases/xxx.db
需要关注的一些类或接口
  * android.content.ContentValues; 用于插入或更新时提供参数
  * android.database.Cursor;    游标
  * android.database.CursorWrapper;  游标包装，继承它并包装游标，让你可以在游标中提供额外的方法。
  * android.database.sqlite.SQLiteDatabase; 
  * android.database.sqlite.SQLiteOpenHelper; 数据库操作便利类。一般都需要继承它。
```

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.CursorWrapper;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

//继承SQLiteOpenHelper作为dao
public class RunDatabaseHelper extends SQLiteOpenHelper {
    public static final String TAG = "RunDatabaseHelper";
    private static final String DB_NAME="db_run"; //数据库名
    private static final int DB_VERSION=1; //数据库版本，这是必要的，为以后数据表变更提供升级的可能。

    private static final String TABLE_RUN="run"; 

    private static final String COLUMN_RUN_ID = "_id";
    private static final String COLUMN_RUN_START_DATE="start_date";
    private static final String TABLE_LOCATION = "location";
    private static final String COLUMN_LOCATION_LATITUDE = "latitude";
    private static final String COLUMN_LOCATION_LONGITUDE = "longitude";
    private static final String COLUMN_LOCATION_ALTITUDE = "altitude";
    private static final String COLUMN_LOCATION_TIMESTAMP = "timestamp";
    private static final String COLUMN_LOCATION_PROVIDER = "provider";
    private static final String COLUMN_LOCATION_RUN_ID = "run_id";


    public RunDatabaseHelper(Context context) {
      //中间那个null，代表的是nullColumnHack,用于插入或更新时，出入的ContentValues对象为null时，提供一个空的值，从而防止方法调用失败。
        super(context, DB_NAME, null, DB_VERSION);
    }
	//初始化创建数据表
    @Override
    public void onCreate(SQLiteDatabase db) {
        String createTableRun="create table run ( _id integer primary key autoincrement,start_date integer ) ";
        db.execSQL(createTableRun);
        String createTableLocation="create table location ( timestamp integer, latitude real,longtitude real,altitude real,provider varchar(100),run_id integer references run(_id)) ";
        db.execSQL(createTableLocation);
    }
	//数据表变动，数据库升级时
    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
	//插入方法用到了ContentValues
    public long insertRun(Run run){
        ContentValues cv=new ContentValues();
        cv.put(COLUMN_RUN_START_DATE,run.getStartDate().getTime());
      //getWritableDatabase()
        return getWritableDatabase().insert(TABLE_RUN,null,cv);
    }
    public long insertLocation(long runId, Location location) {
        ContentValues cv = new ContentValues();
        cv.put(COLUMN_LOCATION_LATITUDE, location.getLatitude());
        cv.put(COLUMN_LOCATION_LONGITUDE, location.getLongitude());
        cv.put(COLUMN_LOCATION_ALTITUDE, location.getAltitude());
        cv.put(COLUMN_LOCATION_TIMESTAMP, location.getTime());
        cv.put(COLUMN_LOCATION_PROVIDER, location.getProvider());
        cv.put(COLUMN_LOCATION_RUN_ID, runId);
        return getWritableDatabase().insert(TABLE_LOCATION, null, cv);
    }
    public RunCursor queryRuns() {
        // equivalent to "select * from run order by start_date asc"
        Cursor wrapped = getReadableDatabase().query(TABLE_RUN,
                null, null, null, null, null, COLUMN_RUN_START_DATE + " asc");
        return new RunCursor(wrapped);
    }
    public RunCursor queryRun(long id) {
        Cursor wrapped = getReadableDatabase().query(TABLE_RUN,
                null, // all columns
                COLUMN_RUN_ID + " = ?", // look for a run ID
                new String[]{ String.valueOf(id) }, // with this value
                null, // group by
                null, // order by
                null, // having
                "1"); // limit 1 row
      
      //使用CursorWrapper实现类封装cursor
        return new RunCursor(wrapped);
    }

    public LocationCursor queryLastLocationForRun(long runId) {
        Cursor wrapped = getReadableDatabase().query(TABLE_LOCATION,
                null, // all columns
                COLUMN_LOCATION_RUN_ID + " = ?", // limit to the given run
                new String[]{ String.valueOf(runId) },
                null, // group by
                null, // having
                COLUMN_LOCATION_TIMESTAMP + " desc", // order by latest first
                "1"); // limit 1
        return new LocationCursor(wrapped);
    }

    
}


```
CursorWrapper实现部分
```

//继承CursorWrapper
public static class RunCursor extends CursorWrapper{
		
        public RunCursor(Cursor cursor) {
            super(cursor);
        }
  	//这就是CursorWrapper便利的体现，调用该实例的getRun()方法直接返回对应的POJO而无需每次调用都getXxxetColumnIndex(xxx))
       public Run getRun(){
            if(isBeforeFirst()||isAfterLast()) return null;
            Run run=new Run();
            run.setId(getLong(getColumnIndex(COLUMN_RUN_ID)));
            run.setStartDate(new Date(getLong(getColumnIndex(COLUMN_RUN_START_DATE))));
            return run;
        }

    }
    public static class LocationCursor extends CursorWrapper {

        public LocationCursor(Cursor c) {
            super(c);
        }

        public Location getLocation() {
            if (isBeforeFirst() || isAfterLast())
                return null;
            // first get the provider out so we can use the constructor
            String provider = getString(getColumnIndex(COLUMN_LOCATION_PROVIDER));
            Location loc = new Location(provider);
            // populate the remaining properties
            loc.setLongitude(getDouble(getColumnIndex(COLUMN_LOCATION_LONGITUDE)));
            loc.setLatitude(getDouble(getColumnIndex(COLUMN_LOCATION_LATITUDE)));
            loc.setAltitude(getDouble(getColumnIndex(COLUMN_LOCATION_ALTITUDE)));
            loc.setTime(getLong(getColumnIndex(COLUMN_LOCATION_TIMESTAMP)));
            return loc;
        }
    }

```

##  隐式Intent 
作用：告诉操作系统我们想要什么，操作系统就会去启动能够胜任工作任务的Activity。如果有多个，操作系统会提供给用户选择的界面。
组成：
1、要执行的操作-Action： 如ACTION_SEND,ACTION_VIEW,ACTION_PICK，ACTION_DIAL,ACTION_CALL等等
2、要访问数据的位置:如网页URL，content URI等等
3、操作涉及的数据类型：MIME，如text/plain等。
4、可选类别-Category：如Intent.CATEGORY_HOME,CATEGORY_INFO，CATEGORY_LAUNCHER等等
前提条件：
Intent结合Intent-filter一起工作，Activity通过IntentFilter定义了能够胜任的工作，即ACTION。而隐式Intent通过工作的名字来调用相关的Activity。

注意：有个时候隐式Intent需要有相关权限才能正常工作。

检查可响应任务的Activity：
```

final Intent pickContact = new Intent(Intent.ACTION_PICK,
                ContactsContract.Contacts.CONTENT_URI);
if (packageManager.resolveActivity(pickContact,
			//限定只搜索带CATEGORY_DEFAULT标志的activity
                PackageManager.MATCH_DEFAULT_ONLY) == null) {
            mSuspectButton.setEnabled(false);
        }

```
也可以这样
```

if (intent.resolveActivity(getPackageManager()) != null) {
            startActivity(intent);
        }

```

实例：
1、发送短信
```

			Intent i =null;
			
                if(StringUtil.isEmpty(crime.getPhoneNumber())){
                  	//如果手机号码为空，发送短信，不指定接收人
                    i= new Intent(Intent.ACTION_SEND);
                    i.setType("text/plain");
                    i.putExtra(Intent.EXTRA_SUBJECT, getString(R.string.crime_report_subject));
                    i.putExtra(Intent.EXTRA_TEXT, getCrimeReport());
                  	//当存在多个的时候，制定选择列表标题。
                    i = i.createChooser(i, getString(R.string.send_report_via));
                }else{
				//如果手机号码不为空，发送短信并指定接收人。
                    i = new Intent(Intent.ACTION_SENDTO);
                    Uri uri=Uri.parse("smsto:"+crime.getPhoneNumber());
                    i.setData(uri);
                    i.putExtra("sms_body",getCrimeReport());
                }

```
2、获取联系人姓名、号码等信息
<uses-permission android:name="android.permission.READ_CONTACTS" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

```

//操作Intent.ACTION_PICK，ContactsContract.Contacts.CONTENT_URI
Intent i = new Intent(Intent.ACTION_PICK, ContactsContract.Contacts.CONTENT_URI);
startActivityForResult(i, REQUEST_CONTACT_CODE);
 @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (resultCode != Activity.RESULT_OK) return;
        
        if (requestCode == REQUEST_CONTACT_CODE) {
            //获取位置
            Uri uri = data.getData();
            String[] queryFields = new String[]{
              		//姓名
                    ContactsContract.Contacts.DISPLAY_NAME
            };
          	//通过ContentResolver访问ContentProvider查询联系人信息。
            Cursor c = getActivity().getContentResolver().query(uri, queryFields, null, null, null);
            if (c.getCount() == 0) {
                c.close();
                return;
            } else {
                c.moveToFirst();
                String suspect = c.getString(c.getColumnIndex(ContactsContract.Contacts.DISPLAY_NAME));
                System.out.println(c.getColumnCount() + ":" + c.getColumnNames().toString());

                crime.setSuspect(suspect);
			//查询联系人号码ContactsContract.CommonDataKinds.Phone.CONTENT_URI，条件为ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME+" = '"+suspect+"'"
                Cursor numCursor=getActivity().getContentResolver().query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI,null,
                        ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME+" = '"+suspect+"'" ,null,null);
                numCursor.moveToFirst();
              		//ContactsContract.CommonDataKinds.Phone.NUMBER
                String number=numCursor.getString(numCursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));

                System.out.println("number:"+number);
                crime.setPhoneNumber(number);
                suspectBtn.setText(suspect+number);
              	  numCursor.close();
                c.close();
            }
            callback.onCrimeUpdated(crime);
        }
    }

```


Andorid支持库有个叫做ShareCompat类，它有一个**ShareCompat.IntentBuilder**内部类，利用这个内部类创建用于发送消息的Intent要方便一些。如：
```

private void sendEmail() {
        Intent intent = ShareCompat
                .IntentBuilder.from(MainActivity.this)
                .setSubject(getString(R.string.app_name))
                .addEmailTo(EMAIL)
                .setType("message/rfc822")
                .createChooserIntent();

        if (intent.resolveActivity(getPackageManager()) != null) {
            startActivity(intent);
        }
    }

```
##  使用Intent拍照 
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="18"/>
这里**android:maxSdkVersion="18"的由来**，因为Context.getExternalFilesDir(string)返回的是在主存储中应用专有目录，既然是专有目录，那么申请权限就显得有点多余了，所以自从Android4.4之后放宽了权限要求，但是如果要使用其他非专有目录还是需要申请android.permission.READ_EXTERNAL_STORAGE权限的。
<uses-feature android:name="android.hardware.camera" android:required="false"/>
MediaStore.ACTION_CAPTURE_IMAGE. MediaStore类提供了一些公共接口，用于处理图像，视频，音乐等多媒体任务。
默认只能拍摄缩略图，并将照片返回到onActivityResult的intent对象里。要想获得全尺寸的照片并通过文件系统存储。我们需要通过传入
MediaStore.EXTRA_OUTPUT指向存储路径的Uri来完成。
```

final Intent captureImage = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);

        boolean canTakePhoto = mPhotoFile != null &&
                captureImage.resolveActivity(packageManager) != null;
        mPhotoButton.setEnabled(canTakePhoto);

        if (canTakePhoto) {
            Uri uri = Uri.fromFile(mPhotoFile);
          	//
            captureImage.putExtra(MediaStore.EXTRA_OUTPUT, uri);
        }

        mPhotoButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startActivityForResult(captureImage, REQUEST_PHOTO);
            }
        });
public File getPhotoFile(Crime crime) {
        File externalFilesDir = mContext
                .getExternalFilesDir(Environment.DIRECTORY_PICTURES);

        if (externalFilesDir == null) {
            return null;
        }

        return new File(externalFilesDir, crime.getPhotoFilename());
    }

```
###  压缩图片 
还有改进的空间
```

import android.app.Activity;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Point;

public class PictureUtils {
    public static Bitmap getScaledBitmap(String path, Activity activity) {
        Point size = new Point();
        activity.getWindowManager().getDefaultDisplay().getSize(size);//获取屏幕大小

        return getScaledBitmap(path, size.x, size.y);
    }

    public static Bitmap getScaledBitmap(String path, int destWidth, int destHeight) {
        // 通过Options获取图片尺寸
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
      		//
        BitmapFactory.decodeFile(path, options);

        float srcWidth = options.outWidth;
        float srcHeight = options.outHeight;
		//缩放因子
        int inSampleSize = 1;
        if (srcHeight > destHeight || srcWidth > destWidth) {
            if (srcWidth > srcHeight) {
                inSampleSize = Math.round(srcHeight / destHeight);
            } else {
                inSampleSize = Math.round(srcWidth / destWidth);
            }
        }
		//通过选项压缩图片
        options = new BitmapFactory.Options();
        options.inSampleSize = inSampleSize;
        return BitmapFactory.decodeFile(path, options);
    }
}


```

##  Android4.x使用Camera拍照 
Android5出了Camera2,而本节讲的是使用Android4.x使用Camera接口。
```

import android.hardware.Camera;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

public class CrimeCameraFragment extends Fragment {
    private static final String TAG="CrimeCameraFragment";
    public static final String EXTRA_PICTURE_FILENAME="extra_picture_filename";
    private Button takeBtn;
    private SurfaceView surfaceView;
    private Camera camera;
    private View progressContainer;
  	//PictureTaken之后，用于显示进度条
    private Camera.ShutterCallback shutterCallback=new Camera.ShutterCallback() {
        @Override
        public void onShutter() {
            progressContainer.setVisibility(View.VISIBLE);
        }
    };
  	//
    private Camera.PictureCallback pictureCallback=new Camera.PictureCallback() {
      		//
        @Override
        public void onPictureTaken(byte[] data, Camera camera) {
            String fileName= UUID.randomUUID()+".jpg";
            FileOutputStream os=null;
            boolean success=true;
            try {
                os=getActivity().openFileOutput(fileName, Context.MODE_PRIVATE);
                os.write(data);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
                success=false;
            } catch (IOException e) {
                e.printStackTrace();
                success=false;
            }finally {
                if(os!=null){
                    try {
                        os.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                        success=false;
                    }
                }
            }
            if(success){
                Intent i=new Intent();
                i.putExtra(EXTRA_PICTURE_FILENAME,fileName);
                getActivity().setResult(Activity.RESULT_OK,i);
            }else{
                getActivity().setResult(Activity.RESULT_CANCELED);
            }
            getActivity().finish();

        }
    };
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View v=inflater.inflate(R.layout.fragment_crime_camera,container,false);
        takeBtn=(Button)v.findViewById(R.id.crime_camera_takeBtn);
        surfaceView=(SurfaceView)v.findViewById(R.id.crime_surfaceView);

        takeBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
//                getActivity().finish();
                if(camera!=null){
                  //拍照，并传入回调接口。
                    camera.takePicture(shutterCallback,null,pictureCallback);
                }

            }
        });
        progressContainer=v.findViewById(R.id.takePicture_progressContainer);
        progressContainer.setVisibility(View.INVISIBLE);
      
      //显示拍照界面SurfaceView+SurfaceHolder
        SurfaceHolder holder=surfaceView.getHolder();
        holder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
        holder.addCallback(new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {
                if(Camera.getNumberOfCameras()>1){
                    camera=Camera.open(1);//使用前置摄像头
                }else{
                    camera = Camera.open();
                }
                Log.d(TAG,"start------------------------------------------------------------");
                try {
                    if(camera!=null){
                      	//
                        camera.setPreviewDisplay(holder);
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
                if(camera==null) return;
                Camera.Parameters parameters=camera.getParameters();
                Camera.Size size=getBestSupportedSize(parameters.getSupportedPreviewSizes(),width,height);
              		//预览界面大小
                parameters.setPreviewSize(size.width,size.height);
              
                Camera.Size pSize=getBestSupportedSize(parameters.getSupportedPictureSizes(),width,height);
              		//图片大小
                parameters.setPictureSize(pSize.width,pSize.height);
              
                camera.setParameters(parameters);
              //开始显示预览界面
                    camera.startPreview();

            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {
                if(camera!=null){
                  	//
                    camera.stopPreview();
                  	//
                    camera.release();
                    camera=null;
                }
            }
        });
        return v;
    }

    private Camera.Size getBestSupportedSize(List<Camera.Size> sizes, int width, int height){
        Camera.Size bestSize=sizes.get(0);
        int largestArea=bestSize.width*bestSize.height;
        for(Camera.Size size:sizes){
            int area=size.width*size.height;
            if(area>largestArea){
                largestArea=area;
                bestSize=size;
            }
        }
        return bestSize;

    }

}


```

##  Master-Detail用户界面 
```

public abstract class SingleFragmentActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(getLayoutId());
        FragmentManager fm=getFragmentManager();
        Fragment f=fm.findFragmentById(R.id.fragmentContainer);
        if(f==null){
            f=createFragment();
            fm.beginTransaction().add(R.id.fragmentContainer,f).commit();
        }
    }
    protected abstract Fragment createFragment();

    protected abstract int getLayoutId();

}

```
为了在手机和平板上显示不同：手机上仅显示列表，而平板上同时显示列表和详细界面。我们有两种选择：
1、使用layout和layou-sw600dp定义Activity的布局
2、使用别名资源/res/values/refs.xml ,/res/values-sw600dp/refs.xml
推荐使用后者。
别名资源是一种指向其他资源的特殊资源。
```

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="activity_masterdetail" type="layout">@layout/activity_fragment</item>
</resources>

```
```

<resources>
    <item name="activity_masterdetail" type="layout">@layout/activity_twopane</item>
</resources>

```
引用的话可以R.layout.activity_masterdetail
```

<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>


```
```

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:divider="?android:attr/dividerHorizontal"
              android:showDividers="middle"
              android:orientation="horizontal">
    <FrameLayout android:id="@+id/fragment_container"
                 android:layout_width="0dp"
                 android:layout_height="match_parent"
                 android:layout_weight="1"
        />
    <FrameLayout
        android:id="@+id/detail_fragment_container"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="3" />
</LinearLayout>

```
Fragment与Fragment之间通过实现Fragment中定义的回调接口的Activity进行通信的方式：
CrimeListFragment中， 列表点击item来展示详细
```

    public interface Callbacks {
        void onCrimeSelected(Crime crime);
    }
    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        mCallbacks = (Callbacks)activity;
    }
    @Override
    public void onDetach() {
        super.onDetach();
        mCallbacks = null;
    }


```
CrimeFragment中，详细修改实时反馈给Activity，来更新列表。
```

    public interface Callbacks {
        void onCrimeUpdated(Crime crime);
    }


```
CrimeListActivity实现这些接口
```

public class CrimeListActivity extends SingleFragmentActivity
        implements CrimeListFragment.Callbacks, CrimeFragment.Callbacks
        
     @Override
    protected int getLayoutResId() {
        return R.layout.activity_masterdetail; //别名
    }
	@Override
    public void onCrimeSelected(Crime crime) {
    		//手机界面，启动detail activity
        if (findViewById(R.id.detail_fragment_container) == null) {
            Intent intent = CrimePagerActivity.newIntent(this, crime.getId());
            startActivity(intent);
            //平板界面，只是replace detail fragment
        } else {
            Fragment newDetail = CrimeFragment.newInstance(crime.getId());
            getSupportFragmentManager().beginTransaction()
                    .replace(R.id.detail_fragment_container, newDetail)
                    .commit();
        }
    }

    @Override
    public void onCrimeUpdated(Crime crime) {
        CrimeListFragment listFragment = (CrimeListFragment)
                getSupportFragmentManager()
                        .findFragmentById(R.id.fragment_container);
        listFragment.updateUI();
    }

```


##  Assets 
Android有assets和resources两大资源系统。assets一般用于存放多媒体，或者比较大的文件的地方。
assets目录与**AssetManager**
获取AssetManager asset=**ctx.getAssets();**
获取assets目录下某个文件夹的文件：String[] soundNames=**asset.list("sounds")**
` 打开文件不能使用File对象 `，而应该像下面这样:
```

InputStream in=asset.open(assetPath)

```
获取FileDescriptor：
```

AssetFileDescriptor afd=asset.openFd(assetPath)
FileDescriptor fd=afd.getFileDescriptor()

```

##  SoundPool播放音频 
SoundPool播放音频时会将整个音频文件加载到内存中然后再播放，适合小音频播放，同时支持多个音频同时播放。
构造：
Android5.0以后: SoundPool.Builder
5.0以前:SoundPool(int,int,int)如SoundPool(5,AudioManager.STREAM_MUSIC,0) 最多播放5个音频，音频流类型(铃声，闹铃，音乐等),采样率转换品质。
加载与播放:
```

AssetFileDescriptor afd = mAssets.openFd(assetPath);
int soundId = mSoundPool.load(afd, 1);//返回id，用于管理音频文件的播放、重播、卸载等。

mSoundPool.play(soundId, 1.0f, 1.0f, 1, 0, 1.0f);//使用id播放，左音道，右音道，优先级，是否循环(-1无限循环)，播放速率

```
释放音频资源:
` mSoundPool.release() `

##  样式与主题 
AppCompat库自带三大主题:
Theme.AppCompat-深色主题
Theme.AppCompat.Light-浅色主题
Theme.AppCompat.Light.DarkActionBar
Theme.AppCompat.Light.NoActionBar
```

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">  --》主题继承
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>  --》工具栏颜色
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item> --》状态栏颜色
        <item name="colorAccent">@color/colorAccent</item> --》部分组件聚焦框颜色，一般需要与状态栏颜色有一定对比
      	  <!--定制主题，由于colorBackground，buttonStyle是Android系统内置的主题属性，所以我们需要使用android:开头. -->
       	 <item name="android:colorBackground">@color/soothing_blue</item>
        <item name="android:buttonStyle">@style/BeatBoxButton</item>
    </style>
	<!--按钮样式-->
    <style name="BeatBoxButton" parent="android:style/Widget.Holo.Button">
        <item name="android:background">@color/dark_blue</item>
    </style>

```
##  XML Drawable、9-patch与mipmap 
凡是可以在屏幕上绘制的东西都叫drawable,如抽象图形,位图图像BitmapDrawable等.
XML Drawable ： state-list drawable, shape drawable, layer-list drawable. 屏幕密度无关的。
```

button_beat_box_normal
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="oval">

    <solid
        android:color="@color/dark_blue"/>
</shape>
-----------------------------------------------
另一个文件button_beat_box_pressed
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="oval">

    <solid
        android:color="@color/red"/>
</shape>

```
```

<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/button_beat_box_pressed"
          android:state_pressed="true"/>
    <item android:drawable="@drawable/button_beat_box_normal" />
</selector>

```
结合两个的layer-list drawable
```

<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape
            android:shape="oval">

            <solid
                android:color="@color/red"/>
        </shape>
    </item>
    <item>
        <shape
            android:shape="oval">

            <stroke
                android:width="4dp"
                android:color="@color/dark_red"/>

        </shape>
    </item>
</layer-list>

```

**9-patch图片:**
特点:以.9.png结尾，以黑线描边来控制指定可以拉伸的区域。(上下左右只需指定一边即可，另一边默认相同)
用于控制图片的拉伸方式。通过使用9-path图片，Android知道图片的哪些部分可以拉伸，哪些不可以。
sdk/tools下有个draw9patch的工具。
![](/data/dokuwiki/android/pasted/20161012-222757.png?450x250)
指定图中圈中的用黑色线描绘的两个边可以拉伸，对边对应地点也一样可以。其余不能拉伸。效果如下：
![](/data/dokuwiki/android/pasted/20161012-222913.png?250x200)


**mimap图像：**
一般仅用于保存启动图标,因为存在这个目录的图片会保留所有版本的图片。
与drawable目录的区别：drawable不同版本的图片可以被删除，但是mimap目录下不同版本的图标一定会被保留。
(因为当针对设备类型定制apk时，可能需要删除一些用不到的版本的drawable图像，但是启动图标又想保留所有版本的，那就存到mimap目录里)


##  开发桌面程序获取已安装的应用列表 
```

		//构建隐式Intent
	   Intent startupIntent = new Intent(Intent.ACTION_MAIN);
        startupIntent.addCategory(Intent.CATEGORY_LAUNCHER);

		//通过PackageManager查询相关Intent的Activity信息
        PackageManager pm = getActivity().getPackageManager();
        List<ResolveInfo> activities = pm.queryIntentActivities(startupIntent, 0);
		//对这些信息根据应用名进行排序，ResolveInfo.loadLabel(pm)获取加载应用名。loadIcon(pm)获取加载图标，比较耗时应该在后台完成
        Collections.sort(activities, new Comparator<ResolveInfo>() {
            public int compare(ResolveInfo a, ResolveInfo b) {
                PackageManager pm = getActivity().getPackageManager();
                return String.CASE_INSENSITIVE_ORDER.compare(
                        a.loadLabel(pm).toString(),
                        b.loadLabel(pm).toString());
            }
        });

```
启动应用：
```

@Override
        public void onClick(View v) {
          	//通过ResolveInfo获取ActivityInfo进而获取包名，类名
            ActivityInfo activityInfo = mResolveInfo.activityInfo;

            Intent i = new Intent(Intent.ACTION_MAIN)
              			//设置全限定类名
                    .setClassName(activityInfo.applicationInfo.packageName,
                            activityInfo.name)
              		//新开一个任务来启动应用的主Activity，而不是在当前任务中。
                    .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

            startActivity(i);
        }

```


将应用作为桌面程序:
```

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
              	//HOME
                <category android:name="android.intent.category.HOME" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>

```


任务与进程:任务只包含Activity。
并发文档:FLAG_ACTIVITY_NEW_DOCUMENT或者activity documentLaunchMode的xml属性。（用于ACTION_SEND_*能够创建独立任务）

##  AsyncTask与HTTP编程 
权限:    <uses-permission android:name="android.permission.INTERNET" />
```

//通过Uri类，来构建一个符合Url编码规范的url
String urlSpec = Uri.parse("https://api.flickr.com/services/rest/")
                    .buildUpon()
                    .appendQueryParameter("method", "flickr.photos.getRecent")
                    .appendQueryParameter("api_key", API_KEY)
                    .appendQueryParameter("format", "json")
                    .appendQueryParameter("nojsoncallback", "1")
                    .appendQueryParameter("extras", "url_s")
                    .build().toString();

```
```

 public byte[] getUrlBytes(String urlSpec) throws IOException {
        URL url = new URL(urlSpec);
   		//打开连接
        HttpURLConnection connection = (HttpURLConnection)url.openConnection();
        try {
            ByteArrayOutputStream out = new ByteArrayOutputStream();
          	//获取输入流
            InputStream in = connection.getInputStream();
          		//确认是否响应码200 ok
            if (connection.getResponseCode() != HttpURLConnection.HTTP_OK) {
                throw new IOException(connection.getResponseMessage() +
                        ": with " +
                        urlSpec);
            }
            int bytesRead = 0;
            byte[] buffer = new byte[1024];
            while ((bytesRead = in.read(buffer)) > 0) {
                out.write(buffer, 0, bytesRead);
            }
            out.close();
            return out.toByteArray();
        } finally {
            connection.disconnect();
        }
    }

```
**JSON解析：**org.json ,推荐使用Gson,fastjson,jackson等
```

JSONObject jsonBody = new JSONObject(jsonString);
JSONObject photosJsonObject = jsonBody.getJSONObject("photos");
        JSONArray photoJsonArray = photosJsonObject.getJSONArray("photo");

        for (int i = 0; i < photoJsonArray.length(); i++) {
            JSONObject photoJsonObject = photoJsonArray.getJSONObject(i);

            GalleryItem item = new GalleryItem();
            item.setId(photoJsonObject.getString("id"));
            item.setCaption(photoJsonObject.getString("title"));
            
            if (!photoJsonObject.has("url_s")) {
                continue;
            }
            
            item.setUrl(photoJsonObject.getString("url_s"));
            items.add(item);
        }

```
**XML解析：**
```

 private static ArrayList<GalleryItem> parseItems(String xmlStr){
        ArrayList<GalleryItem> items=new ArrayList<>();
        try {
            XmlPullParserFactory factory=XmlPullParserFactory.newInstance();
            XmlPullParser parser=factory.newPullParser();
          
            parser.setInput(new StringReader(xmlStr));
            int eventType=parser.next();
            while(eventType!=XmlPullParser.END_DOCUMENT){
                if(eventType==XmlPullParser.START_TAG && XML_EL_PHOTO.equals(parser.getName())){
                    String id=parser.getAttributeValue(null,"id");
                    String caption=parser.getAttributeValue(null,"title");
                    String url=parser.getAttributeValue(null,EXTRA_SMALL_URL);
                    String owner=parser.getAttributeValue(null,"owner");

                    GalleryItem item=new GalleryItem();
                    item.setCaption(caption);
                    item.setId(id);
                    item.setUrl(url);
                    item.setOwner(owner);

                    items.add(item);
                }
                eventType=parser.next();
            }
        } catch (XmlPullParserException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return items;
    }

```


**AsyncTask**
```

/*三个泛型参数，第一个，doInBackground参数类型，
第二个onProgressUpdate()参数类型，用于结合publishProgress()来处理进度,
第三个参数,doInBackground返回值类型，onPostExecute接收值类型。
*/
private class FetchItemsTask extends AsyncTask<Void,Void,List<GalleryItem>> {
		//后台线程执行
        @Override
        protected List<GalleryItem> doInBackground(Void... params) {
          //publishProgress()
            return new FlickrFetchr().fetchItems();
        }
		//doInBackground执行完之后，主线程执行
        @Override
        protected void onPostExecute(List<GalleryItem> items) {
            mItems = items;
            setupAdapter();
        }
  	//进度处理，主线程执行
  		@Override
        protected void onProgressUpdate(){}

    }

```
` 注意：后台线程执行完之后，onPostExecute处理的view不一定存在了，因为fragment可以脱离activity而存在，所以在之后需要确认。 `，如下：
```

//由于这会用于线程回调，所以最好判断一下是否为null
	   //if(getActivity()==null|| gridView==null||)
        if(!isAdded()) return;  //isAdded()用于判断Fragment是否处于Activity之中。
        if(items!=null){
            mPhotoRecyclerView.setAdapter(new GalleryItemAdapter(getActivity(),items,downloader));
        }else{
            mPhotoRecyclerView.setAdapter(null);
        }

```

**AsyncTask的清理**：
AsyncTask.isCancelled()
AsyncTask.cancel(boolean)
###  AsyncTaskLoader 
AsyncTask加载数据遇到设备旋转时，可以通过Fragment.setRetainInstance(true)来解决，但这并不是万能药，
**在遇到fragment因内存吃紧而被销毁等意外情形时，Loader是一种更好的选择方案。Loader用于从某些数据源加载数据。(数据源可以是磁盘，数据库，网络，ContentProvider,甚至是另一进程)**
**AsyncTaskLoader**是Loader接口实现的抽象类。
为什么会选择Loader？因为遇到设备选择场景时，Loader处于**LoaderManager**管理之下，此时LoaderManager会帮我们管理Loader及其加载的数据，及时Loader的生命周期，而不受设备选择的影响。

###  RecyclerView网格列表分页 
使用RecyclerView,以及实现**RecyclerView.OnScrollListener**方法。

###  动态调整网格列 
**RecyclerView.addOnGlobalLayoutListener(ViewTreeObserver.OnGlobalLayoutListener)**

##  Looper、Handler、HandlerThread 
HandlerThread与AsyncThread的区别：前者适用于重复且长时间运行的任务，而后者适用于段时间且很少重复的工作。
为了按需加载使用Looper、Handler、HandlerThread机制
Looper管理者消息队列，使用消息队列的线程叫消息循环。而Handler负责收发Message，HandlerThread就是一个后台消息循环线程。
主(UI)线程也是一个消息循环，它也有自己的Looper和消息队列。
消息循环线程之间可以通过各自Handler发送和接收Message的形式来通信。

Message属性:
  * what-定义消息代码
  * obj-随消息一起发送的对象
  * target-处理消息的Handler

Handler:
发送消息时通过调用Handler.obtainMessage(..)从循环池获取可重用的Message对象。然后调用Message的sendToTarget()方法将其发送给目标Handler。然后Handler会将这个Message防止在Looper消息队列的尾部。
如handler.obtainMessage(MSG_DOWNLOAD,imgView).sendToTarget();
```

import android.os.Handler;
import android.os.HandlerThread;
import android.os.Message;
import android.util.Log;
import android.util.LruCache;

public class ThumbnailDownloader extends HandlerThread{
    private static final String TAG="ThumbnailDownloader";
    private ConcurrentHashMap<ImageView,String> reqMap=new ConcurrentHashMap<>();
    private Handler handler;
    private static final int MSG_DOWNLOAD=0x111;
    private Handler respHandler;
    private LruCache<String,Bitmap> cache=new LruCache<>(30);
    private Listener<ImageView> listener;
  	//定义一个回调接口
    public  interface Listener<ImageView>{
        void onThumbnailDownloaded(ImageView v,Bitmap thumbnail);
    }
    public ThumbnailDownloader(Handler respHandler,Listener<ImageView> listener) {
        super(TAG);
        this.respHandler=respHandler;
        this.listener=listener;
    }
	//onLooperPrepared
    @Override
    protected void onLooperPrepared() {
        super.onLooperPrepared();
      //在这里初始化当前线程的handler
        handler=new Handler(){
            @Override
            public void handleMessage(Message msg) {
                if(msg.what==MSG_DOWNLOAD){
                    ImageView imageView= (ImageView) msg.obj;
                    Log.i(TAG,"handle msg for url:"+reqMap.get(imageView));
                    handleRequest(imageView);
                }
            }
        };
    }

    public void setListener(Listener<ImageView> listener) {
        this.listener = listener;
    }

    public void queueThumbnail(ImageView imgView, String url){
        Log.i(TAG,"into queue:"+url);
        if(imgView==null||url==null) return;
        reqMap.put(imgView,url);
		//发送消息
        handler.obtainMessage(MSG_DOWNLOAD,imgView).sendToTarget();
    }
    private void handleRequest(final ImageView imageView){
        final String url=reqMap.get(imageView);
        if(url==null) return;
        try {
            final Bitmap bitmap;
            if(cache.get(url)==null){
                byte[] imgBytes=FlickrFectchr.getContent(url);
                BitmapFactory.Options options=new BitmapFactory.Options();
                options.outWidth=imageView.getMeasuredWidth();
                options.outHeight=imageView.getMeasuredHeight();
                bitmap= BitmapFactory.decodeByteArray(imgBytes,0,imgBytes.length,options);

                cache.put(url,bitmap);
            }else{
                bitmap=cache.get(url);
            }

            Log.d(TAG,url +" Bitmap created");
          	//使用主线程传过来的handler来post一段代码以在主线程中执行。
            respHandler.post(new Runnable() {
                @Override
                public void run() {
                    if(!url.equals(reqMap.get(imageView)) ){ //这一步很有必要，因为ImageView会被重用，此时有可能url已经变了。
                        return;
                    }
                    reqMap.remove(imageView);
                  //调用监听器回调
                    listener.onThumbnailDownloaded(imageView,bitmap);
                }
            });
        } catch (IOException e) {
            e.printStackTrace();
            Log.d(TAG,url+" bitmap create failed!",e);
        }

    }

    public LruCache<String, Bitmap> getCache() {
        return cache;
    }

    public void clearQueue(){
      	//清理工作，移除特定消息。
        handler.removeMessages(MSG_DOWNLOAD);
        reqMap.clear();
        cache.evictAll();
        Log.d(TAG,"clear queue");
    }
}


```

主线程调用queueThumbnail-》在HandlerThread发送消息-》HandlerThread中的Handler处理消息-》消息处理完毕调用主线程传过来的Handler post执行代码。

**主线程Fragment初始化HandlerThread的步骤:** ` 先调用start()然后紧接着调用getLooper() `
```

 downloader=new ThumbnailDownloader(new Handler(),null);
 downloader.start(); 
downloader.getLooper();

```
清理HandlerThread：
```

   @Override
    public void onDestroyView() {
        super.onDestroyView();
        downloader.clearQueue();//清理消息队列
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        downloader.quit();//退出消息循环
    }

```

图片加载库推荐Picasso

##  搜索SearchView 
```

<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@android:drawable/ic_menu_search"
        app:showAsAction="ifRoom"					app
        app:actionViewClass="android.support.v7.widget.SearchView"  app命名空间下的actionViewClass="android.support.v7.widget.SearchView
        />
    <item android:id="@+id/menu_search_clear"
        android:title="@string/menu_search_clear"
        android:icon="@android:drawable/ic_menu_close_clear_cancel"
        app:showAsAction="ifRoom"/>

</menu>

```
```

 setHasOptionsMenu(true);

```                                 
```

        MenuItem search=menu.findItem(R.id.menu_search);
        SearchView searchView= (SearchView) search.getActionView();
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                if(StringUtil.isEmpty(query)){
                    return false;
                }
                PreferenceManager.getDefaultSharedPreferences(getActivity()).edit().putString(QUERY_KEY,query).commit();
                updateItems();
                return true;
            }
            @Override
            public boolean onQueryTextChange(String newText) {
                return false;
            }
        });
	searchView.setOnSearchClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String query = QueryPreferences.getStoredQuery(getActivity());
                searchView.setQuery(query, false);
            }
        });

```

##  Service与Notification 
**Service能够接收Intent，需要在AndroidManifest中配置。**
` 普通Service是运行在UI线程中的，而IntentService是单独开启一个线程运行的。 `
Service的生命周期：
如果是startService(Intent)启动的服务：
onCreate()-》onStartCommand(Intent,标识符,启动ID)-》onDestroy()
onStartCommand返回值决定服务的类型Service.START_NOT_STICKY、START_REDELIVER_INTENT、START_STICKY

**服务的类型:**
  ***non-sticky服务:**
IntentService是一种non-sticky服务.START_NOT_STICKY、START_REDELIVER_INTENT.non-stiky服务在服务任务自己已完成任务时停止。通过调用**stopSelf**([最新启动id])
这里的START_NOT_STICKY与START_REDELIVER_INTENT的区别，系统需要在服务完成之前关闭它，则前者消亡就消亡了，后者会选择在资源不在吃紧时，尝试重启服务。

  * **sticky服务**:START_STICKY它会持续运行，直到外部组件调用Context.stopService(Intent)方法让它停止。适用于如音乐播放等。但是不推荐。
  * **bind服务**：bindService(Intent,ServiceConnection,int)绑定。分为本地绑定与远程绑定。推荐远程绑定场所应用，提供了调用其他进程服务的方法。

###  检测网络状态 
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

private boolean isNetworkAvailableAndConnected() {
  		//ConnectivityManager
        ConnectivityManager cm =
                (ConnectivityManager) getSystemService(CONNECTIVITY_SERVICE);
		//NetworkInfo
        boolean isNetworkAvailable = cm.getActiveNetworkInfo() != null;
        boolean isNetworkConnected = isNetworkAvailable &&
                cm.getActiveNetworkInfo().isConnected();

        return isNetworkConnected;
    }

```
```

import android.app.IntentService;
import android.app.Notification;
import android.app.PendingIntent;
import android.support.v4.app.NotificationCompat;
import android.support.v4.app.NotificationManagerCompat;

public class PollService extends IntentService {
    private static final String TAG = "PollService";
    private static final long POLL_INTERVAL = AlarmManager.INTERVAL_FIFTEEN_MINUTES;
    public static Intent newIntent(Context context) {
        return new Intent(context, PollService.class);
    }
    public static void setServiceAlarm(Context context, boolean isOn) {
        Intent i = PollService.newIntent(context);
      		//使用PendingIntent：参数context,请求码，intent,如何创建的标志。
        PendingIntent pi = PendingIntent.getService(
                context, 0, i, 0);
		//AlarmManager
        AlarmManager alarmManager = (AlarmManager)
                context.getSystemService(Context.ALARM_SERVICE);

        if (isOn) {
          	//通过AlarmManager来定时发送PendingIntent
            alarmManager.setInexactRepeating(AlarmManager.ELAPSED_REALTIME,
                    SystemClock.elapsedRealtime(), POLL_INTERVAL, pi);
        } else {
          	//记得清理。
            alarmManager.cancel(pi);
          	//
            pi.cancel();
        }
    }

    public static boolean isServiceAlarmOn(Context context) {
        Intent i = PollService.newIntent(context);
      	//PendingIntent.FLAG_NO_CREATE表示，如果不存在该PendingIntent就不要创建新的。
        PendingIntent pi = PendingIntent
                .getService(context, 0, i, PendingIntent.FLAG_NO_CREATE);
        return pi != null;
    }

    public PollService() {
        super(TAG);
    }

    @Override
    protected void onHandleIntent(Intent intent) {

        if (!isNetworkAvailableAndConnected()) {
            return;
        }
        。。。。
            Intent i = PhotoGalleryActivity.newIntent(this);
      		//PendingIntent
            PendingIntent pi = PendingIntent
                    .getActivity(this, 0, i, 0);
			//Notification通过NotificationCompat.Builder来构造
            Notification notification = new NotificationCompat.Builder(this)
                    .setTicker(resources.getString(R.string.new_pictures_title))
                    .setSmallIcon(android.R.drawable.ic_menu_report_image)
                    .setContentTitle(resources.getString(R.string.new_pictures_title))
                    .setContentText(resources.getString(R.string.new_pictures_text))
                    .setContentIntent(pi)
                    .setAutoCancel(true)//
                    .build();
			//使用NotificationManagerCompat而不是NotificationManager
            NotificationManagerCompat notificationManager = 
                    NotificationManagerCompat.from(this);
      		//notify发送通知
            notificationManager.notify(0, notification);
        }

       。。。。
    }
}


```
Android5.0之后可以使用**JobSchedular和JobService**、JobInfo来创建更为强大的任务调度执行机制。

sync adapter

##  BroadcastReceiver 
BroadcastReceiver可以处理Intent，需要在AndroidManifest里登记，并指定Intent-filter
BroadcastReceiver生命很短，而且运行在主线程上。
例如，获取开机信息的：
```

<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
<receiver android:name=".StartupReceiver" >
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>

```
```

public class StartupReceiver extends BroadcastReceiver{
    private static final String TAG = "StartupReceiver";

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.i(TAG, "Received broadcast intent: " + intent.getAction());

        boolean isOn = QueryPreferences.isAlarmOn(context);
        PollService.setServiceAlarm(context, isOn);
    }

}

```
**动态注册的broadcastreceiver**
```

public abstract class VisibleFragment extends Fragment {
    private static final String TAG = "VisibleFragment";

    private BroadcastReceiver mOnShowNotification = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            // If we receive this, we're visible, so cancel
            // the notification
            Log.i(TAG, "canceling notification");
            setResultCode(Activity.RESULT_CANCELED);
        }
    };

    @Override
    public void onStart() {
        super.onStart();
        IntentFilter filter = new IntentFilter(PollService.ACTION_SHOW_NOTIFICATION);
        getActivity().registerReceiver(mOnShowNotification, filter,
                PollService.PERM_PRIVATE, null);
    }

    @Override
    public void onStop() {
        super.onStop();
        getActivity().unregisterReceiver(mOnShowNotification);
    }
}


```
**使用私有权限：**
```

    <permission android:name="com.bignerdranch.android.photogallery.PRIVATE"
                android:protectionLevel="signature" /> //android:protectionLevel安全级别
   <uses-permission android:name="com.bignerdranch.android.photogallery.PRIVATE" />

<receiver android:name=".NotificationReceiver"
                  android:exported="false"> //内部使用
            <intent-filter
                android:priority="-999"> //优先级
                <action
                    android:name="com.bignerdranch.android.photogallery.SHOW_NOTIFICATION" />
            </intent-filter>
        </receiver>

```
**android:protectionLevel安全级别：**
normal、dangerous、signature、signatureOrSystem

**使用Ordered broadcast**
sendOrderedBroadcast
```

Intent i = new Intent(ACTION_SHOW_NOTIFICATION);
        i.putExtra(REQUEST_CODE, requestCode);
        i.putExtra(NOTIFICATION, notification);
        sendOrderedBroadcast(i, PERM_PRIVATE, null, null,
                Activity.RESULT_OK, null, null);

```
**然后其中的某个broadcast可以通过调用` setResultCode `来改变后续的结果码**
```

private BroadcastReceiver mOnShowNotification = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            // If we receive this, we're visible, so cancel
            // the notification
            Log.i(TAG, "canceling notification");
            setResultCode(Activity.RESULT_CANCELED);
        }
    };

```
由于NotificationReceiver的优先级低，所以在ordered broadcast的情况下，已经被前面的setResultCode(Activity.RESULT_CANCELED);修改了结果码了。
```

public class NotificationReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context c, Intent i) {
        Log.i(TAG, "received result: " + getResultCode());
        if (getResultCode() != Activity.RESULT_OK) {
            // A foreground activity cancelled the broadcast
            return;
        }

```

**应用内通知：本地广播(LocalBroadcastManager)、EventBus库、RxJava**


##  WebView 
```

       mProgressBar =
                (ProgressBar)v.findViewById(R.id.fragment_photo_page_progress_bar);
        mProgressBar.setMax(100); // WebChromeClient reports in range 0-100

        mWebView = (WebView) v.findViewById(R.id.fragment_photo_page_web_view);
		//启用JS支持
        mWebView.getSettings().setJavaScriptEnabled(true);
		//设置WebChromeClient
        mWebView.setWebChromeClient(new WebChromeClient() {
          	//进度
            public void onProgressChanged(WebView webView, int newProgress) {
                if (newProgress == 100) {
                    mProgressBar.setVisibility(View.GONE);
                } else {
                    mProgressBar.setVisibility(View.VISIBLE);
                    mProgressBar.setProgress(newProgress);
                }
            }
			//标题
            public void onReceivedTitle(WebView webView, String title) {
                AppCompatActivity activity = (AppCompatActivity) getActivity();
                activity.getSupportActionBar().setSubtitle(title);
            }
        });
		//WebViewClient
        mWebView.setWebViewClient(new WebViewClient() {
          	//是否由浏览器自己去加载点击的链接，false由浏览器去加载，true自己处理
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                return false;
            }
        });
		//loadUrl
        mWebView.loadUrl(mUri.toString());

```
**WebView设备旋转问题：**
由于WebView是属于视图，所以Fragment.setRetainInstance(true)无效。
而onSaveInstanceState()也是不行的，也会导致每次重新加载页面
解决方案：
```

        <activity
            android:name=".PhotoPageActivity"
            android:configChanges="keyboardHidden|orientation|screenSize" /> 让我们自己处理设备选择，而不是默认行为

```
**注入JavaScript对象**
```

public class WebAppInterface {
    Context mContext;
    /** Instantiate the interface and set the context */
    WebAppInterface(Context c) {
        mContext = c;
    }
    /** Show a toast from the web page */
    @JavascriptInterface  //注意Android4.2之后，只有以@JavascriptInterface注解的public方法才会暴露给JavaScript
    public void showToast(String toast) {
        Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show();
    }
}

```
```

webView.addJavascriptInterface(new WebAppInterface(this), "Android");

```
```

<input type="button" value="Say hello" onClick="showAndroidToast('Hello Android!')" />

<script type="text/javascript">
    function showAndroidToast(toast) {
        Android.showToast(toast);
    }
</script>

```
webview.canGoBack,webview.goBack, activity.onBackPressed


##  示例：DragAndDraw 
定制View，Canvas与Paint,onTouchEvent
```

public class BoxDrawingView extends View {
    public static final String TAG = "BoxDrawingView";
    private Box currentBox;
    private ArrayList<Box> boxList=new ArrayList<>();
    private Paint backgroundPaint;
    private Paint boxPaint;
    public BoxDrawingView(Context context) {
        this(context,null);
    }
    public BoxDrawingView(Context context, AttributeSet attrs) {
        super(context, attrs);
        backgroundPaint=new Paint();
        backgroundPaint.setColor(0xfff8efe0);
        boxPaint=new Paint();
        boxPaint.setColor(0x22ff0000);
    }
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        PointF cur=new PointF(event.getX(),event.getY());
        Log.i(TAG, "onTouchEvent: at X="+cur.x+" Y="+cur.y);
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                Log.i(TAG, "onTouchEvent: ACTION_DOWN");
                currentBox=new Box(cur);
                boxList.add(currentBox);
                break;
            case MotionEvent.ACTION_MOVE:
                Log.i(TAG, "onTouchEvent: ACTION_MOVE");
                if(currentBox!=null){
                    currentBox.setCurrent(cur);
                    invalidate();
                }
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                Log.i(TAG, "onTouchEvent: ACTION_CANCEL");
                currentBox=null;
                break;
            default:
                break;
        }
        return  true;

    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawPaint(backgroundPaint);
        for(Box box:boxList){
            float left=Math.min(box.getOrigin().x,box.getCurrent().x);
            float right=Math.max(box.getOrigin().x,box.getCurrent().x);
            float bottom=Math.max(box.getOrigin().y,box.getCurrent().y);
            float top=Math.min(box.getOrigin().y,box.getCurrent().y);

            canvas.drawRect(left,top,right,bottom,boxPaint);
        }
    }
}

```
##  属性动画 
略。
##  AsyncTaskLoader与CursorAdapter 
```

public abstract class DataLoader<D> extends AsyncTaskLoader<D> {
    private D data;
    public DataLoader(Context context) {
        super(context);
    }
    @Override
    protected void onStartLoading() {
        if(data!=null){
            deliverResult(data);
        }else{
            forceLoad();
        }
    }
    @Override
    public void deliverResult(D data) {
        this.data=data;
        if(isStarted())
            super.deliverResult(data);
    }
}

```
```

public class RunLoader extends DataLoader<Run> {
    private int id;
    public RunLoader(Context context,int id) {
        super(context);
        this.id=id;
    }

    @Override
    public Run loadInBackground() {
        return RunManager.getInstance(getContext()).getRun(id);
    }
}

```
```

public abstract class SQLiteCursorLoader extends AsyncTaskLoader<Cursor> {
    private Cursor cursor;
    public SQLiteCursorLoader(Context context) {
        super(context);
    }
    protected abstract Cursor loadCursor();
  
    @Override
    public Cursor loadInBackground(){
        Cursor cursor=loadCursor();
        if(cursor!=null){
            //提前加载cursor内容
            cursor.getCount();
        }
        return cursor;
    }
    @Override
    public void deliverResult(Cursor data) {
        Cursor old=cursor;
        cursor=data;
        if(isStarted()){
            super.deliverResult(data);
        }
        if(old!=null&&old!=cursor&&!old.isClosed()){
            old.close();
        }
    }
    @Override
    protected void onStartLoading() {
        if(cursor!=null){
            deliverResult(cursor);
        }
        if(takeContentChanged()||cursor==null){
            forceLoad();
        }
    }
    @Override
    protected void onStopLoading() {
        cancelLoad();
    }
    @Override
    public  void onCanceled(Cursor cursor) {
        if(cursor!=null && !cursor.isClosed()){
            cursor.close();
        }
    }
    @Override
    protected void onReset() {
        super.onReset();

        onStopLoading();

        if(cursor!=null && !cursor.isClosed()){
            cursor.close();
        }
        cursor=null;
    }
}

```
```

public class RunListCursorLoader extends SQLiteCursorLoader {

    public RunListCursorLoader(Context context) {
        super(context);
    }

    @Override
    protected Cursor loadCursor() {
        return RunManager.getInstance(getContext()).queryRuns();
    }
}

```
```

public class RunCursorAdapter extends CursorAdapter {
    private RunDatabaseHelper.RunCursor runCursor;
    public RunCursorAdapter(Context context, RunDatabaseHelper.RunCursor c) {
        super(context, c, 0);
        runCursor=c;
    }
    @Override
    public View newView(Context context, Cursor cursor, ViewGroup parent) {
        LayoutInflater inflater= (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        return inflater.inflate(android.R.layout.simple_list_item_1,parent,false);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor) {
        Run run=runCursor.getRun();
        TextView textView= (TextView) view;
        textView.setText(new String(run.getStartDate().toString()));

    }
}

```
##  GPS相关 
```

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

```
```

 @Override
       locationManager = (LocationManager) ctx.getSystemService(Context.LOCATION_SERVICE);
private PendingIntent getLocationPendingIntent(boolean shouldCreate) {
        Intent i = new Intent(ACTION_LOCATION);
        int flag = shouldCreate ? 0 : PendingIntent.FLAG_NO_CREATE;
        PendingIntent pi = PendingIntent.getService(ctx, 0, i, flag);
        return pi;
    }

    public void startLocationUpdates() {
        String provider = LocationManager.GPS_PROVIDER;

        PendingIntent pi = getLocationPendingIntent(true);
        if (ActivityCompat.checkSelfPermission(ctx, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED && ActivityCompat.checkSelfPermission(ctx, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            return;
        }

        //为避免用户第一次获取位置等待时间过长，我们先获取用户最近一次的GPS数据。
        Location lastLoc=locationManager.getLastKnownLocation(provider);
        if(lastLoc!=null){
            lastLoc.setTime(System.currentTimeMillis());
            broadcastLocation(lastLoc);
        }
        locationManager.requestLocationUpdates(provider, 1, 0, pi);

    }
    public void stopLocationUpdates(){
        PendingIntent pi=getLocationPendingIntent(false);
        if(pi!=null){
            locationManager.removeUpdates(pi);
            pi.cancel();
        }
    }
    private void broadcastLocation(Location loc){
        Intent i=new Intent(ACTION_LOCATION);
        i.putExtra(LocationManager.KEY_LOCATION_CHANGED,loc);
        ctx.sendBroadcast(i);
    }

    public boolean isTrackingRun(){
        PendingIntent pi=getLocationPendingIntent(false);
        return pi!=null;
    }

```


##  Android6.0与权限申请 
Android6.0之后除了需要在AndroidManifest当中声明权限之外，在代码中还需要额外申请。如：
```

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

```
```

if (ActivityCompat.checkSelfPermission(ctx, Manifest.permission.ACCESS_FINE_LOCATION) 
    != PackageManager.PERMISSION_GRANTED 
    && ActivityCompat.checkSelfPermission(ctx, Manifest.permission.ACCESS_COARSE_LOCATION) 
    != PackageManager.PERMISSION_GRANTED) {
            return;
}

```

##  MaterialDesign新View组件 
com.android.supoort.cardview-v7支持库中的CardView.
com.android.supoort.design:22.2.0库中的FloatingActionButton
com.android.supoort.design:22.2.0库中的Snackbar

CardView:http://blog.csdn.net/birdsaction/article/details/45197499/
![](/data/dokuwiki/android/pasted/20161015-131619.png)
FloatingActionButton:http://blog.csdn.net/lmj623565791/article/details/46678867
![](/data/dokuwiki/android/pasted/20161015-131703.png)
Snackbar:http://www.tuicool.com/articles/BfEbMvB http://blog.csdn.net/yangshangwei/article/details/50817996
![](/data/dokuwiki/android/pasted/20161015-131729.png)
