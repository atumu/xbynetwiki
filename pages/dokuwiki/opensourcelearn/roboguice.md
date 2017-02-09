title: roboguice 

#  Android 依赖注入框架RoboGuice 

在开发应用时一个基本原则是模块化，并且尽最大可能性地降低模块之间的耦合性。Dependency injection 大大降低了类之间的依赖性，可以通过annotation描述类之间的依赖性，避免了直接调用类似的构造函数或是使用Factory来参加所需的类，从而降低类或模块之间的耦合性，以提高代码重用并增强代码的可维护性。
Google Guice提供了Java平台上一个轻量级的 Dependency injection 框架，并可以支持开发Android应用。本指南将使用Android平台来说明Google Guice的用法。(其他类似框架有Dagger,Buffernife)
RoboGuice 为Android平台上基于Google Guice开发的一个库，可以大大简化Android应用开发的代码和一些繁琐重复的代码。比如代码中可能需要大量使用findViewById在XML中查找一个View，并将其强制转换到所需类型，onCreate 中可能有大量的类似代码。RoboGuice 允许使用annotation 的方式来描述id于View之间的关系，其余的工作由roboGuice库来完成。

**RoboGuice主要功能：**
  * 控件注入：用@InjectViews方法初始化控件，例如：@InjectView（R.id.textview1）TextView textView1。
  * 资源注入：用@InjectResources方法初始化资源，例如：@InjectResource（R.string.app_name）String name。
  * 系统服务注入：用@Inject方法初始化并获取系统服务，例如：@Inject LayoutInflater inflater。
  * POJO对象注入：用@Inject方法注入并初始化POJO对象，例如：@Inject Foo foo。
以下示例对比看看RoboGuice的作用：
```

不使用RoboGuice:
public void onCreate(Bundle savedInstanceState) {   
        super.onCreate(savedInstanceState);   
        setContentView(R.layout.main);  
        name      = (TextView) findViewById(R.id.name);   
        thumbnail = (ImageView) findViewById(R.id.thumbnail);   
        loc       = (LocationManager) getSystemService(Activity.LOCATION_SERVICE);   
        icon      = getResources().getDrawable(R.drawable.icon);   
        myName    = getString(R.string.app_name);   
        name.setText( "Hello, " + myName );   
    }   
}   
使用RoboGuice:
@ContentView(R.layout.main)  
   class RoboWay extends RoboActivity {   
       @InjectView(R.id.name)             TextView name;   
       @InjectView(R.id.thumbnail)        ImageView thumbnail;   
       @InjectResource(R.drawable.icon)   Drawable icon;   
       @InjectResource(R.string.app_name) String myName;   
       @Inject                            LocationManager loc;   
  
       public void onCreate(Bundle savedInstanceState) {   
           super.onCreate(savedInstanceState);   
           name.setText( "Hello, " + myName );   
       }   
   }   

```
#  使用前准备工作 

RoboGuice官方Demo: https://github.com/roboguice/roboguice/tree/master/astroboy
##  安装 

方式一：添加依赖库：
获取依赖库:` mvn -f pom.xml dependency:copy-dependencies `

方式二：在build.gradle中的dependecies片段中添加：` (推荐) `
```

dependencies {  
    provided 'org.roboguice:roboblender:3.0'  
    compile 'org.roboguice:roboguice:3.0'  
}  

```
参考https://github.com/roboguice/roboguice/wiki/InstallationNonMaven
方式三：Maven:
```

	<dependency>
            <groupId>org.roboguice</groupId>
            <artifactId>roboguice</artifactId>
            <version>x.y.z</version>
        </dependency>
        <!-- If you use RG 3, you can optionally add RoboBlender (advised) -->
        <dependency>
            <groupId>org.roboguice</groupId>
            <artifactId>roboblender</artifactId>
            <scope>provided</scope>
            <version>x.y.z</version>
        </dependency>
        <!-- For the optional Nullable annotation -->
        <dependency>
            <groupId>com.google.code.findbugs</groupId>
            <artifactId>jsr305</artifactId>
            <version>1.3.9</version>
        </dependency>

```
##  继承类： 

activities, services, and fragments都需要继承自RoboGuice提供的下列类：
RoboActivity
RoboListActivity
RoboExpandableListActivity
RoboMapActivity
RoboPreferenceActivity
RoboAccountAuthenticatorActivity
RoboActivityGroup
RoboTabActivity
RoboFragmentActivity
RoboLauncherActivity
RoboService
RoboIntentService
RoboFragment
RoboListFragment
RoboDialogFragment
etc.
#   Injection 

  * Inherit from RoboActivity
  * Set your content view
  * Annotate your views with @InjectView

```

public class MyActivity extends RoboActivity {  
        @InjectView(R.id.text1) TextView textView;  
  
        @Override  
        protected void onCreate( Bundle savedState ) {  
            setContentView(R.layout.myactivity_layout);  
            textView.setText("Hello!");  
        }  
    }  
@ContentView注解：
[java] view plaincopy
@ContentView(R.layout.myactivity_layout)  
  public class MyActivity extends RoboActivity {  
      @InjectView(R.id.text1) TextView textView;  
  
      @Override  
      protected void onCreate( Bundle savedState ) {  
          textView.setText("Hello!");  
      }  
  }  

```
##  将View注入Fragment@InjectView 

**由于Fragment中的依赖注入发生在onViewCreated()中，所以需要在onViewCreated中调用super.onViewCreate()之后才能使用注入的View.
注意这里的：onViewCreated是在onCreateView调用之后恢复保存的状态到View之前立马调用，**
```

@InjectView TextView commentEditText;  
  
@Override  
public void onViewCreated(View view, Bundle savedInstanceState) {  
    super.onViewCreated(view, savedInstanceState);  
  
    commentEditText.setText("Some comment");  
}  

```
##  注入Resource到Activity:@InjectResource 
```

public class MyActivity extends RoboActivity {  
    @InjectResource(R.anim.my_animation) Animation myAnimation;  
}  

```
##  注入System Service:@Inject 
```

class MyActivity extends RoboActivity {  
    @Inject Vibrator vibrator;  
    @Inject NotificationManager notificationManager; 
  </code>
##  注入POJO:@Inject 
<code java>
注意：需要提供一个无参数构造器
class MyActivity extends RoboActivity {  
    @Inject Foo foo; // this will basically call new Foo();  
}  
当然如果不想使用默认构造器，可以注解构造器：
class Foo {  
    Bar bar;  
    @Inject  
    public Foo(Bar bar) {  
        this.bar = bar;  
    }  
}  
也可以注解POJO的Field:
class Foo {  
    @Inject Bar bar;  
  
    // Roboguice doesn't need a constructor, but you might...  
}  

```
##  注入Singleton 
```

前提：对单例类使用@Singleton
@Singleton //a single instance of Foo is now used though the whole app  
class Foo {  
}  

class MyActivity extends RoboActivity {  
    @Inject Foo foo; // this will basically call new Foo();  
}  

```
<note important>注意：这个单例一旦创建会一直驻留内存，其他类似的还有@ContextSingleton,@FragmentSingletion，他们的作用范围比较小，利于垃圾回收。</note>

#  自定义Binding 
```

public interface IFoo {}  
  
public class Foo implements IFoo {}  
[java] view plaincopy
public class MyActivity extends RoboActivity {  
    //How to tell RoboGuice to inject an instance of Foo ?  
    @Inject IFoo foo;  
}  

```
要想注入时绑定接口IFoo到Foo，实例化Foo.需要进行绑定。
步骤：
1.向AndroidManifest.xml中添加meta-data
```

<application ...>  
    <meta-data android:name="roboguice.modules"  
               android:value="com.example.MyModule,com.example.MyModule2" />                  
</application>  

```
2.继承AbstractModule
```

public class MyModule extends AbstractModule {   
    //a default constructor is fine for a Module  
  
    public void bind() {  
        bind(IFoo.class).to(Foo.class);  
    }  
}  

public class MyModule2 extends AbstractModule {  
    //if your module requires a context, add a constructor that will be passed a context.  
    private Context context;  
  
    //with RoboGuice 3.0, the constructor for AbstractModule will use an `Application`, not a `Context`  
    public MyModule( Context context ) {  
        this.context = context;  
    }  
  
    public void bind() {  
        bind(IFoo.class).toInstance( new Foo(context));  
    }  
}  

```
##  注入到Service与BroadcastReceiver: 
```

public class MyService extends RoboService {  
  
   @Inject ComputeFooModule computeFooModule;  

  public class MyBroadcastReceiver extends RoboBroadcastReceiver {  
  
   @Inject ComputeFooModule computeFooModule;  
    </code>
##  注入到自定义View 
<code java>
class MyView extends View {  
    @Inject Foo foo;   
    @InjectView(R.id.my_view) TextView myView;  
  
    public MyView(Context context) {  
        inflate(context,R.layout.my_layout, this);  
        RoboGuice.getInjector(getContext()).injectMembers(this);//注意这一行，为了使其正常工作。  
    }  
  
    public MyView(Context context, AttributeSet attrs, int defStyle) {  
        super(context, attrs, defStyle);  
        inflate(context,R.layout.my_layout, this);  
        RoboGuice.getInjector(getContext()).injectMembers(this);  
    }  
  
    public MyView(Context context, AttributeSet attrs) {  
    super(context, attrs);  
        inflate(context,R.layout.my_layout, this);  
        RoboGuice.getInjector(getContext()).injectMembers(this);  
    }  
  
    @Override  
    public void onFinishInflate() {  
        super.onFinishInflate();  
        //All injections are available from here  
        //in both cases of XML and programmatic creations (see below)  
        myView.setText(foo.computeFoo());  
    }  
}  

```
#  使用RoboGuice Event 
```

public class MyActivity extends RoboActivity {  
  
    // Any method with void return type and a single parameter with @Observes annotation  
    // can be used as an event listener.  This one listens to onResume.      
    public void doSomethingOnResume( @Observes OnResumeEvent onResume ) {  
        Ln.d("Called doSomethingOnResume in onResume");  
    }  
  
    // As you might expect, some events can have parameters.  The OnCreate event  
    // has the savedInstanceState parameter that Android passes to onCreate(Bundle)  
    public void doSomethingElseOnCreate( @Observes OnCreateEvent onCreate ) {  
        Ln.d("onCreate savedInstanceState is %s", onCreate.getSavedInstanceState())  
   }  
  
    // And of course, you can have multiple listeners for a given event.  
    // Note that ordering of listener execution is indeterminate!  
    public void xxx( @Observes OnCreateEvent onCreate ) {  
        Ln.d("Hello, world!")  
    }  
}  

```
本文还有很多特性没有介绍到，后续可能会进行补充。
参考：
http://www.oschina.net/p/roboguice
https://github.com/roboguice/roboguice/wiki
http://mobile.51cto.com/abased-426620.htm