title: fragment中监听onkey事件 

#  android:Fragment中监听onKey事件 

项目中越来越多的用到Fragment，在用Fragment取代TabHost的时候遇到了一个问题，我们都知道，TabHost的Tab为Activity实例，有OnKey事件，但是Fragment中没有，但是又必须监听OnKey事件怎么办（不仅仅是退出哦）,如果仅仅是退出我们可以在Activity中进行统一处理.
　　下面记录一下在ActionBar中监听Fragment的onKey事件。
　　ActionBar实现Onkey事件，判断当前的fragment是哪一个，是不是所需要的Fragment,然后在需要监听OnKey事件的Fragment中写一个静态方法，传递keycode与event事件即可。
```

package info.androidhive.tabsswipe;
import info.androidhive.tabsswipe.adapter.TabsPagerAdapter;
import android.annotation.SuppressLint;
import android.app.SearchManager;
import android.content.Context;
import android.support.v7.app.ActionBar;
import android.support.v7.app.ActionBar.Tab;
import android.support.v7.app.ActionBarActivity;
import android.support.v7.widget.SearchView;
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v4.view.MenuItemCompat;
import android.support.v4.view.ViewPager;
import android.util.Log;
import android.view.KeyEvent;
import android.view.Menu;
import android.view.MenuItem;

@SuppressLint("NewApi")
public class MainActivity extends ActionBarActivity  implements
        ActionBar.TabListener {

    private ViewPager viewPager;
    private TabsPagerAdapter mAdapter;
    private ActionBar actionBar;
    private Fragment fg;
    // Tab titles
    private String[] tabs = { "TopRatedFragment", "GamesFragment", "MoviesFragment" };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initilization
        viewPager = (ViewPager) findViewById(R.id.pager);
        actionBar = getSupportActionBar();
        mAdapter = new TabsPagerAdapter(getSupportFragmentManager());
        viewPager.setOffscreenPageLimit(3);
        viewPager.setAdapter(mAdapter);
        actionBar.setHomeButtonEnabled(false);
        actionBar.setNavigationMode(ActionBar.NAVIGATION_MODE_TABS);        

        // Adding Tabs
        for (String tab_name : tabs) {
            actionBar.addTab(actionBar.newTab().setText(tab_name)
                    .setTabListener(this));
        }

        /**
         * on swiping the viewpager make respective tab selected
         * */
        viewPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener() {

            public void onPageSelected(int position) {
                // on changing the page
                // make respected tab selected
//                actionBar.setSelectedNavigationItem(position);
                actionBar.selectTab(actionBar.getTabAt(position));
                mAdapter.getItem(position);
                
            }

            public void onPageScrolled(int arg0, float arg1, int arg2) {
            }

            public void onPageScrollStateChanged(int arg0) {
            }
        });
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
//        
        SearchManager searchManager = (SearchManager) getSystemService(Context.SEARCH_SERVICE);
        getMenuInflater().inflate(R.menu.main, menu);
        MenuItem searchItem = menu.findItem(R.id.action_settings);
        SearchView searchview = (SearchView)MenuItemCompat.getActionView(searchItem);
        searchview.setSearchableInfo(searchManager.getSearchableInfo(getComponentName()));
        return super.onCreateOptionsMenu(menu);
    }
    
    
    public void onTabReselected(Tab arg0,
            android.support.v4.app.FragmentTransaction arg1) {
        // TODO Auto-generated method stub
        
    }

    public void onTabSelected(Tab arg0,
            android.support.v4.app.FragmentTransaction arg1) {
        // TODO Auto-generated method stub
        viewPager.setCurrentItem(arg0.getPosition());
        fg = mAdapter.getItem(arg0.getPosition());
        Log.d("fg", fg+"");
    }

    public void onTabUnselected(Tab arg0,
            android.support.v4.app.FragmentTransaction arg1) {
        // TODO Auto-generated method stub
        
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        // TODO Auto-generated method stub
        Log.d("ActionBar", "OnKey事件");
        if(fg instanceof GamesFragment){
           return GamesFragment.onKeyDown(keyCode, event);
        }
        return super.onKeyDown(keyCode, event);
    }
}

```
fragment
```

package info.androidhive.tabsswipe;

import info.androidhive.tabsswipe.R;
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.util.Log;
import android.view.KeyEvent;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Toast;

public class GamesFragment extends Fragment {

    private View view;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        if (view == null) {
            view = inflater.inflate(R.layout.fragment_games, container, false);
        }
        ViewGroup parent = (ViewGroup) view.getParent();
        if (parent != null) {
            parent.removeView(view);
        }
        return view;
    }

    @Override
    public void onResume() {
        super.onResume();
        // 判断当前fragment是否显示
        if (getUserVisibleHint()) {
            showdata();
        }
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        // 每次切换fragment时调用的方法
        if (isVisibleToUser) {
            showdata();
        }
    }

    private void showdata() {
        Toast.makeText(getActivity(), "Game", Toast.LENGTH_LONG).show();
    }

    public static boolean onKeyDown(int keyCode, KeyEvent event) {
        // TODO Auto-generated method stub
        if (keyCode == event.KEYCODE_BACK) {
            Log.d("GameFragmet事件", "OK");
        }
      	//如果处理完成，返回true阻止事件继续传递。到此结束。
        return true;
    }
}

```