title: android保存键值对 

#  Android保存键值对 
如果想保存一个相对较小的key-values集合，可以使用 ` SharedPreferences ` API. SharedPreferences对象指向包含key-value对的文件，并且提供简单的读写方式。每个SharedPreferences文件均由框架管理，私人或共享均可使用。本课程将告诉你如何使用SharedPreferences API来存储和检索简单数值。
Android平台给我们提供了一个SharedPreferences类，它是一个轻量级的存储类，特别适合用于保存软件**配置参数**。使用SharedPreferences保存数据，其背后是用**xml**文件存放数据，
文件存放在` /data/data/<package name>/shared_prefs `目录下：
##  获取SharedPreferences 

您可以创建新的共享首选项文件，或者采取两种方式之一来调用现有文件。
` getSharedPreferences() ` 如果需要**多个确定名称的共享**首选项文件，且这些文件用第一个参数来指定。你可以从应用程序的任何`  Context `调用这一函数。
` getPreferences() ` 当你仅需要一个配置文件。由于这是` 你的Activity的唯一一个配置文件 `，所以不必提供名称。
例如下面的代码在一个片段内执行。它访问了由资源字符串R.string.preference_file_key标识的共享首选项文件，并且使用私密模式打开该文件，这样只有你的应用程序才能访问这一文件。
```

Context context = getActivity();
SharedPreferences sharedPref = context.getSharedPreferences(
        getString(R.string.preference_file_key), Context.MODE_PRIVATE);

```
当命名共享首选项文件时，你要使用应用程序**唯一可识别的名称**，比如"com.example.myapp.PREFERENCE_FILE_KEY"。
另外如果你只需要一个活动的共享首选项文件，就可以使用` getPreferences() `方式：
` SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE); `
##  Write to Shared Preferences 
要写入一个共享首选项文件，需要调用SharedPreferences上的` edit() `来创建` SharedPreferences.Editor `。
Pass the keys and values you want to write with methods such as putInt() and putString()。然后调用` commit（） `来保存更改，例如：
```

SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
SharedPreferences.Editor editor = sharedPref.edit();
editor.putInt(getString(R.string.saved_high_score), newHighScore);
editor.commit();//不要忘了调用commit()提交更改

```
##  Read from Shared Preferences 
从共享首选项文件中检测值，可以使用getInt() 、 getString()等调用方法来提供你想要的关键值，当键不存在时会返回任意默认值。例如：
```

SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
int defaultValue = getResources().getInteger(R.string.saved_high_score_default);
long highScore = sharedPref.getInt(getString(R.string.saved_high_score), defaultValue);

```
