title: android手电筒小程序实现代码 

#  android手电筒小程序实现代码 

涉及到Camera类的使用。
```

public class MainActivity extends ActionBarActivity {
    public static final String TAG="MainActivity";
    private Button torchButton;
    private boolean isON=false;
    private Camera cam;
    private Camera.Parameters camParams;
    public static final int FLASH_TORCH_NOT_SUPPORTTED=0;
    public static final int FLASH_NOT_SUPPORTED=1;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        openTorch();
    }

    public  void openTorch(){
        if(!(getPackageManager().hasSystemFeature(PackageManager.FEATURE_CAMERA))){
            Log.d(TAG, "Camera not supported");
            finish();
        }
        if(getPackageManager().hasSystemFeature(PackageManager.FEATURE_CAMERA_FLASH)){
            torchButton=(Button)findViewById(R.id.torchButton);
            torchButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                  
                    try {
                        if(cam==null)
                            cam = Camera.open();
                        boolean isNull=(cam==null);
                       
                        camParams = cam.getParameters();
                        
                        List<String> flashModes = camParams.getSupportedFlashModes();
                        if (!isON) {
                            isON = true;
                            torchButton.setText(R.string.torch_off);
                            if(flashModes.contains(Camera.Parameters.FLASH_MODE_TORCH)){
                                camParams.setFlashMode(Camera.Parameters.FLASH_MODE_TORCH);
                            }else{
                                showDialog(MainActivity.this,FLASH_TORCH_NOT_SUPPORTTED);
                            }
                        } else {
                            isON = false;
                            torchButton.setText(R.string.torch_on);
                            camParams.setFlashMode(Camera.Parameters.FLASH_MODE_OFF);
                        }
                        cam.setParameters(camParams);
                        cam.startPreview();
                    }catch(Exception e) {
                        e.printStackTrace();
                        cam.stopPreview();
                        cam.release();

                    }
                }
            });
        }else{
            showDialog(MainActivity.this,FLASH_NOT_SUPPORTED);
        }

    }
    public void showDialog(Context context,int dialogId){
        switch(dialogId){
            case FLASH_NOT_SUPPORTED:
                dialogBuild(context,"sorry ,don't support flash");
            case FLASH_TORCH_NOT_SUPPORTTED:
                dialogBuild(context,"sorry,don't support flash mode");
        }
    }
    public void dialogBuild(Context context ,String msg ){
        new AlertDialog.Builder(context).setMessage(msg).setCancelable(false)
                .setNeutralButton("Close",new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        finish();
                    }
                }).show();
    }
    @Override
    protected void onDestroy(){
        super.onDestroy();
        if(cam!=null){
            cam.release();
        }
    }

    /**
     * 由于在AndroidManifest.xml中设置了android:configChanges="orientation",故而在屏幕转向变化时，该Activity不会重启，只会调用下面这个钩子方法。
     * @param newConfig
     */
    @Override
    public void onConfigurationChanged(Configuration newConfig){
        super.onConfigurationChanged(newConfig);
        if(newConfig.orientation== Configuration.ORIENTATION_LANDSCAPE){
            //设置为横屏的布局文件
        }else{
            //设置为竖屏的布局文件
        }
    }
}

```