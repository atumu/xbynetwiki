title: android_wifi密码查看器实现思路 

#  android_wifi密码查看器实现思路 

概述

最近出了一个wifi万能钥匙很火，但是用wifi万能钥匙连接上wifi我们是看不到密码，假如我想给平板连接，这是我们只能用手机3G网络建立热点然后让平板上的wifi万能钥匙进行连接，这样显然很麻烦。假如手机端有个wifi密码查看器就好了。wifi密码查看器配合wifi万能钥匙使用是很不错的想法。
其实wifi密码查看器就是查看已连接过wifi的密码。因为已连接过wifi的密码都会集中保存在一个系统文件中。

实现

前提：设备获取过root权限。
wifi密码的保存文件为/data/misc/wifi/wpa_supplicant.conf
我们只需执行linux shell命令获取里面的信息再进行解析即可。
```

public class MainActivity extends ListActivity {
    public static final String TAG="MainActivity";
    private boolean isRoot=false;
    private ArrayList<String> strlist= new ArrayList<String>();
    private DataOutputStream dos;
    private DataInputStream dis;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
//        setContentView(R.layout.activity_main);
        setTitle(R.string.title);
        try {
            Process p=Runtime.getRuntime().exec("su");
            Log.d(TAG,"root");
            dos = new DataOutputStream(p.getOutputStream());
            dis = new DataInputStream(p.getInputStream());
            dos.writeBytes("cat /data/misc/wifi/wpa_supplicant.conf" + "\n");
            dos.flush();
            dos.writeBytes("exit\n");
            dos.flush();
            String line;
            while((line= dis.readLine())!=null){
                String str="";
                if(line.trim().startsWith("ssid")){
                    str=line.trim();
                    line= dis.readLine();
                    if(line.trim().startsWith("psk")){
                        str+="\n"+line.trim();
                        strlist.add(str);
                    }
                    Log.d(TAG, str);
                }

            }
            Collections.reverse(strlist);
            p.waitFor();

        } catch (InterruptedException e) {
            e.printStackTrace();

        } catch (IOException e) {
            Log.d(TAG,"not root");
            new AlertDialog.Builder(this).setMessage("对不起，您的设备没有获取root权限").setNeutralButton("确定",null).show();

            e.printStackTrace();
            finish();
        }finally{
            if(dos!=null){
                try {
                    dos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(dis!=null){
                try {
                    dis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        if(strlist.size()>0){
            ArrayAdapter<String> adapter=new ArrayAdapter<String>(this,R.layout.activity_main,strlist);
            setListAdapter(adapter);
        }
    }
@Override
    public boolean onCreateOptionsMenu(Menu menu){
    MenuInflater mflater=getMenuInflater();
    mflater.inflate(R.menu.menu_main,menu);
    return true;
}
    @Override
    public boolean onOptionsItemSelected(MenuItem item){
        if(item.getItemId()==R.id.about){
            AboutBox.show(this);
        }
        return true;
    }

}

```

