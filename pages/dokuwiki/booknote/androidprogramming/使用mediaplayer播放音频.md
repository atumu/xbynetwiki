title: 使用mediaplayer播放音频 

#  使用MediaPlayer播放音频 
MediaPlayer支持的格式:WAV, MP3,Ogg Vorbis, MPEG-4, and 3GPP等
` 注意：MediaPlayer运行在一个不同的线程上 `。所以即使触发线程死亡，还能继续播放。一般需要在**onDestroy()**方法中手动进行销毁释放。
一般音频文件存放在res/raw (raw存放那些不需要编译系统处理的各类文件)
为保持代码的整洁与独立，推荐对MediaPlayer进行封装。
##  清单声明  
在开发你的应用程序中使用媒体播放器前，先确保你的Manifest有适当的声明来允许使用相关的功能。
Internet许可 -如果你使用的是基于网络内容的流媒体播放器，你的应用程序必须请求访问网络。
`  <uses-permission android:name="android.permission.INTERNET" /> `
唤醒锁定许可 -如果你的程序需要保持屏幕变暗或处理器睡眠，或使用` MediaPlayer.setScreenOnWhilePlaying() `或` MediaPlayer.setWakeMode() `方法的时候，你必须请求许可。
`  <uses-permission android:name="android.permission.WAKE_LOCK" /> `

```

import android.content.Context;
import android.media.MediaPlayer;

public class AudioPlayer {

    private MediaPlayer mPlayer;
    
    public void stop() {
        if (mPlayer != null) {
            mPlayer.release();
            mPlayer = null;
        }
    }

    public void play(Context c) {
        //为避免多个MediaPlayer实例同时播放
        stop();

        mPlayer = MediaPlayer.create(c, R.raw.one_small_step);
        //监听完成事件，一旦完成播放立刻释放资源。
        mPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
            public void onCompletion(MediaPlayer mp) {
                stop();
            }
        });

        mPlayer.start();
    }
    
    public boolean isPlaying() {
        return mPlayer != null;
    }

```
```

  @Override
    public void onDestroy() {
        super.onDestroy();
        mPlayer.stop();
    }

```
##  Android资源Uri构建： 
```

Uri resourceUri = Uri.parse("android.resource://" +
"<pkg_name>/raw/apollo_17_stroll");

```
##  异步准备 
**调用prepare() 方法可能需要较长的时间来执行**，因为它可能涉及获取和解码媒体数据。因此，如同任何方法，可能需要很长时间执行，你不应该从应用程序的UI线程调用它。
为了避免你的UI线程挂起，产生另一个线程准备MediaPlayer 当完成时通知主线程。你可以写自己的线程的逻辑，框架提供` prepareAsync() ` 方法，方便的使用MediaPlayer 。该方法在后台开始准备媒体并立即返回。当媒体准备好了，` MediaPlayer.OnPreparedListener的onPrepared 方法 `，通过配置` setonpreparedlistener() `方法来调用。
##  状态管理  
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150604-160804.png)
**关于MediaPlayer ，你需要记住的另一点是它的状态。**即，在你编写自己的代码的时候，必须时刻意识到**MediaPlayer 有一个内部状态**；
MediaPlayer 类里有文件显示一个完整的状态转换图，阐明哪些方法可以把MediaPlayer 从一个状态改变到另一个状态。例如,当您` 创建 `一个新的MediaPlayer ,它就处于**闲置状态**。这时,你应该调用android.net.Uri) ` setDataSource() `方法来初始化它，把它设置为**初始化状态**。然后，你必须使用` prepare()方法或prepareAsync() `方法。当MediaPlayer 准备好了，它将进入**准备状态**,这就意味着你可以调用` start() `方法来播放媒体。另外，在状态转换图上阐明了，你可以调用` start(),pause()和 seekTo() `这些方法在**Started,Paused和PlaybackCompleted**状态之间进行转换。
` 如果您调用stop()方法,这时请注意,你需要再次调用setDataSource()准备MediaPlayer ，才可以再一次调用start()方法。 ` 
**在编写代码与MediaPlayer 对象交互时，心里要随时想着状态转换图**，因为从错误的状态调用它的方法,是引起错误的常见原因。
##  释放MediaPlayer 
 ```

 if (mPlayer != null) {
            mPlayer.release();
            mPlayer = null;
        }

```
##  使用MediaPlayer 
它支持多种不同的媒体来源，如：
  * 本地资源
  * 内部的URIs，比如你可能获得的内容解析器
  * 外部的URIs（流）

这是一个如何播放本地音频的例子（资源文件保存在你的应用程序res/raw/ directory的资源文件目录下）：
```

MediaPlayer mediaPlayer = MediaPlayer.create(context, R.raw.sound_file_1);
mediaPlayer.start(); // no need to call prepare(); create() does that for you

```
这是你如何通过URI从可用的本地系统中播放（这是你通过内容解析器获得的数据，例如）：
` 注意使用Uri播放时在start()前先调用prepare()，但是prepare()方法可能需要很长时间，所以一般不应该在UI线程上调用它，不过我们可以使用prepareAsync(),实现MediaPlayer.OnPreparedListener，以便当你准备工作完毕后，得到可以开始播放的通知。 `
```

Uri myUri = ....; // initialize Uri here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(getApplicationContext(), myUri);
mediaPlayer.prepare();
mediaPlayer.start();

```
通过一个远程的URL经由HTTP流来播放，就像下面这样子：
```

String url = "http://........"; // your URL here
MediaPlayer mediaPlayer = new MediaPlayer();
mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mediaPlayer.setDataSource(url);
mediaPlayer.prepare(); // might take long! (for buffering, etc)
mediaPlayer.start();

```
注意：如果你通过一个URL来获取一个在线媒体文件，该文件必须能够支持渐进式下载。

##  使用Service控制MediaPlayer 
如果你希望后台播放媒体，你希望用户操作其他应用时继续播放，你必须开始一个Service并且从那里控制MediaPlayer实例。
###  异步运行

首先,如一个**Activity,Service里的所有任务默认在单一线程中完成**。如果你从同一个应用程序里运行一个` Activity和一个Service，它们默认使用相同的线程(“主线程”) `。因此，**Service需要迅速处理传入的意图并且响应它们的时候从不执行冗长的计算**。如果预计调用一些复杂的任务或阻塞，你必须异步处理这些任务：由另一个线程自己实现自己，或使用框架处理异步。 
例如，当你从主要线程使用一个MediaPlayer ，` 你应该调用prepareAsync()方法而不是prepare() 方法，实现MediaPlayer.OnPreparedListener，以便当你准备工作完毕后，得到可以开始播放的通知。 ` 
```

public class MyService extends Service implements MediaPlayer.OnPreparedListener {
    private static final ACTION_PLAY = "com.example.action.PLAY";
    MediaPlayer mMediaPlayer = null;
 
    public int onStartCommand(Intent intent, int flags, int startId) {
        ...        
        if (intent.getAction().equals(ACTION_PLAY)) {
            mMediaPlayer = ... // initialize it here
            mMediaPlayer.setOnPreparedListener(this);
            mMediaPlayer.prepareAsync(); // prepare async to not block main thread
        }
    }
    /** Called when MediaPlayer is ready */
    public void onPrepared(MediaPlayer player) {
        player.start();
    }
}

```
###  处理异步错误 
实现` MediaPlayer.OnErrorListener `，并且将它设置在你的MediaPlayer 实体中。 
```

public class MyService extends Service implements MediaPlayer.OnErrorListener {
    MediaPlayer mMediaPlayer;
    public void initMediaPlayer() {
        // ...initialize the MediaPlayer here...
 
        mMediaPlayer.setOnErrorListener(this);
    }    
    @Override
    public boolean onError(MediaPlayer mp, int what, int extra) {
        // ... react appropriately ...
        // The MediaPlayer has moved to the Error state, must be reset!
    }
}

```
###  使用唤醒锁 
**应用程序在后台播放媒体，其服务在运行期间，设备可能会进入休眠状态。**因为Android系统希望在设备休眠时节省电池。**系统试图关闭手机的应用程序**,是没有必要的，包括CPU和WiFi硬件。但是，如果你的服务正在运行或播放着音乐，**你希望防止系统干扰你的回放。** 
为了确保您的服务在这些条件下能继续运行，**你需要使用“wake locks”。唤醒锁是一种信号系统，它发出信号,显示：应用程序正在使用或可用的功能，或手机闲置。**
<note tip>注意:你应该尽量少用唤醒锁，只有在必要时候才使用它们。它们会使设备的电池寿命大大降低。</note>
你MediaPlayer 正在播放时，需要确保CPU持续运行，当初始化你的MediaPlayer时，调用` setWakeMode() `方法。一旦你这样做了,当暂停或停止时候，MediaPlayer 持有指定的锁：
```

mMediaPlayer = new MediaPlayer();
// ... other initialization here ...
mMediaPlayer.setWakeMode(getApplicationContext(), PowerManager.PARTIAL_WAKE_LOCK);

```
在这个例子中获得唤醒锁是指在保证CPU在唤醒状态。当你通过网络获取媒体和您正在使用WiFi时,**你可能希望有个WifiLock**，可以手动获取并释放。当你开始通过远程URL准备MediaPlayer，你应该创建并获得wi - fi锁。 代码如下：
```

WifiLock wifiLock = ((WifiManager) getSystemService(Context.WIFI_SERVICE))
    .createWifiLock(WifiManager.WIFI_MODE_FULL, "mylock");
     wifiLock.acquire();

```
**当你暂停或停止你的媒体时,或当你不再需要这样的网络,你应该释放该锁**: 代码如下：
` wifiLock.release(); `
###  作为前景服务运行 

**服务通常用于执行后台任务(后台Service可能被中断重启)**，例如获取电子邮件，同步数据，下载内容,或其他。在这些情况下,用户不会意识到这个服务的执行，**甚至可能不会注意到这些服务被打断,后来重新启动**。毫无疑问，后台播放音乐是一个服务，用户能意识到，**任何中断都会严重影响到用户体验。**此外，用户可能会希望在这个服务执行期间作用于它。这种情况，**服务应该运行一个“前景服务”**。**前台服务在系统中持有一个更高水平的重要性，系统几乎从未将服务扼杀**，因为它对用户有着直接的重要性。**当应用在前台运行**，该服务还必须提供一个**状态栏**来通知用户意识有服务正在运行同时允许他们打开一个活动,**可以与服务进行交互**。
为了把你的服务变为**前景服务**，您必须为状态栏创建一个**Notification**，并且从` Service调用startForeground() `方法。
```

String songName;
// assign the song name to songName
PendingIntent pi = PendingIntent.getActivity(getApplicationContext(), 0,
                new Intent(getApplicationContext(), MainActivity.class),
                PendingIntent.FLAG_UPDATE_CURRENT);
Notification notification = new Notification();
notification.tickerText = text;
notification.icon = R.drawable.play0;
notification.flags |= Notification.FLAG_ONGOING_EVENT;
notification.setLatestEventInfo(getApplicationContext(), "MusicPlayerSample",
                "Playing: " + songName, pi);
startForeground(NOTIFICATION_ID, notification);

```
通知区域可见的设备告诉你，服务在前台运行。如果用户选择了这个通知，系统将调用你提供的PendingIntent。在上面的例子中，它打开了一个Activity。（MainActivity） 
![](/data/dokuwiki/booknote/androidprogramming/pasted/20150604-162351.png)
调用stopForeground()方法来释放它。 代码如下：
` stopForeground(true); `
##  处理音频焦点 
例如，一个用户正在听音乐，同时，另一个应用程序有很重要的事需要通知用户，由于吵闹的音乐用户可能不会听到提示音。从Android 2.2开始,Android平台为应用程序**提供了一个方式来协商设备的音频输出。这个机制被称为音频焦点。 ** 
当您的应用程序需要输出音频如音乐或一个通知,这时你就**必须请求音频焦点**。一旦得到焦点，它就可以自由的使用声音输出设备，同时它会不断监听焦点的更改。**如果它被通知已经失去了音频焦点，它会要么立即杀死音频或立即降低到一个安静的水平**（被称为“ducking”——有一个标记,指示哪一个是适当的）当它再次接收焦点时，继续不断播放。
**音频焦点是自然的合作**。应用程序都期望（强烈鼓励）遵守音频焦点指南，但规则并不是系统强制执行的。如果应用程序失去音频焦点后想要播放嘈杂的音乐，在系统中没有什么会阻止他。然而,这样可能会让用户有更糟糕的体验,并可能卸载这运行不当的应用程序。
请求音频焦点,您必须从AudioManager调用` requestAudioFocus() `方法，下面展示一个例子：
```

AudioManager audioManager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
int result = audioManager.requestAudioFocus(this, AudioManager.STREAM_MUSIC,
    AudioManager.AUDIOFOCUS_GAIN);
 
if (result != AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
    // could not get audio focus.}

```
requestAudioFocus()的第一个参数是` AudioManager.OnAudioFocusChangeListener `，每当音频焦点有变动的时候其` onAudioFocusChange() `方法被调用。您还应该在你的服务和活动上实现这个接口。 
```

class MyService extends Service
                implements AudioManager.OnAudioFocusChangeListener {
    // ....
    public void onAudioFocusChange(int focusChange) {
        // Do something based on focus change...
    }
}

```
**focusChange** 参数告诉你音频焦点是如何改变的，并且可以使用以下的值之一（**他们都是在AudioManager中定义常量的**）：
  * AUDIOFOCUS_GAIN: 你已经得到了音频焦点。
  * AUDIOFOCUS_LOSS: 你已经失去了音频焦点很长时间了。你必须停止所有的音频播放。因为你应该不希望长时间等待焦点返回，这将是你尽可能清除你的资源的一个好地方。例如，**你应该释放MediaPlayer**。
  * AUDIOFOCUS_LOSS_TRANSIENT:你暂时失去了音频焦点，但很快会重新得到焦点。**你必须停止所有的音频播放，但是你可以保持你的资源**，因为你可能很快会重新获得焦点。
  * AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK: **你暂时失去了音频焦点，但你可以小声地继续播放音频**（低音量）而不是完全扼杀音频。

```

public void onAudioFocusChange(int focusChange) {
    switch (focusChange) {
        case AudioManager.AUDIOFOCUS_GAIN:
            // resume playback
            if (mMediaPlayer == null) initMediaPlayer();
            else if (!mMediaPlayer.isPlaying()) mMediaPlayer.start();
            mMediaPlayer.setVolume(1.0f, 1.0f);
            break;
        case AudioManager.AUDIOFOCUS_LOSS:
            // Lost focus for an unbounded amount of time: stop playback and release media player
            if (mMediaPlayer.isPlaying()) mMediaPlayer.stop();
            mMediaPlayer.release();
            mMediaPlayer = null;
            break;
        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
            // Lost focus for a short time, but we have to stop
            // playback. We don't release the media player because playback
            // is likely to resume
            if (mMediaPlayer.isPlaying()) mMediaPlayer.pause();
            break;
        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
            // Lost focus for a short time, but it's ok to keep playing
            // at an attenuated level
            if (mMediaPlayer.isPlaying()) mMediaPlayer.setVolume(0.1f, 0.1f);
            break;
    }
}

```
##  处理AUDIO_BECOMING_NOISY意图 
许多编写良好的应用程序有以下特点，**当一个事件导致音频变得聒噪时，自动停止音频播放。(通过外部扬声器输出)**。例如,一个用户戴着耳机听音乐，可能会不小心切断耳机和设备的链接。虽然,这种行为不会自动发生。如果您没有实现这个特性，设备的外部扬声器会将音频播放出来，这可能是用户不希望发生的。
这些情况下通过处理` ACTION_AUDIO_BECOMING_NOISY `意图，可以让你的应用程序停止播放音乐，通过在你的manifest里添加以下代码，你可以注册一个接收器： 
```

<receiver android:name=".MusicIntentReceiver">
   <intent-filter>
      <action android:name="android.media.AUDIO_BECOMING_NOISY" />
   </intent-filter></receiver>

```
```

public class MusicIntentReceiver implements android.content.BroadcastReceiver {
   @Override
   public void onReceive(Context ctx, Intent intent) {
      if (intent.getAction().equals(
                    android.media.AudioManager.ACTION_AUDIO_BECOMING_NOISY)) {
          // signal your service to stop playback
          // (via an Intent, for instance)
      }
   }
}

```
##  从ContentResolver检索媒体 
在媒体播放器应用程序中，另一个可能有用的特性是用户可以在设备上检索音乐。你可以通过为外部媒体查询` ContentResolver `，代码如下：
```

ContentResolver contentResolver = getContentResolver();
Uri uri = android.provider.MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
Cursor cursor = contentResolver.query(uri, null, null, null, null);
if (cursor == null) {
    // query failed, handle error.
} else if (!cursor.moveToFirst()) {
    // no media on the device
} else {
    int titleColumn = cursor.getColumnIndex(android.provider.MediaStore.Audio.Media.TITLE);
    int idColumn = cursor.getColumnIndex(android.provider.MediaStore.Audio.Media._ID);
    do {
       long thisId = cursor.getLong(idColumn);
       String thisTitle = cursor.getString(titleColumn);
       // ...process entry...
    } while (cursor.moveToNext());
}

```
要和MediaPlayer一起使用,你可以这样做,代码如下:
```

long id = /* retrieve it from somewhere */;
Uri contentUri = ContentUris.withAppendedId(
        android.provider.MediaStore.Audio.Media.EXTERNAL_CONTENT_URI, id);
mMediaPlayer = new MediaPlayer();
mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mMediaPlayer.setDataSource(getApplicationContext(), contentUri);

```

##  AudioManager音频 

###  音频流与硬件音量键控制音频音量 
` setVolumeControlStream(AudioManager.STREAM_MUSIC); `

###  使用硬件播放键来控制音频播放 
在许多手机,有线和无线耳机上都有媒体播放按键，如播放、暂停、停止、快进和快退.当用户操作这些键时，系统会广播一个含有**ACTION_MEDIA_BUTTON**动作的intent。
为了响应媒体按键,需要在manifest里面注册一个**BroadcastReceiver**，用来监听如下所示广播.
```

<receiver android:name=".RemoteControlReceiver">
    <intent-filter>
        <action android:name="android.intent.action.MEDIA_BUTTON" />
    </intent-filter>
</receiver>

```
在receiver实现里面，需要获的触发广播的按键。Intent在**EXTRA_KEY_EVENT**里面包含了按键，KeyEvent类包含了一个**KEYCODE_MEDIA_*静态常量列表**，每项代表了各种媒体按键,如**KEYCODE_MEDIA_PLAY_PAUSE**(暂停)和**KEYCODE_MEDIA_NEXT(**下一个)。
```

public class RemoteControlReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
            KeyEvent event = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
            if (KeyEvent.KEYCODE_MEDIA_PLAY == event.getKeyCode()) {
                // Handle key press.
            }
        }
    }
}

AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
...
 
// Start listening for button presses
am.registerMediaButtonEventReceiver(RemoteControlReceiver);
...
 
// Stop listening for button presses
am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);

```
