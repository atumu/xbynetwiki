title: springmvc_flash_attribute_的讲解与使用 

#  SpringMVC Flash Attribute 的讲解与使用 
参考：http://www.oschina.net/translate/spring-mvc-flash-attribute-example?p=2#comments
http://blog.csdn.net/jackpk/article/details/19121777
Spring MVC 3.1版本加了一个很有用的特性，**Flash属性**，它能解决一个长久以来缺少解决的问题，一个**POST/Redirect/GET模式**问题。
正常的MVC Web应用程序在每次提交都会POST数据到服务器。一个正常的Controller (被注解 @Controller标记)从请求获取数据和处理它 (保存或更新数据库)。一旦操作成功，用户就会被带到（forward）一个操作成功的页面。传统上来说，这样的POST/Forward/GET模式,有时候会导致多次提交问题. 例如用户按F5刷新页面，这时同样的数据会再提交一次。
为了解决这问题,** POST/Redirect/GET 模式**被用在MVC应用程序上. **一旦用户表单被提交成功， 我们重定向（Redirect）请求到另一个成功页面。这样能够令浏览器创建新的GET请求和加载新页面。**这样用户按下F5，是直接GET请求而不是再提交一次表单。

虽然这一方法看起来很完美，并且**解决了表单多次提交的问题**，但是**它又引入了一个获取请求参数和属性的难题**. 通常当我们生成一次**http重定向请求的时候，被存储到请求数据会丢失**，使得下一次GET请求不可能访问到这次请求中的一些有用的信息.
**` Flash attributes ` 的到来就是为了处理这一情况.** Flash attributes 为一个请求存储意图为另外一个请求所使用的属性提供了一条途径.
` Flash attributes ` 在对请求的重定向生效之前被` 临时存储 `（通常是在session)中，并且在` 重定向之后被立即移除. `但是要注意的一点是：` **如果此时在重定向之后的页面再次按F5刷新，由于数据被移除，将得不到相关属性。** `
![](/data/dokuwiki/spring/pasted/20150914-060619.png)

要想在你的 Spring MVC 应用中使用 Flash attribute，要用 3.1 版本或以上。并且要在 spring-servlet.xml 文件中加入 mvc:annotation-driven。
` <mvc:annotation-driven /> `
这些都完成之后，Flash attribute 就会自动设为“开启”，以供使用了。只需在你的 Spring controller 方法中加入RedirectAttributes redirectAttributes。
```

   @RequestMapping(value="addcustomer", method=RequestMethod.POST)
    public String addCustomer(@ModelAttribute("customer") Customer customer,
            final RedirectAttributes redirectAttributes) {
    //...
        redirectAttributes.addFlashAttribute("message", "Successfully added..");
    //...
 
        return "redirect:some_other_request_name";
    }

```
addFlashAttribute 方法会自动向 output flash map 中添加给定的参数，并将它传递给后续的请求` 。但是，一旦重定向成功就会被立即移除，下次刷新将会出现数据丢失的情况。 `
这里的原理是放到session中，session在跳到页面后马上移除对象。所以你刷新一下后这个值就会丢掉。
