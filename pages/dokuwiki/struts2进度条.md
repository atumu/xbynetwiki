title: struts2进度条 

#  struts2InAction之struts2进度条 
![](/data/dokuwiki/pasted/20150723-050620.png)
##  Execute And Wait拦截器 
execAndWait不是默认拦截器栈defaultStack的一部分，所以使用时需要首先声明它，而且应该**将其放在最后一个进行声明**。如下
```

 <action name="HeavyDuty1" class="app17a.HeavyDuty">
            <interceptor-ref name="defaultStack"/>
            <interceptor-ref name="execAndWait">
                <param name="delay">1500</param>
            </interceptor-ref>
            <result>/jsp/OK.jsp</result>
        </action>

```
execAndWait拦截器**基于会话**，所以只为每一个会话创建一个实例。同时当其处理Action时它会在` 后台创建一个线程来进行动作的处理 `。并在这个动作完成之前将用户带到一个名为` wait `的result等待页面。如下：
```

<result name="wait">/jsp/Wait.jsp</result>

```
![](/data/dokuwiki/pasted/20150723-050736.png)
![](/data/dokuwiki/pasted/20150723-050750.png)
![](/data/dokuwiki/pasted/20150723-051407.png)