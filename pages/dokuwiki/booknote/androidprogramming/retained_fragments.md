title: retained_fragments 

#  Retained Fragments(保留Fragment) 
##  保留fragment实例： 
为应对设备配置的变化，可以使用fragment的**setRetainInstance(true)**;是这个fragment可以被保留.
保留的Fragment利用了这样的一个事实：` 可销毁和重建fragment视图，但是无需销毁fragment本身 `
```

public class HelloMoonFragment extends Fragment {
    private AudioPlayer mPlayer = new AudioPlayer();
    
    private Button mPlayButton;
    private Button mStopButton;
      
    void updateButtons() {
        boolean isEnabled = !mPlayer.isPlaying();
        mPlayButton.setEnabled(isEnabled);
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
      //设置保留fragment
       setRetainInstance(true);

    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        mPlayer.stop();
    }
    
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup parent, Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.fragment_hello_moon, parent, false);

        mPlayButton = (Button)v.findViewById(R.id.hellomoon_playButton);
        mStopButton = (Button)v.findViewById(R.id.hellomoon_stopButton);
        
        updateButtons();
        
        mPlayButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                mPlayer.play(getActivity());
                updateButtons();
            }
        });
        
        mStopButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {                
                mPlayer.stop();
                updateButtons();
            }
        });

        return v;
    }
}

```
不保留fragment谁设备旋转变化
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150604-153515.png)
保留fragment随着设备旋转变化：
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150604-153553.png)
**虽然保留的fragment没有被销毁，但是它已经脱离了消亡中的Activity并处于保留状态。**
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150604-153721.png)
##  处理设备旋转的另外一项选择onSaveInstanceState(Bundle) 
与fragment setRetainInstance(true)的区别：前者可以保存需要长时间保存的东西，因为前者即使系统需要内存回收销毁后还能重建状态。但是后者仅限于设备配置变化的重建状态。
所以如果需要长时间保存（即使应用退出或者Activity被系统回收）的东西。则不应该使用fragment的setRetainInstance(true),而应该使用Activity的onSaveInstanceState(Bundle)