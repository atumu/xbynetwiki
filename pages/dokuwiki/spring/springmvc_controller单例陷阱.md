title: springmvc_controller单例陷阱 

#  springmvc Controller单例陷阱 
原文：http://lavasoft.blog.51cto.com/62575/1394669/
**Spring MVC`  Controller默认是单例的 `**：
单例的原因有二：
1、为了性能。
2、不需要多例。

1、这个不用废话了，单例不用每次都new，当然快了。
2、不需要实例会让很多人迷惑，因为spring mvc官方也没明确说不可以多例。
我这里说不需要的原因是看开发者怎么用了，如果你给controller中定义很多的属性，那么单例肯定会出现竞争访问了。
因此，**只要controller中不定义属性**，那么单例完全是安全的。下面给个例子说明下：以下通过@Scope("prototype")将Controller声明为多例模式。
```

@Controller
@RequestMapping("/demo/lsh/ch5")
@Scope("prototype")
public class MultViewController {
    private static int st = 0;      //静态的
    private int index = 0;          //非静态
    @RequestMapping("/show")
    public String toShow(ModelMap model) {
        User user = new User();
        user.setUserName("testuname");
        user.setAge("23");
        model.put("user", user);
        return "/lsh/ch5/show";
    }
    @RequestMapping("/test")
    public String test() {
        System.out.println(st++ + " | " + index++);
        return "/lsh/ch5/test";
    }
}

```
0 | 0
1 | 0
2 | 0
3 | 0
4 | 0
去掉@Scope("prototype")改为单例的：
0 | 0
1 | 1
2 | 2
3 | 3
4 | 4
从此可见，单例是不安全的，会导致属性重复使用。

**最佳实践：**
1、` 不要在controller中定义成员变量。 `
2、万一必须要定义一个非静态成员变量时候，则通过**注解@Scope("prototype")，将其设置为多例模式。**
