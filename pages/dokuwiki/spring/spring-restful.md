title: spring-restful 

#  spring-restful 
##  概述： 
具体介绍可参考：http://www.ruanyifeng.com/blog/2011/09/restful
越来越多的人开始意识到，**网站即软件**，而且是一种新型的软件。这种"互联网软件"采用客户端/服务器模式，建立在分布式体系上，通过互联网通信，具有高延时（high latency）、高并发等特点。
RESTful架构风格，就是目前最流行的一种互联网软件架构风格。
Fielding将他对互联网软件的架构原则，定名为**REST，即Representational State Transfer的缩写**。我对这个词组的翻译是"**表现层状态转化"**。
如果一个架构符合REST原则，就称它为RESTful架构。REST相比于SOAP（Simple Object Access protocol，简单对象访问协议）以及XML-RPC更加简单明了
**JSR-311**是 Sun Microsystems 的规范，可以为开发 RESTful Web 服务定义一组 Java API。**Jersey是对 JSR-311 的参考实现。**
开发人员可以轻松使用 Ajax 和 RESTful Web 服务一起创建丰富的界面。

spring3对rest的支持是构建在Spring MVC之上的。
REST与RPC的区别：REST是` 面向资源 `的，而RPC是面向服务的。

##  RESTful无状态要点 
` Web 应用程序最重要的 REST 原则是，客户端和服务器之间的交互在请求之间是无状态的。 `从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外，无状态请求可以由任何可用服务器回答，这十分适合云计算之类的环境。客户端可以缓存数据以改进性能。

说白了我们不能使用cookie，不能使用session了。如果稍微有点http经验的人，都知道，很多时候，我们都把一些基本内容放在cookie里面，服务器每次读取或者写入的时候，服务器端就直接设置session就可以了，这样，每次，客户端直接携带自己的cookie值上来，我们就知道它是谁，怎么把数据给它。但restful api的风格不允许这样，那服务器应该采取何种方案呢？

**目前网上大多数做法是token方式，第一次登录的时候，先提交用户名密码，服务器收集到以后，先验证一下，如果验证通过了，这时候服务器端基于用户名、密码、当前时间戳等内容，用md5或者des或者aes等加密方式，生成一个token值，然后把token值存放到redis里面，记录它对应哪个用户，然后把这个token值发给客户端。客户端收到token值以后，下次访问服务器端任何接口的时候，直接携带这个token，服务器端就知道它是谁了，该给它什么数据。哪以何种方式给服务器端呢？通常的做法就是把token值放在header里面。当然这个不是绝对，你也可以放在body里面，这个都是仁者见仁智者见智的事。好了，我们就先放在header头里面。**
###  REST主要名词 

**资源（Resources）**
REST的名称"表现层状态转化"中，省略了主语。"表现层"其实指的是"**资源"（Resources）的"表现层"**。
**所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。**它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。你可以用一个URI（统一资源定位符）指向它，每种资源对应一个特定的URI。要获取这个资源，访问它的URI就可以，**因此URI就成了每一个资源的地址或独一无二的识别符。**
所谓"上网"，就是与互联网上一系列的"资源"互动，调用它的URI。
**表现层**（Representation）
**"资源"是一种信息实体，它可以有多种外在表现形式。**我们把"资源"具体呈现出来的形式，叫做它的"表现层"（Representation）。
比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现。
**URI只代表资源的实体，不代表它的形式。**严格地说，有些网址最后的".html"后缀名是不必要的，因为这个后缀名表示**格式，属于"表现层"范畴**，**而URI应该只代表"资源"的位置。**它的具体表现形式，应该在HTTP请求的头信息中用` Accept和Content-Type `字段指定，**这两个字段才是对"表现层"的描述。**

**状态转化（State Transfer）**
访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，势必涉及到数据和状态的变化。
互联网通信协议HTTP协议，是一个无状态协议。这意味着，所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，**让服务器端发生"状态转化"（State Transfer）。而这种转化是建立在表现层之上的，所以就是"表现层状态转化"。**
客户端用到的手段，只能是HTTP协议。具体来说，就是HTTP协议里面，**四个表示操作方式的动词：GET、POST、PUT、DELETE。**它们分别对应四种基本操作：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源。

**综合上面的解释，我们总结一下什么是RESTful架构：**
　　（1）每一个URI代表一种资源；
　　（2）客户端和服务器之间，**传递这种资源的某种表现层（如客服端与服务端之间通过json这一表现层传递资源）**；
　　（3）客户端通过四个HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

RESTful架构最常见的一种设计错误，就是URI包含动词。**因为"资源"表示一种实体，所以应该是名词，URI不应该有动词，**动词应该放在HTTP协议中。

**spring对rest的支持**：
![](/data/dokuwiki/spring/pasted/20150811-025746.png)
###  关于rest uri的理解 
![](/data/dokuwiki/spring/pasted/20150811-025844.png)![](/data/dokuwiki/spring/pasted/20150811-025852.png)
##  编写面向资源的控制器 
在URL中嵌入参数：
```

@Controller
@RequestMapping("/spitters")
public class SpittersController {
  private SpitterService spitterService;
  
  @Inject
  public SpittersController(SpitterService spitterService) {
    this.spitterService = spitterService;
  }
  
  @RequestMapping(value="/{spitterName}/spittles", 
                  method=RequestMethod.GET)
  public @ResponseBody List<Spittle> spittlesForSpitter(
                 @PathVariable("spitterName") String spitterName) {
    Spitter spitter = spitterService.getSpitter(spitterName);
    return spitterService.getSpittlesForSpitter(spitter);
  }
}

```
###  执行REST动作： 
http的四个方法:GET,POST,PUT,DELETE
**使用PUT更新资源：**
```

  @RequestMapping(value = "/{username}", method = RequestMethod.PUT, 
                  headers = "Content-Type=application/json")  //服务端响应内容类型为json
  @ResponseStatus(HttpStatus.NO_CONTENT)    //http响应状态吗204
  public void updateSpitter(@PathVariable String username, 
                        @RequestBody Spitter spitter) {
    spitterService.saveSpitter(spitter);
  }

```
**处理DELETE请求：**
```

  @RequestMapping(value="/{username}", method=RequestMethod.DELETE)
  public String deleteSpitter(@PathVariable String username) {
    return "redirect:/home";
  }

```
**使用POST创建资源：**
```

 @RequestMapping(method=RequestMethod.POST)
  public String addSpitterFromForm(@Valid Spitter spitter, 
      BindingResult bindingResult,
      @RequestParam(value="image", required=false) MultipartFile image) {   
    if(bindingResult.hasErrors()) {
      return "spitters/edit";
    }    
    spitterService.saveSpitter(spitter);   
    try {
      if(!image.isEmpty()) {
        validateImage(image);
        saveImage(spitter.getId() + ".jpg", image);
      }
    } catch (ImageUploadException e) {
      bindingResult.reject(e.getMessage());
      return "spitters/edit";
    }
    return "redirect:/spitters/" + spitter.getUsername();
  }  

```
###  表述资源 
Spring的` ContentNegotiatingViewResolver `是一个特殊的视图解析器，它考虑到了客户端所需的内容类型，**通过配置它会选择最适合的视图**。它需要作为一个bean配置在spring应用上下文中。
![](/data/dokuwiki/spring/pasted/20150811-031352.png)
ContentNegotiatingViewResolver将考虑请求的Accept头部信息并使用它请求的媒体类型，但是它会首先查看URL文件的扩展名。
。。。。
另一种方式：
**使用HTTP信息转换器。**` @ResponseBody,@RequestBody `
![](/data/dokuwiki/spring/pasted/20150811-031949.png)
![](/data/dokuwiki/spring/pasted/20150811-032049.png)
![](/data/dokuwiki/spring/pasted/20150811-032100.png)
![](/data/dokuwiki/spring/pasted/20150811-032110.png)
![](/data/dokuwiki/spring/pasted/20150811-032144.png)
**在请求体中接收资源状态**
![](/data/dokuwiki/spring/pasted/20150811-032404.png)
##  编写REST客户端 
略。。。。
##  提交RESTful表单 
![](/data/dokuwiki/spring/pasted/20150811-032907.png)
**在JSP中渲染隐藏的方法域：**
![](/data/dokuwiki/spring/pasted/20150811-032947.png)
![](/data/dokuwiki/spring/pasted/20150811-033019.png)
**配置过滤器发布真正的请求:**
![](/data/dokuwiki/spring/pasted/20150811-033136.png)
![](/data/dokuwiki/spring/pasted/20150811-033200.png)