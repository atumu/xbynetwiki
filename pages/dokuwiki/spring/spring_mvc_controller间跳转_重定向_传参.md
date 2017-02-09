title: spring_mvc_controller间跳转_重定向_传参 

#  spring mvc controller间跳转 重定向 传参 
参考：http://blog.csdn.net/jackpk/article/details/19121777
一部分在[[spring:springmvc_flash_attribute_的讲解与使用]]
需求背景
需求：spring MVC框架controller间跳转，需重定向。有几种情况：
  * 不带参数跳转，
  * 带参数拼接url形式跳转，
  * 带参数不拼接参数跳转，页面也能显示。

解决办法
（1）我在后台一个controller跳转到另一个controller，为什么有这种需求呢，是这样的。我有一个列表页面，然后我会进行新增操作，新增在后台完成之后我要跳转到列表页面，不需要传递参数，列表页面默认查询所有的。
方式一：使用ModelAndView
return new ModelAndView("redirect:/toList");
这样可以重定向到toList这个方法
方式二：返回String
 return "redirect:/ toList "; 
其它方式：其它方式还有很多，这里不再做介绍了，比如说response等等。这是不带参数的重定向。

（2）第二种情况，列表页面有查询条件，跳转后我的查询条件不能丢掉，这样就需要带参数的了，带参数可以拼接url

方式一：自己手动拼接url
new ModelAndView("redirect:/toList？param1="+value1+"&param2="+value2);
 这样有个弊端，就是传中文可能会有乱码问题。

方式二：用` RedirectAttributes `，这个是发现的一个比较好用的一个类
这里用它的` addAttribute `方法，这个实际上**重定向过去以后你看url，是它自动给你拼了你的url。**
缺点，如需传输敏感数据则不行。
使用方法：
attr.addAttribute("param", value);
return "redirect:/namespace/toController";
这样在toController这个方法中就可以通过获得参数的方式获得这个参数，再传递到页面。过去的url还是和方式一一样的。

（3）带参数不拼接url页面也能拿到值（重点是这个）使用Spring3.1新加的` RedirectAttributes的addFlashAttribute `。
```

 @RequestMapping("/save")
    public String save(@ModelAttribute("form") Bean form,RedirectAttributes attr)
                   throws Exception {


        String code =  service.save(form);
        if(code.equals("000")){
            attr.addFlashAttribute("name", form.getName());  
            attr.addFlashAttribute("success", "添加成功!");
            return "redirect:/index";
        }else{
            attr.addAttribute("projectName", form.getProjectName());  
            attr.addAttribute("enviroment", form.getEnviroment());  
            attr.addFlashAttribute("msg", "添加出错!错误码为："+rsp.getCode().getCode()+",错误为："+rsp.getCode().getName());
            return "redirect:/maintenance/toAddConfigCenter";
        }
    }

```
页面取值不用我说了吧，直接用el表达式就能获得到，这里的原理是` 放到session中，session在跳到页面后马上移除对象。所以你刷新一下后这个值就会丢掉 `。


 带参数可使用` RedirectAttributes `参数进行传递：
 注意：1.使用` RedirectAttributes的addAttribute `方法传递参数会跟随在URL后面，如上代码即为http:/index.action?a=a
2.使用` addFlashAttribute `不会跟随在URL后面，会把该参数值**暂时保存于session**，` 待重定向url获取该参数后立即从session中移除 `，这里的` redirect必须是方法映射路径，jsp无效 `。你会发现redirect后的` jsp页面中name只会出现一次，刷新后name再也不会出现了 `，这验证了上面说的，name被访问后就会从session中移除。对于重复提交可以使用此来完成.

<note tip>选择：当数据不是很敏感的时候尽量采用RedirectAttributes的addAttribute重定向参数传递，当数据比较敏感时采用RedirectAttributes的addFlashAttribute进行重定向参数传递。</note>