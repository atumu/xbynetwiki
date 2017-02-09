title: springmvc与struts2区别 

#  springmvc与Struts2区别 
参考：
http://blog.csdn.net/chenleixing/article/details/44570681
http://blog.csdn.net/gstormspire/article/details/8239182
从设计实现角度来说，SpringMVC更加清晰。即使我们去对比Struts2的原理图和SpringMVC的类图，它依然很让人困惑，远没有SpringMVC更加直观：
![](/data/dokuwiki/spring/pasted/20150914-043705.png)![](/data/dokuwiki/spring/pasted/20150914-043711.png)

SpringMVC设计思路：将整个处理流程规范化，并把每一个处理步骤分派到不同的组件中进行处理。
这个方案实际上涉及到两个方面：
  * l **处理流程规范化** —— 将处理流程划分为**若干个步骤（任务）**，并使用一条明确的**逻辑主线**将所有的步骤串联起来
  * l **处理流程组件化** —— 将处理流程中的**每一个步骤（任务）都定义为接口，并为每个接口赋予不同的实现模式**

处理流程规范化是目的，对于处理过程的步骤划分和流程定义则是手段。因而**处理流程规范化**的首要内容就是考虑一个通用的Servlet响应程序大致应该包含的逻辑步骤：
l 步骤1—— 对Http请求进行初步处理，查找与之对应的Controller处理类（方法）   ——**HandlerMapping**
l 步骤2—— 调用相应的Controller处理类（方法）完成业务逻辑                    ——**HandlerAdapter**
l 步骤3—— 对Controller处理类（方法）调用时可能发生的异常进行处理            ——**HandlerExceptionResolver**
l 步骤4—— 根据Controller处理类（方法）的调用结果，进行Http响应处理       ——**ViewResolver**
 
正是这**基于组件、接口的设计**，支持了SpringMVC的另一个特性：**行为的可扩展性**。

第四、组件化的设计方案和特定的设计原则让SpringMVC形散神聚。
  * 神 —— SpringMVC总是**沿着一条固定的逻辑主线运行**
  * 形 —— SpringMVC却拥**有多种不同的行为模式**


最重要的一点:**Struts2的Action是每个请求创建一个Action实例，而SpringMVC中的` Controller是单例的 `。**

1、Struts2是**类级别的**拦截， 一个类对应一个request上下文，
SpringMVC是**方法级别的**拦截，一个方法对应一个request上下文，而方法同时又跟一个url对应,所以说从架构本身上SpringMVC就容易实现restful url,而struts2的架构实现起来要费劲，因为Struts2中Action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了。

2、由上边原因，**SpringMVC的方法之间基本上独立的，独享request response数据，请求数据通过参数获取，处理结果通过ModelMap交回给框架**，` 方法之间不共享变量 `，
而Struts2搞的就比较乱，虽然方法之间也是独立的，但其所有Action变量是共享的，这不会影响程序运行，却给我们编码 读程序时带来麻烦，每次来了请求就创建一个Action，一个Action对象对应一个request上下文。
3、由于Struts2需要针对每个request进行封装，把request，session等servlet生命周期的变量封装成一个一个Map，供给每个Action使用，并保证线程安全，所以在原则上，是比较耗费内存的。

4、 拦截器实现机制上，**Struts2有以自己的interceptor机制**，**SpringMVC用的是独立的AOP方式**，这样导致Struts2的配置文件量还是比SpringMVC大。

5、**SpringMVC的入口是servlet，而Struts2是filter**（这里要指出，filter和servlet是不同的。以前认为filter是servlet的一种特殊），这就导致了二者的机制不同，这里就牵涉到servlet和filter的区别了。
6、**SpringMVC集成了Ajax**，使用非常方便，只需一个注解` @ResponseBody `就可以实现，**然后直接返回响应文本即可**，而Struts2拦截器集成了Ajax，在Action中处理时一般必须安装插件或者自己写代码集成进去，使用起来也相对不方便。

7、**SpringMVC验证支持JSR303**，处理起来相对更加灵活方便，而Struts2验证比较繁琐，感觉太烦乱。

8、**Spring MVC和Spring是无缝的。**从这个项目的管理和安全上也比Struts2高（当然Struts2也可以通过不同的目录结构和相关配置做到SpringMVC一样的效果，但是需要xml配置的地方不少）。
9、 设计思想上，Struts2更加符合OOP的编程思想， SpringMVC就比较谨慎，在servlet上扩展。
10、**SpringMVC开发效率和性能高于Struts2。**
11、**SpringMVC可以认为已经100%零配置。**
