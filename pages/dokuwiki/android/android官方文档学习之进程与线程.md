title: android官方文档学习之进程与线程 

#  android官方文档学习之进程与线程 
进程的生命期
** Android系统会尽量维持一个进程的生命，直到最终需要为新的更重要的进程腾出内存空间**。为了决定哪个进程该终止，系统会跟据运行于进程内的组件的和组件的状态把进程置于不同的重要性等级。当需要系统资源时，重要性等级越低的先被淘汰。
 重要性等级被分为５个档。下面列出了不同类型的进程的重要性等级(第一个进程类型是最重要的，也是最后才会被终止的）
１前台进程
２可见进程
３服务进程
4 后台进程
５空进程:一个没有任何active组件的进程。保留这类进程的唯一理由是**高速缓存**，这样可以提高下一次一个组件要运行它时的**启动速度**。系统经常为了平衡在进程高速缓存和底层的内核高速缓存之间的整体系统资源而终止它们。

线程：
单一线程规则：
  * 不要阻塞界面线程
  * 不要在界面线程之外操作界面。

**Android提供了很多从其它线程来操作界面的方法**。下面是可用的方法：
  * Activity.runOnUiThread(Runnable)
  * View/Handler.post(Runnable)
  * View/Handler.postDelayed(Runnable,long)

##  使用-AsyncTask 
```

 public void onClick(View v) {
    new DownloadImageTask().execute(&lt;nowiki&gt;"http://example.com/image.png"&lt;/nowiki&gt;);
 }
 private class DownloadImageTask extends AsyncTask<String, Integer, Bitmap> {
    /** 该方法运行在后台线程中。参数来自AsyncTask.execute()方法,
         *这里将主要负责执行那些很耗时的后台处理工作。*/
    protected Bitmap doInBackground(String... urls) {
        return loadImageFromNetwork(urls[0]);
    }
 
    /** 后台的计算结果将通过该方法传递到UI 线程，并且在界面上展示给用户*/
    protected void onPostExecute(Bitmap result) {
        mImageView.setImageBitmap(result);
    }
 }

```
 因为AsyncTask将工作分成了两部分，UI线程和工作线程都做自己应该做的那部分任务，因此我们就可以简单的实现一个安全的UI更新任务了。
` AsyncTask `定义了**三种泛型类型 Params，Progress和Result**。Params是启动任务执行的输入参数，比如HTTP请求的URL。Progress是后台任务执行的百分比。Result是后台执行任务最终返回的结果，比如String,Integer等。在特定场合下，并不是所有类型都被使用，如果没有被使用，可以用java.lang.Void类型代替。
` doInBackground() `会在工作线程中自动执行。
` onPreExecute(), onPostExecute(), and onProgressUpdate() `这三个方法都是在UI线程中调用。
` doInBackground() `方法返回的值会传递给` onPostExecute() `方法。你可以` 在doInBackground()方法中随时调用publishProgress()方法 `以此在UI线程中更新执行进度。
` onProgressUpdate(Integer... progresses) `：在调用` publishProgress(Integer... progresses) `时，此方法被执行，直接将进度信息更新到UI组件上。

你可以随时在任何线程中取消任务。

**在使用的时候，有几点需要格外注意：**
1.**异步任务的实例必须在UI线程中创建。**
2.**execute(Params... params)方法必须在UI线程中调用。**
3.不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Integer... progresses)，onPostExecute(Result result)这几个方法。
4.不能在doInBackground(Params... params)中更改UI组件的信息。
5.**一个任务实例只能执行一次，如果执行第二次将会抛出异常**。
##  AsyncTask使用示例 
```

public class MainActivity extends Activity {  
      
    private static final String TAG = "ASYNC_TASK";  
      
    private Button execute;  
    private Button cancel;  
    private ProgressBar progressBar;  
    private TextView textView;  
      
    private MyTask mTask;  
      
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
          
        execute = (Button) findViewById(R.id.execute);  
        execute.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View v) {  
                //注意每次需new一个实例,新建的任务只能执行一次,否则会出现异常  
                mTask = new MyTask();  
                mTask.execute("http://www.baidu.com");  
                  
                execute.setEnabled(false);  
                cancel.setEnabled(true);  
            }  
        });  
        cancel = (Button) findViewById(R.id.cancel);  
        cancel.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View v) {  
                //取消一个正在执行的任务,onCancelled方法将会被调用  
                mTask.cancel(true);  
            }  
        });  
        progressBar = (ProgressBar) findViewById(R.id.progress_bar);  
        textView = (TextView) findViewById(R.id.text_view);  
          
    }  
      
    private class MyTask extends AsyncTask<String, Integer, String> {  
        //onPreExecute方法用于在执行后台任务前做一些UI操作  
        @Override  
        protected void onPreExecute() {  
            Log.i(TAG, "onPreExecute() called");  
            textView.setText("loading...");  
        }  
          
        //doInBackground方法内部执行后台任务,不可在此方法内修改UI  
        @Override  
        protected String doInBackground(String... params) {  
            Log.i(TAG, "doInBackground(Params... params) called");  
            try {  
                HttpClient client = new DefaultHttpClient();  
                HttpGet get = new HttpGet(params[0]);  
                HttpResponse response = client.execute(get);  
                if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {  
                    HttpEntity entity = response.getEntity();  
                    InputStream is = entity.getContent();  
                    long total = entity.getContentLength();  
                    ByteArrayOutputStream baos = new ByteArrayOutputStream();  
                    byte[] buf = new byte[1024];  
                    int count = 0;  
                    int length = -1;  
                    while ((length = is.read(buf)) != -1) {  
                        baos.write(buf, 0, length);  
                        count += length;  
                        //调用publishProgress公布进度,最后onProgressUpdate方法将被执行  
                        publishProgress((int) ((count / (float) total) * 100));  
                        //为了演示进度,休眠500毫秒  
                        Thread.sleep(500);  
                    }  
                    return new String(baos.toByteArray(), "gb2312");  
                }  
            } catch (Exception e) {  
                Log.e(TAG, e.getMessage());  
            }  
            return null;  
        }  
          
        //onProgressUpdate方法用于更新进度信息  
        @Override  
        protected void onProgressUpdate(Integer... progresses) {  
            Log.i(TAG, "onProgressUpdate(Progress... progresses) called");  
            progressBar.setProgress(progresses[0]);  
            textView.setText("loading..." + progresses[0] + "%");  
        }  
          
        //onPostExecute方法用于在执行完后台任务后更新UI,显示结果  
        @Override  
        protected void onPostExecute(String result) {  
            Log.i(TAG, "onPostExecute(Result result) called");  
            textView.setText(result);  
              
            execute.setEnabled(true);  
            cancel.setEnabled(false);  
        }  
          
        //onCancelled方法用于在取消执行中的任务时更改UI  
        @Override  
        protected void onCancelled() {  
            Log.i(TAG, "onCancelled() called");  
            textView.setText("cancelled");  
            progressBar.setProgress(0);  
              
            execute.setEnabled(true);  
            cancel.setEnabled(false);  
        }  
    }  
}  

```
线程安全的方法

进程间通信
