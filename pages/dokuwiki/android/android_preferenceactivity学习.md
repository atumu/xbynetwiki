title: android_preferenceactivity学习 

#  Android PreferenceActivity学习 

官方参考文档地址:http://developer.android.com/guide/topics/ui/settings.html

##  PreferenceActivity与PreferenceFragment介绍 
 
PreferenceActivity 继承ListActivity 它是以一个列表的形式在展现内容，它最主要的特点是添加Preference可以让控件的状态持久化储存.
尤其是软件开发肯定会有一堆设置选项选项，每次进入Activity都去手动的去取储存的数据，这样代码会变得很复杂很麻烦。 这个时候Preference就出来了，它就是专门解决这些特殊的选项保存与读取的显示。用户每次操作事件它会及时的以键值对的形式记录在SharedPreferences中，Activity每次启动它会自动帮我们完成数据的读取以及UI的显示。
android开发中一共为我们提供了几个组件，分别是` DialogPreference,RingtonePreference,CheckBoxPreference组件、EditTextPreference组件、ListPreference组件、SwitchPreference组件，MultiSelectListPreference `
同时可以以  ` <PreferenceCategory android:title="@string/preference_attributes"> `形式进行组织分组。
![](/data/dokuwiki/android/pasted/20150605-104104.png)
我们还可以利用` PreferenceFragment `

首选项布局定义文件在res/xml/preferences.xml中。
##  Preference中使用Intent 
```

<Preference android:title="@string/prefs_web_page" >
    <intent android:action="android.intent.action.VIEW"
            android:data="http://www.example.com" />
</Preference>

```
android:action
The action to assign, as per the setAction() method.
android:data
The data to assign, as per the setData() method.
android:mimeType
The MIME type to assign, as per the setType() method.
android:targetClass
The class part of the component name, as per the setComponent() method.
android:targetPackage
The package part of the component name, as per the setComponent() method.
##  CheckBoxPreference组件 
CheckBoxPreference 选中为true 取消选中为false 它的值会以boolean的形式储存在` SharedPreferences `中。
```

<?xml version="1.0" encoding="utf-8"?> 
<PreferenceScreen 
  xmlns:android="http://schemas.android.com/apk/res/android"> 
    <PreferenceCategory android:title="CheckBoxPreference">   
    <CheckBoxPreference android:key="checkbox_0" 
        android:title="CheckBox_A" 
        android:summary="这是一个勾选框A" > 
    </CheckBoxPreference> 
      
    <CheckBoxPreference android:key="checkbox_1" 
        android:title="CheckBox_B" 
        android:summary="这是一个勾选框B" > 
    </CheckBoxPreference> 
    </PreferenceCategory> 
</PreferenceScreen>

``` 
![](/data/dokuwiki/android/pasted/20150605-101400.png)

注意 extends ` PreferenceActivity `并在onCreate中调用 ` addPreferencesFromResource() `;  
```

public class CheckBoxActivity extends PreferenceActivity {  
 
    Context mContext = null;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  
    // 从资源文件中添Preferences ，选择的值将会自动保存到SharePreferences  
    addPreferencesFromResource(R.xml.checkbox);  
      
    mContext = this;  
      
    //CheckBoxPreference组件  
    CheckBoxPreference mCheckbox0 = (CheckBoxPreference) findPreference("checkbox_0");  
    mCheckbox0.setOnPreferenceClickListener(new OnPreferenceClickListener() {  
          
        @Override  
        public boolean onPreferenceClick(Preference preference) {  
        //这里可以监听到这个CheckBox 的点击事件  
        return true;  
        }  
    });  
      
    mCheckbox0.setOnPreferenceChangeListener(new OnPreferenceChangeListener() {  
          
        @Override  
        public boolean onPreferenceChange(Preference arg0, Object newValue) {  
        //这里可以监听到checkBox中值是否改变了  
        //并且可以拿到新改变的值  
          Toast.makeText(mContext, "checkBox_0改变的值为" +  (Boolean)newValue, Toast.LENGTH_LONG).show();    
        return true;  
        }  
    });  
 
    CheckBoxPreference mCheckbox1 = (CheckBoxPreference) findPreference("checkbox_1");  
    mCheckbox1.setOnPreferenceClickListener(new OnPreferenceClickListener() {  
          
        @Override  
        public boolean onPreferenceClick(Preference preference) {  
        //这里可以监听到这个CheckBox 的点击事件  
        return true;  
        }  
    });  
      
    mCheckbox1.setOnPreferenceChangeListener(new OnPreferenceChangeListener() {  
          
        @Override  
        public boolean onPreferenceChange(Preference arg0, Object newValue) {  
        //这里可以监听到checkBox中值是否改变了  
        //并且可以拿到新改变的值  
          Toast.makeText(mContext, "checkBox_1改变的值为" +  (Boolean)newValue, Toast.LENGTH_LONG).show();    
        return true;  
        }  
    });  
      
    }  
}

```
##  EditTextPreference组件 
EditTextPreference 点击后会弹出一个输入框，输入的内容会以字符串的的形式储存在SharedPreferences中。
```

<?xml version="1.0" encoding="utf-8"?> 
<PreferenceScreen 
  xmlns:android="http://schemas.android.com/apk/res/android"> 
    <PreferenceCategory android:title="EditTextPreference">   
    <EditTextPreference android:key="edit_0" 
        android:title="输入信息_A" 
        android:summary="请输入您的信息" 
        android:defaultValue="请输入信息" 
        android:dialogTitle="输入框"> 
    </EditTextPreference> 
          
     <EditTextPreference android:key="edit_1" 
        android:title="输入信息_B" 
        android:summary="请输入您的信息" 
        android:defaultValue="请输入信息" 
        android:dialogTitle="输入框"> 
    </EditTextPreference> 
    </PreferenceCategory> 
</PreferenceScreen> 

```
![](/data/dokuwiki/android/pasted/20150605-101702.png)
```

public class EditTextActivity extends PreferenceActivity {  
 
    Context mContext = null;  
 
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  
    // 从资源文件中添Preferences ，选择的值将会自动保存到SharePreferences  
    addPreferencesFromResource(R.xml.edittext);  
 
    mContext = this;  
 
    // EditTextPreference组件  
    EditTextPreference mEditText = (EditTextPreference) findPreference("edit_0");  
      
    //设置dialog按钮信息  
    mEditText.setPositiveButtonText("确定");  
    mEditText.setNegativeButtonText("取消");  
      
    //设置按钮图标  
    mEditText.setDialogIcon(R.drawable.jay);  
    }  
 
    
} 

```
###  几个常用标签属性： 

android:key  选项的名称或键
android:title  选项的标题
android:summary  选项的简短摘要
android:entries  列表项的文本
android:entryValues  列表中每一项的值
android:dialogTitle  对话框标题
android:defalutValue  列表中选项的默认值
对于CheckBoxPreference还有两个特殊的属性
android:summaryOn  选项被选中时显示的摘要
android:summaryOff  选项未选中时显示的摘要
##  ListPreference组件 
在res/array中先写两个数组，一个用与list的显示内容，一个用户list的选中数值。
```

<?xml version="1.0" encoding="utf-8"?> 
<resources> 
 
<string-array name="auto_logout_time_key"> 
        <item>10 mins.</item> 
        <item>20 mins.</item> 
        <item>30 mins.</item> 
        <item>60 mins.</item> 
</string-array> 
 
<string-array name="auto_logout_time_value"> 
        <item>600000</item> 
        <item>1200000</item> 
        <item>1800000</item> 
        <item>3600000</item> 
</string-array> 
</resources> 

```
ListPreference点击后会弹出一个列表框，选中后会将选中的内容(上面数组中的值)会以字符串的的形式储存在SharedPreferences中。
![](/data/dokuwiki/android/pasted/20150605-102209.png)
```

<?xml version="1.0" encoding="utf-8"?> 
<PreferenceScreen 
  xmlns:android="http://schemas.android.com/apk/res/android"> 
  <PreferenceCategory android:title="ListPreference">     
        <ListPreference   
            android:key="list_0" 
            android:title="登录设置A"   
            android:dialogTitle="选择在线时间" 
            android:entries="@array/auto_logout_time_key" 
            android:entryValues="@array/auto_logout_time_value" > 
        </ListPreference> 
      
        <ListPreference   
            android:key="list_0" 
            android:title="登录设置A"   
            android:dialogTitle="选择在线时间" 
            android:entries="@array/auto_logout_time_key" 
            android:entryValues="@array/auto_logout_time_value" > 
        </ListPreference> 
    </PreferenceCategory> 
</PreferenceScreen> 

```
##  RingtonePreference组件 
RingtonePreference点击后会弹出一个系统铃声的列表框，选中后会将选中的内容(uri字符集)会以字符串的的形式储存在SharedPreferences中。
```

<?xml version="1.0" encoding="utf-8"?> 
<PreferenceScreen 
  xmlns:android="http://schemas.android.com/apk/res/android"> 
 <PreferenceCategory android:title="RingtonePreference">      
     <RingtonePreference    
         android:key="ringtone_0"    
         android:summary="选择系统铃声A"    
         android:title="铃声设置"    
         android:ringtoneType="all"    
         android:showSilent="true" ></RingtonePreference> 
      
    <RingtonePreference    
        android:key="ringtone_!"    
        android:summary="选择系统铃声B"    
        android:title="铃声设置"    
        android:ringtoneType="all"    
        android:showSilent="true" ></RingtonePreference> 
      
    </PreferenceCategory> 
</PreferenceScreen> 

```
` android:ringtoneType ` 系统一共提供了4中响铃模式的类型分别为** 铃声(ringtone) 通知( notification) 警告(alarm) 全部(all)** 
![](/data/dokuwiki/android/pasted/20150605-102653.png)
##  读取数据 
在PreferenceActivity中可以用下面这种方式拿到SharedPreferences中储存的数值，通过` PreferenceManager.getDefaultSharedPreferences(this) ` 方法拿到控件默认储存的` sharedPreferences `对象。
```

SharedPreferences prefs =PreferenceManager.getDefaultSharedPreferences(this) ;  
   boolean something = prefs.getBoolean("something",false);

``` 
##  Using Preference Headers 
![](/data/dokuwiki/android/pasted/20150605-110618.png)![](/data/dokuwiki/android/pasted/20150605-110630.png)
###  Creating the headers file 
```

<?xml version="1.0" encoding="utf-8"?>
<preference-headers xmlns:android="http://schemas.android.com/apk/res/android">
    <header 
        android:fragment="com.example.prefs.SettingsActivity$SettingsFragmentOne"
        android:title="@string/prefs_category_one"
        android:summary="@string/prefs_summ_category_one" />
    <header 
        android:fragment="com.example.prefs.SettingsActivity$SettingsFragmentTwo"
        android:title="@string/prefs_category_two"
        android:summary="@string/prefs_summ_category_two" >
        <!-- key/value pairs can be included as arguments for the fragment. extra元素定义了传达给PreferenceFragment的参数-->
        <extra android:name="someKey" android:value="someHeaderValue" />
    </header>
</preference-headers>

```
```

public static class SettingsFragment extends PreferenceFragment {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        String settings = getArguments().getString("settings");
        if ("notifications".equals(settings)) {
            addPreferencesFromResource(R.xml.settings_wifi);
        } else if ("sync".equals(settings)) {
            addPreferencesFromResource(R.xml.settings_sync);
        }
    }
}

```
###  Displaying the headers 
```

public class SettingsActivity extends PreferenceActivity {
    @Override
    public void onBuildHeaders(List<Header> target) {
        loadHeadersFromResource(R.xml.preference_headers, target);
    }
}

```
##  Listening for preference changes 
```

public class SettingsActivity extends PreferenceActivity
                              implements OnSharedPreferenceChangeListener {
    public static final String KEY_PREF_SYNC_CONN = "pref_syncConnectionType";
    ...

    public void onSharedPreferenceChanged(SharedPreferences sharedPreferences,
        String key) {
        if (key.equals(KEY_PREF_SYNC_CONN)) {
            Preference connectionPref = findPreference(key);
            // Set summary to be the user-description for the selected value
            connectionPref.setSummary(sharedPreferences.getString(key, ""));
        }
    }
}


@Override
protected void onResume() {
    super.onResume();
    getPreferenceScreen().getSharedPreferences()
            .registerOnSharedPreferenceChangeListener(this);
}

@Override
protected void onPause() {
    super.onPause();
    getPreferenceScreen().getSharedPreferences()
            .unregisterOnSharedPreferenceChangeListener(this);
}

```
在其他Activity中监听变化.
```

SharedPreferences.OnSharedPreferenceChangeListener listener =
    new SharedPreferences.OnSharedPreferenceChangeListener() {
  public void onSharedPreferenceChanged(SharedPreferences prefs, String key) {
    // listener implementation
  }
};
prefs.registerOnSharedPreferenceChangeListener(listener);

```

##  实例： 
```

<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
    <CheckBoxPreference
        android:key="touchCheck"
        android:title="@string/checkbox_title"
        android:summary="@string/checkbox_description"
        android:defaultValue="true" />
    <!-- NOTE: EditTextPreference accepts EditText attributes. -->
    <!-- NOTE: EditTextPreference's summary should be set to its value by the activity code. -->
    <EditTextPreference
        android:key="editNums"
        android:title="@string/edit_title"
        android:summary="@string/edit_hint"
        android:numeric="integer"
        android:defaultValue="20"
        android:selectAllOnFocus="true"
        android:inputType="number"
        android:singleLine="true"
        android:maxLines="1" />
</PreferenceScreen>

```
```

public class MyPreferenceActivity extends PreferenceActivity {
    public static final String EDITNUMS_KEY="editNums";
    @Override
    protected void onCreate(Bundle bundle){
        super.onCreate(bundle);
        addPreferencesFromResource(R.xml.pref);
        final Preference editText= getPreferenceScreen().findPreference(EDITNUMS_KEY);
        editText.setOnPreferenceChangeListener(new Preference.OnPreferenceChangeListener() {
            @Override
            public boolean onPreferenceChange(Preference preference, Object newValue) {
                if(newValue!=null&&newValue.toString().trim().length()>0){
                    int value=Integer.valueOf((String)newValue);
                    if(value>=10&&value<=30){
                        return true;
                    }
                }
                Toast.makeText(MyPreferenceActivity.this,"输入错误",Toast.LENGTH_SHORT).show();
                return false;
            }
        });
    }
}

```
```

SharedPreferences prefs =PreferenceManager.getDefaultSharedPreferences(this) ;  
   boolean something = prefs.getBoolean("something",false);

``` 
##  自定义Preference 
关于自定义更多部分请参考官方文档：http://developer.android.com/guide/topics/ui/settings.html#Custom
在加载了preferences.xml的PreferenceActivity中， a top-level preference是一个PreferenceScreen，可用getPreferenceScreen()获取。PreferenceScreen和PreferenceCategory继承自PreferenceGroup，它们可以包含一个或多个PreferenceScreen，PreferenceCategory或者是具体的preference（如EditTextPreference、CheckBoxPreference）。由于PreferenceScreen，PreferenceCategory，EditTextPreference等都是继承自Preference，因此可以通过` setLayoutResource() `方法设置自己的布局样式。
下面介绍加顶部布局，其实也是添加加一个preference，通过preferenceScreen的addPreference添加。首先自定义一个PreferenceHead，布局中有一个返回按钮。

**自定义的prefernece代码** ：
` 继承Preference `，
覆盖` OnBindview `，
` 覆盖构造器 `Preference(Context context, AttributeSet attrs) ，Preference(Context context, AttributeSet attrs, int defStyle)
```

public class PreferenceHead extends Preference {
 
    private OnClickListener onBackButtonClickListener;
 
   public PreferenceHead(Context context, AttributeSet attrs, int defStyle) {  
        super(context, attrs, defStyle);  
        setLayoutResource(R.layout.preference);       
    }  
  
    public PreferenceHead(Context context, AttributeSet attrs) {  
        this(context, attrs, 0); //becare this states  
    } 
    @Override
    protected void onBindView(View view) {
        super.onBindView(view);
        Button btBack = (Button) view.findViewById(R.id.back);
        btBack.setOnClickListener(new OnClickListener() {
 
            @Override
            public void onClick(View v) {
                if (onBackButtonClickListener != null) {
                    onBackButtonClickListener.onClick(v);
                }
            }
        });
    }
 
    public void setOnBackButtonClickListener(OnClickListener onClickListener) {
        this.onBackButtonClickListener = onClickListener;
    }
}

```
**res/layout/preference_head.xml**:
```

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="60.0dip"
    android:background="#8000FF00"
    android:gravity="center_vertical" >
 
    <Button
        android:id="@+id/back"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="10.0dip"
        android:text="返回" />
 
</LinearLayout>

```
然后在代码中实现
```

ph.setOnBackButtonClickListener(new OnClickListener() {
 
    @Override
    public void onClick(View v) {
        finish();
    }
});

```
这样就完成了一个具有返回按钮的顶部布局的 PreferenceActivity，效果图如下
![](/data/dokuwiki/android/pasted/20150605-105319.png)

参考：
http://my.oschina.net/eclipse88/blog/52316
http://blog.csdn.net/wed110/article/details/29592735
http://xys289187120.blog.51cto.com/3361352/656784