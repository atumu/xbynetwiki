title: struts2_action参数注入三种方式 

#  struts2 action参数注入三种方式 
struts2接收前台传来的参数有3种方法
**1、第一种：action 设置相应的属性**
在相应的action中设置与将要传进来的参数名相同的变量 
eg： 前台传给后台两个参数 name=chance & age = 1,那么后台的action中 要设定这样的变量：private String name; private int age;**同时，必须设置set，和 get方法**
struts2会自动将前台参数赋值进来。

**2、第二中：DomainModule**
domainmodule 中文解释：域模型，domainmodul接收参数的方式就是，在action中设一个module对象
例如，一个bbs系统，实现用户注册模块，那么后台module中 我们就会相应的建立一个 User类，这个user类就是我们前面说过的module，
分析下为什么要用DomainModule来接收参数，我们从前台向后台的action中传递参数，action做的处理无非是紧接着再把这个参数传递给对应的module，那么假如我们的module有100个变量（呵呵，我说的是假如），那么我们用第一种方法接收参数时就要在action中设置100个变量，但如果我们用domainmodule的话就简单的多，说了这么多，还没讲到怎么用呵呵，切入正题：

eg：有一个用户登录系统，前台需要向后台传递一个user的name 和 password 两个属性，
那么我们可以这么处理，**首先在相应的action 设一个` private User user `; 变量（假如我们已经有了User这个Module了）注意：我们依然需要` 继续给user设置 set get方法 `**，好那么前台传递参数的时候可以这么来写，
` action？user.name=chance&user.password=123 `
另外需要注意的是，在action中设置的变量 无论是基本类型，还是引用类型，**我们只需要声明，但不需要定义（简单的说，就是我们不需要去 new 一个变量）new的过程 由struts来帮我们完成**

下面我们在来考虑一个问题，还拿上面的用户登录系统来举例，通常用户登录的时候 除了用户名，密码，还会填写一个 确认密码（其实这个工作完全可以交给客户端的js来处理，这里只是为了说明问题），但是在 User Module抽象封装的过程它是不会有 confimPassword这一项的，这样我们就不能用domainModule来解决这个问题，怎么办?
解决方法就是**引入 DTO（又交 do，或vo）data transform object,它的工作就两点：接收一下，传递一下；**
  * 接收一下：前台传过来的user对象，我们不直接传递给usermodule 而是**传递给 dto对象，例如userDto**（它里面会有一个confimPassword变量）
  * 传递一下：dto接收来参数后 进过一番数据处理，确认密码输入正确那么就会 把 必要的参数变量传递给 usermodule 

**第三种方式,在项目中使用模型驱动(ModelDriven)**
需要让Action实现` com.opensymphony.xwork2.ModelDriven ` 接口，使用它的` getModel() `方法来通知Struts2要注入的属性类型，**并且声明属性时一定要实例化,但不需get，set方法**(这是与第二种方式的区别)。
但是与第二种方式有一个很大的区别:那就是前台对这些属性名的写法.
**''第二种方式需要这样写：<input name="user.name" /> ,<input name="user.password" />
但是第三种方式这样写：<input name="name"/>,<input name="password"/>''**
```

public class EditUserInfo extends ActionSupport implements ModelDriven{ 实现ModelDriven接口
private User user = new User(); //模型对象初始化
public Object getModel() { ModelDriven接口的方法。

	return user;
}
public EditUserInfo() {
}
protected String execute() {
System.out.println(user.getName());
System.out.println(user.getPassword());
//业务处理......
return Action.SUCCESS;
}

```
**ModelDriven背后的机制就是ValueStack。**界面通过：username/age/address这样的名称，就能够被直接赋值给user对象，这证明**user对象正是ValueStack中的一个root对象！**
那么，**为什么user对象会在ValueStack中呢？**它是什么时候被压入ValueStack的呢？答案是：` ModelDrivenInterceptor `
当一个请求经过ModelDrivenInterceptor的时候，在这个拦截器中，会判断当前要调用的Action对象是否实现` 了ModelDriven接口 `，如果实现了这个接口，则调用` getModel() `方法，**并把返回值（本例是返回user对象）压入ValueStack。**这一点可以从ModelDrivenInterceptor的源代码中看出来。由于struts-default中定义了ModelDriven拦截器，所以我们无需额外定义。
```

public class ModelDrivenInterceptor extends AbstractInterceptor {
    protected boolean refreshModelBeforeResult = false;
    public void setRefreshModelBeforeResult(boolean val) {
        this.refreshModelBeforeResult = val;
    }
    @Override
    public String intercept(ActionInvocation invocation) throws Exception {
        Object action = invocation.getAction();
 
        if (action instanceof ModelDriven) {
            ModelDriven modelDriven = (ModelDriven) action;
            ValueStack stack = invocation.getStack();
            Object model = modelDriven.getModel();
            if (model !=  null) {
              stack.push(model);//从ModelDrivenInterceptor中，即可以看到model对象被压入ValueStack中！
            }
            if (refreshModelBeforeResult) {
                invocation.addPreResultListener(new RefreshModelBeforeResult(modelDriven, model));
            }
        }
        return invocation.invoke();
    }

```
  
参考：
http://blog.csdn.net/li_tengfei/article/details/6098145
http://blog.csdn.net/li_tengfei/article/details/6098134