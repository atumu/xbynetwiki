title: 自定义adapter 

#  Android自定义Adapter 
##  自定义ArrayAdapter: 
只需要覆盖public View getView(int position,View convertView, ViewGroup parent)
```

public class MyArrayAdapter extends ArrayAdapter<String>{
    private Activity ctx;
    public MyArrayAdapter(Activity context,  String[] objects) {
        super(context, R.layout.layout_rss_list,objects);
        ctx=context;

    }
    @Override
    public View getView(int position,View convertView, ViewGroup parent){
        if(convertView==null){
            convertView=ctx.getLayoutInflater().inflate(R.layout.layout_rss_list,null);
        }
        BootstrapButton btn=(BootstrapButton)convertView.findViewById(R.id.list_rss_btn);
        btn.setText(getItem(position));
        return  convertView;
    }
}

```
##  自定义CursorAdapter: 
覆盖 
public View newView(Context context, Cursor cursor, ViewGroup parent)
 public void bindView(View view, Context context, Cursor cursor)
```

public class MyCursorAdapter extends CursorAdapter {

    public MyCursorAdapter(Context context, Cursor c) {
        super(context, c, 0);
    }

    @Override
    public View newView(Context context, Cursor cursor, ViewGroup parent) {
        LayoutInflater inflater=(LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        return inflater.inflate(R.layout.layout_rss_list,parent,false);
    }

    @Override
    public void bindView(View view, Context context, Cursor cursor) {
        BootstrapButton btn=(BootstrapButton)view.findViewById(R.id.list_rss_btn);
        Log.d("haha", RSSModelDao.Properties.Title.columnName);
//        Log.d("haha",cursor.getColumnIndex(colName)+"");
//        Log.d("haha", cursor.getString(cursor.getColumnIndex(colName)));
        Log.d("haha", cursor.getString(cursor.getColumnIndex(RSSModelDao.Properties.Title.columnName)) + ":btntext");
        btn.setText(cursor.getString(cursor.getColumnIndex(RSSModelDao.Properties.Title.columnName)));
//        btn.setBootstrapType(BootstrapButton.);
        btn.setClickable(false);//防止ListView点击事件被Button阻止,重要。


    }
    }


```