title: android内部数据存储文件 

#  android内部数据存储文件 
内部存储：
  * 始终可用。
  * 保存的文件只能用于默认应用程序。
  * 当用户卸载应用程序时，系统会从内部存储删除应用程序所有文件。
  * 当要确保无论用户还是其他应用程序均可访问文件时，内部存储无疑是最好的选择。
内部存储位置` /data/data/<pkg_name>/ `
**应用读写文件最便利的方式是使用Context类提供的IO方法。**
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150605-051303.png)
注意对文件的读写方法进行封装。以保持独立性。推荐在` onPause() `方法中进行文件的保持。
在内部存储保存文件时，你可以调用两种方法之一来获取相应的目录文件：
` getFilesDir() ` （` /data/data/<pkg_name>/files `）返回表示应用程序内部目录的文件
` getCacheDir() ` （` /data/data/<pkg_name>/cache `）返回表示应用程序临时缓存文件的内部目录的文件。一旦不再需要要确保删除每个文件。确保删除不再需要的文件并且设置合理大小的内存总量，比如1MB。如果系统存储开始运行缓慢，它可能会不经警告而删除缓存文件。
` getDir() `,(` /data/data/<pkg_name>/ `)
 ` openFileOutput() `，(` /data/data/<pkg_name>/ `)
` fileList() ` ,(` /data/data/<pkg_name>/files `)

要在这些目录中创建一个新文件可以使用 File()函数，用来传递由上述指定内部存储目录的方法之一所提供的文件。例如：
` File file = new File(context.getFilesDir(), filename); `
或者你可以调用 ` openFileOutput() `，(` /data/data/<pkg_name>/ `)以便获得能写入内部目录文件的FileOutputStream。例如下面展示了如何给文件编写文本：
```

String filename = "myfile";
String string = "Hello world!";
FileOutputStream outputStream;
 
try {
  outputStream = openFileOutput(filename, Context.MODE_PRIVATE);// /data/data/<pkg_name>/filename
  outputStream.write(string.getBytes());
  outputStream.close();
} catch (Exception e) {
  e.printStackTrace();
}

```
openFileOutput()方法的第二参数用于指定操作模式，有四种模式，分别为：
##  Context中的四种文件操作模式 
  * Context.MODE_PRIVATE：为默认操作模式，代表该文件是**私有**数据，只能被应用本身访问，在该模式下，写入的内容会**覆盖**原文件的内容，如果想把新写入的内容追加到原文件中。可以使用Context.MODE_APPEND
  * Context.MODE_APPEND：模式会检查文件是否存在，存在就往文件**追加**内容，否则就创建新文件。
  * Context.MODE_WORLD_READABLE和Context.MODE_WORLD_WRITEABLE用来**控制其他应用是否有权限读写**该文件。MODE_WORLD_READABLE：表示当前文件**可以被其他应用读取**；MODE_WORLD_WRITEABLE：表示当前文件可以被其他应用写入。
如果需要缓存文件，就应该使用` createTempFile() `。例如从URL中提取文件名并且在应用程序内部缓存目录创建文件，方法如下：
```

public File getTempFile(Context context, String url) {
    File file;
    try {
        String fileName = Uri.parse(url).getLastPathSegment();
        file = File.createTempFile(fileName, null, context.getCacheDir());
    catch (IOException e) {
        // Error while creating file
    }
    return file;
}

```
##  删除文件 
如果文件是用内部存储的方式来保存的，你也可以调用` deleteFile() `来请求` Context `定位和删除文件：
` myContext.deleteFile(fileName); `
注意：用户**卸载**应用程序时，安卓系统会删除以下内容：
  * 内部存储的所有文件。
  * 使用 ` getExternalFilesDir() `外部存储的所有文件。
然而你要手动删除` getCacheDir() ` 定期创建的所有缓存文件，同时定期删除其他不需要的文件。
##  实例：以JSON形式保存数据 
```

public class CriminalIntentJSONSerializer {

    private Context mContext;
    private String mFilename;

    public CriminalIntentJSONSerializer(Context c, String f) {
        mContext = c;
        mFilename = f;
    }

    public ArrayList<Crime> loadCrimes() throws IOException, JSONException {
        ArrayList<Crime> crimes = new ArrayList<Crime>();
        BufferedReader reader = null;
        try {
            // open and read the file into a StringBuilder
            InputStream in = mContext.openFileInput(mFilename);
            reader = new BufferedReader(new InputStreamReader(in));
            StringBuilder jsonString = new StringBuilder();
            String line = null;
            while ((line = reader.readLine()) != null) {
                // line breaks are omitted and irrelevant
                jsonString.append(line);
            }
            // parse the JSON using JSONTokener
            JSONArray array = (JSONArray) new JSONTokener(jsonString.toString()).nextValue();
            // build the array of crimes from JSONObjects
            for (int i = 0; i < array.length(); i++) {
                crimes.add(new Crime(array.getJSONObject(i)));
            }
        } catch (FileNotFoundException e) {
            // we will ignore this one, since it happens when we start fresh
        } finally {
            if (reader != null)
                reader.close();
}
        return crimes;
    }

    public void saveCrimes(ArrayList<Crime> crimes) throws JSONException, IOException {
        // build an array in JSON
        JSONArray array = new JSONArray();
        for (Crime c : crimes)
            array.put(c.toJSON());

        // write the file to disk
        Writer writer = null;
        try {
            OutputStream out = mContext.openFileOutput(mFilename, Context.MODE_PRIVATE);
            writer = new OutputStreamWriter(out);
            writer.write(array.toString());
        } finally {
            if (writer != null)
                writer.close();
        }
    }
}

```
```

public class Crime {

    private static final String JSON_ID = "id";
    private static final String JSON_TITLE = "title";
    private static final String JSON_DATE = "date";
    private static final String JSON_SOLVED = "solved";
    
    private UUID mId;
    private String mTitle;
    private Date mDate;
    private boolean mSolved;
    
    public Crime() {
        mId = UUID.randomUUID();
        mDate = new Date();
    }

    public Crime(JSONObject json) throws JSONException {
        mId = UUID.fromString(json.getString(JSON_ID));
        mTitle = json.getString(JSON_TITLE);
        mSolved = json.getBoolean(JSON_SOLVED);
        mDate = new Date(json.getLong(JSON_DATE));
    }

    public JSONObject toJSON() throws JSONException {
        JSONObject json = new JSONObject();
        json.put(JSON_ID, mId.toString());
        json.put(JSON_TITLE, mTitle);
        json.put(JSON_DATE, mDate.getTime());
        json.put(JSON_SOLVED, mSolved);
        return json;
    }

```
```

public class CrimeLab {
    private static final String TAG = "CrimeLab";
    private static final String FILENAME = "crimes.json";

    private ArrayList<Crime> mCrimes;
    private CriminalIntentJSONSerializer mSerializer;

    private static CrimeLab sCrimeLab;
    private Context mAppContext;

    private CrimeLab(Context appContext) {
        mAppContext = appContext;
        mSerializer = new CriminalIntentJSONSerializer(mAppContext, FILENAME);

        try {
            mCrimes = mSerializer.loadCrimes();
        } catch (Exception e) {
            mCrimes = new ArrayList<Crime>();
            Log.e(TAG, "Error loading crimes: ", e);
        }
    }

    public static CrimeLab get(Context c) {
        if (sCrimeLab == null) {
            sCrimeLab = new CrimeLab(c.getApplicationContext());
        }
        return sCrimeLab;
    }

    public Crime getCrime(UUID id) {
        for (Crime c : mCrimes) {
            if (c.getId().equals(id))
                return c;
        }
        return null;
    }
    
    public void addCrime(Crime c) {
        mCrimes.add(c);
        saveCrimes();
    }

    public ArrayList<Crime> getCrimes() {
        return mCrimes;
    }

    public boolean saveCrimes() {
        try {
            mSerializer.saveCrimes(mCrimes);
            Log.d(TAG, "crimes saved to file");
            return true;
        } catch (Exception e) {
            Log.e(TAG, "Error saving crimes: " + e);
            return false;
        }
    }
}

```
