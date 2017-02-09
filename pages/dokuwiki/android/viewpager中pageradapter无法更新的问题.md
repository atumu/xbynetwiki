title: viewpager中pageradapter无法更新的问题 

#  android:ViewPager中PagerAdapter无法更新的问题 

#  补充：关于自定义PagerAdapter: 

需要重写至少以下四个方法：（API规定）
  * instantiateItem(ViewGroup, int)
  * destroyItem(ViewGroup, int, Object)
  * getCount()
  * isViewFromObject(View, Object)
但是如果你想更新PagerAdapter,还需要重写
  *  public int getItemPosition(Object object) 
官方API文档中
android.support.v4.app.FragmentPagerAdapter中有个自定义FragmentPagerAdapter的例子可以参考。还可以参考http://www.2cto.com/kf/201401/272085.html


#  PagerAdapter无法更新问题解决 

对于PagerAdapter（FragmentStatePagerAdapter也是如此），**调用它的notifyDataSetChanged()并不会更新视图**。因为在它调用的getItemPosition()只会检测Item位置是否变化.所以我们需要做一些处理。
由于ViewPager在收到adapter的**notifyDataSetChanged()**调用时会调用` getItemPosition `方法，所以这个方法是我们能够解决PageAdapter更新的核心.
如果该方法**return POSITION_NONE**，ViewPager会通过视图重建来更新视图，**但是不推荐在getItemPosition中返回POSITION_NONE，因为这样是非常低效的**。将会导致ViewPager完全销毁所有视图。并重新创建。这将导致很大的消耗。尤其是对于如果你继承FragmentStatePagerAdapter的时候，这将导致instantiateItem()中创建Fragment的销毁与重建。这完全是没有必要的。	
**而采用下面这种方式，只是会创建一些需要更新的子组件，而instantiateItem()返回的object则不用销毁与重建**。在本例中这个object就是LinearLayout,但是如果对于FragmentStatePagerAdapter则不会导致instantiateItem()返回Fragment被重建。所以有效地进行PageAdapter的视图更新。
解决思路：
![](/data/dokuwiki/android/pasted/20150516-140924.png)
#  对于PagerAdapter直接之类 

```

public class MyRSSPagerAdapter extends PagerAdapter {
    private SQLiteOpenHelper helper;
    private RSSModelDao dao;
    private Context ctx;
    private final SQLiteDatabase db;
    private LayoutInflater inflater;
    private TextView textView;
    private WebView webView;
    private RSSModel model;
    private String webViewContent;
    public MyRSSPagerAdapter(Context ctx, RSSModel model) {
        this.model = model;
        helper = RssUtils.getInstance().getSQLiteOpenHelper();
        db = helper.getWritableDatabase();
        dao = RssUtils.getInstance().getRssDao(db);
        this.ctx = ctx;
        inflater = (LayoutInflater) ctx.getSystemService(Context.LAYOUT_INFLATER_SERVICE);

    }

    public void chageModel(RSSModel model) {
        if (model != null) {
            Log.d("haha", "model not null");
            this.model = model;
//            notifyDataSetChanged();

        }
    }


    @Override
    public int getCount() {
        return (int) (dao.count());
    }
    @Override
    public boolean isViewFromObject(View view, Object object) {
	/**官方文档中推荐这么写。*/
        return view == object;
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {

        ((ViewPager) container).removeView((View) object);
/**注意不要调用super.destroyItem(container,position,object);*/
    }
	/**instantiateItem()返回的Object不一定要是View，它可以是像Fragment等对象。而参数ViewGroup这是ViewPager对象*/
    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        final int pos = position;
        LinearLayout layout = (LinearLayout) inflater.inflate(R.layout.adapter_rss_pager, null);
        updateView(layout);
        ((ViewPager) container).addView(layout);
        return layout;
    }
    private View updateView(View layout){
        textView = (TextView) layout.findViewById(R.id.titleView1);
        webView = (WebView) layout.findViewById(R.id.webView1);
       WebSettings settings = webView.getSettings();
        //开启缩放支持
        settings.setSupportZoom(true);
        settings.setBuiltInZoomControls(true);
        settings.setJavaScriptEnabled(true);
        //默认对缩放比例有限制，导致用户体验不好，所以需要设置为使用任意比例缩放。
        settings.setUseWideViewPort(true);
        //使页面之间可以点击链接导航
        webView.setWebViewClient(new WebViewClient());
        webView.setWebChromeClient(new WebChromeClient());
        webView.getSettings().setDefaultTextEncodingName("UTF -8");//设置默认为utf-8
        if (model != null) {
            Log.d("haha", "RSSAdapterset:" + model.getTitle());
            textView.setText(model.getTitle());
            webView.loadDataWithBaseURL(null, model.getSummary(), "text/html", "UTF-8", null);
        }else{
            textView.setText("");
            webView.loadDataWithBaseURL(null, "", "text/html", "UTF-8", null);
        }
        webView.addJavascriptInterface(new WebViewContent(),"handler");
        webView.setWebViewClient(new WebViewClient() {
            @Override
            public void onPageFinished(WebView view, String url) {
                view.loadUrl("javascript:window.handler.getContent('<html>'+document.getElementsByTagName('html')[0].innerHTML+'</html>');");
                super.onPageFinished(view, url);
            }
        });
       return layout;
    }
	/**
	* 由于ViewPager在收到adapter的notifyDataSetChanged()调用时会调用getItemPosition方法，所以这个方法是我们能够解决PageAdapter更新的核心
	*如果该方法return POSITION_NONE，ViewPager会通过视图重建来更新视图，但是不推荐在getItemPosition中返回POSITION_NONE，因为这样是非常低效的。将会导致ViewPager完全销毁所有视图。并重新创建。这将导致很大的消耗。尤其是对于如果你继承FragmentStatePageAdapter的时候，
	*这将导致instantiateItem()中创建Fragment的销毁与重建。这完全是没有必要的。
	*
	*    @Override
    * public int getItemPosition(Object object) {
    *    return POSITION_NONE;
    *}
	*而采用下面这种方式，只是会创建一些需要更新的子组件，而instantiateItem()返回的object则不用销毁与重建。在本例中这个object就是LinearLayout,但是如果对于
	*FragmentStatePageAdapter则不会导致instantiateItem()返回Fragment被重建。所以有效地进行PageAdapter的视图更新。
	*/
    @Override
     public int getItemPosition(Object object) {
        View v=(View)object;
	/**重新从LayoutInflator加载view组件已到达更新视图的目的。*/
        updateView(v);
	/**注意千万不要调用旧的view引用的设置方法。需要重新从LayoutInflator加载view组件，否则报错
      *  textView.setText(model.getTitle());
      *  webView.loadDataWithBaseURL(null, model.getSummary(), "text/html", "UTF-8", null);
      */
        return super.getItemPosition((View)object);
    }

}

```
然后通知改变：
```

  rssPageAdapter.chageModel(data);
rssPageAdapter.notifyDataSetChanged();

```
#  对于Fragment(State)PageAdapter子类： 
解决思路：不知道是否有更好的思路。
![](/data/dokuwiki/android/pasted/20150516-141512.png)
首先创建一个PageItem用于存储名称与对应的Fragment:
```

public class PagerItem {
private String mTitle;
private Fragment mFragment;


public PagerItem(String mTitle, Fragment mFragment) {
    this.mTitle = mTitle;
    this.mFragment = mFragment;
}
public String getTitle() {
    return mTitle;
}
public Fragment getFragment() {
    return mFragment;
}
public void setTitle(String mTitle) {
    this.mTitle = mTitle;
}

public void setFragment(Fragment mFragment) {
    this.mFragment = mFragment;
}

}

```
然后定义FragmentPagerAdapter子类：
```

public class MyPagerAdapter extends FragmentPagerAdapter {
  private FragmentManager mFragmentManager;
private ArrayList<PagerItem> mPagerItems;//维护一个PagerItem列表

public MyPagerAdapter(FragmentManager fragmentManager, ArrayList<PagerItem> pagerItems) {
    super(fragmentManager);
    mFragmentManager = fragmentManager;
    mPagerItems = pagerItems;
}
  public void setPagerItems(ArrayList<PagerItem> pagerItems) {
  /**
  *你可以在任何时候调用viewPager.setAdapter(),但是如果没有移除之前的Fragment,是不会产生效果的。所以
  *在更新任何Fragment视图前通过FragmentManager移除原有Fragment*/
    if (mPagerItems != null)
        for (int i = 0; i < mPagerItems.size(); i++) {
            mFragmentManager.beginTransaction().remove(mPagerItems.get(i).getFragment()).commit();
        }
    mPagerItems = pagerItems;
 }
}

```
如需更新数据和视图，调用:
```

ArrayList<PagerItem> pagerItems = new ArrayList<PagerItem>();
    pagerItems.add(new PagerItem("Fragment1", new MyFragment1()));
    pagerItems.add(new PagerItem("Fragment2", new MyFragment2()));

    mPagerAdapter.setPagerItems(pagerItems);
    mPagerAdapter.notifyDataSetChanged();

```
参考：
[自定义PageAdapter](http://codego.net/511532/)
http://stackoverflow.com/questions/7263291/viewpager-pageradapter-not-updating-the-view