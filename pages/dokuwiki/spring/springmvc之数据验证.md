title: springmvc之数据验证 

#  springmvc之数据验证 
可以参考：http://jinnianshilongnian.iteye.com/blog/1733708 http://www.cnblogs.com/liukemng/p/3738055.html  系列springmvc文章
` **目前SpringMVC的验证机制普遍采用支持JSR-303和JSP-349标准的Hibernate-Validator** `，而非其本身自带的验证体系。
这里我们采用` Hibernate-validator `来进行验证，Hibernate-validator实现了` JSR-303 `验证框架支持注解风格的验证。

##  第一步：添加Hibernate-validator依赖 
首先我们要到http://hibernate.org/validator/下载需要的jar包，这里以4.3.1.Final作为演示，
解压后把` hibernate-validator-4.3.1.Final.jar、jboss-logging-3.1.0.jar、validation-api-1.0.0.GA.jar `这三个包添加到项目中。
Maven:
```

	<dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>1.1.0.Final</version>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>5.2.2.Final</version>
        </dependency>

```
##  第二步：Spring配置总添加对JSR-303验证框架的支持 
主要配置项：
验证器工厂
验证消息的本地化

**配置项目中的springmvc配置文件**，如下：
```

<!-- 默认的注解映射的支持 -->  
    <mvc:annotation-driven validator="validator" conversion-service="conversion-service" />
    
    <bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
        <property name="providerClass"  value="org.hibernate.validator.HibernateValidator"/>
        <!--不设置则默认为classpath下的 ValidationMessages.properties -->
        <property name="validationMessageSource" ref="validatemessageSource"/>
    </bean>
    <bean id="conversion-service" class="org.springframework.format.support.FormattingConversionServiceFactoryBean" />
    <bean id="validatemessageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">  
        <property name="basename" value="classpath:validatemessages"/>  
        <property name="fileEncodings" value="utf-8"/>   
       <property name="cacheSeconds" value="-1"/>   
    </bean> 

```
其中<property name="basename" value="classpath:validatemessages"/>中的classpath:validatemessages为注解验证消息所在的文件validatemessages.properties，需要我们在resources文件夹下添加。
##  第三步：Controller编写与@Valid注解 
主要接口和类:
@javax.validation.Valid
org.springframework.validation.Errors
BindingResult

在com.demo.web.controllers包中添加一个ValidateController.java内容如下：`  @Valid `对需要校验的对象进行标注
```

@Controller
@RequestMapping(value = "/validate")
public class ValidateController {
    
    @RequestMapping(value="/test", method = {RequestMethod.GET})
    public String test(Model model){

        if(!model.containsAttribute("contentModel")){
            model.addAttribute("contentModel", new ValidateModel());
        }
        return "validatetest";
    }
    
    @RequestMapping(value="/test", method = {RequestMethod.POST})
    public String test(Model model, @Valid @ModelAttribute("contentModel") ValidateModel validateModel, BindingResult result) throws NoSuchAlgorithmException{
        
        //如果有验证错误 返回到form页面
        if(result.hasErrors())
            return test(model);
        return "validatesuccess";     
    }
    
}

```
其中` @Valid @ModelAttribute `("contentModel") ValidateModel validateModel的@Valid 意思是在把数据绑定到@ModelAttribute("contentModel") 后就进行验证。
` **注意@Valid之后必须紧跟着BindingResult或者Errors参数。否则会导致异常。** `
##  第四步编写需要被检验的领域模型类 
验证约束都在javax.validation.constraints包中

在com.demo.web.models包中添加一个ValidateModel.java内容如下：
```

mport javax.validation.constraints.Past;
import javax.validation.constraints.Size;
import javax.validation.constraints.NotNull; 
public class Product implements Serializable {
    private static final long serialVersionUID = 78L;

    @Size(min=1, max=10, message="{username.not.empty})
    private String name;

   @NotNull(message="{username.not.empty}")  
    private String username;  
//验证注解的元素值是Email
@Email
private String value;
 //验证注解的元素值与指定的正则表达式匹配.如常见的用户名，以字母或下划线开头，后边可以跟字母数字下划线，长度在5-20之间
  @Pattern(regexp = "^[a-zA-Z_][\\w]{4,19}$", message="{user.name.error}")
  private String value;
 

```
##  第五步编写验证错误信息属性文件 

在注解验证消息所在的文件即` validatemessages.properties `文件中添加以下内容：
age.not.inrange=\u5E74\u9F84\u8D85\u51FA\u8303\u56F4\u3002
email.not.correct=\u90AE\u7BB1\u5730\u5740\u4E0D\u6B63\u786E\u3002
email.not.empty=\u7535\u5B50\u90AE\u4EF6\u4E0D\u80FD\u60DF\u6050\u3002
**其中name.not.empty等分别对应了ValidateModel.java文件中message=”xxx”中的xxx名称**，后面的内容是在输入中文是自动转换的ASCII编码，当然你也可以直接把xxx写成提示内容，而不用另建一个validatemessages.properties文件再添加，但这是不正确的做法，因为这样硬编码的话就没有办法进行国际化了。
**如果没有指定message属性，则prop文件的key为constraint.object.property形式：如Pattern.product.name**
在views文件夹中添加validatetest.jsp和validatesuccess.jsp两个视图，内容分别如下：
```

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>

<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    <form:form modelAttribute="contentModel" method="post">     
        
        <form:errors path="*"></form:errors><br/><br/>
            
        name：<form:input path="name" /><br/>
        <form:errors path="name"></form:errors><br/>
        
        age：<form:input path="age" /><br/>
        <form:errors path="age"></form:errors><br/>
        
        email：<form:input path="email" /><br/>
        <form:errors path="email"></form:errors><br/>

        <input type="submit" value="Submit" />
        
    </form:form>  
</body>
</html>

```
```

<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    验证成功！
</body>
</html>

```
其中特别要指出的是validatetest.jsp视图中<form:form modelAttribute="contentModel" method="post">的modelAttribute="xxx"后面的名称xxx必须与对应的@Valid @ModelAttribute("xxx") 中的xxx名称一致，否则模型数据和错误信息都绑定不到。
<form:errors path="name"></form:errors>即会显示模型对应属性的错误信息**，当path="*"时则显示模型全部属性的错误信息。**

##  使用@Valid实现递归/级联验证 
 @NotNull
 @Valid
 private People people;

标注接口而非实现。

##  编写自己的验证约束 
  * @Target多了个ElementType.ANNOTATION_TYPE,
  * @Constraint(validatedBy = {MyValidator.class})表示是一个Bean验证注解，而且可以指定自定义验证器。
  * @Pattern表示该注解约束将继承@Pattern中声明的限制。
  * @ReportAsSingleViolation表示复合限制应该被认为是一个限制。
  * 验证注解有3个必须特性:message,groups,payload.

@Email.List内部注解定义了在目标制定多个@Email限制的一种方式
       
```

@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE,
        ElementType.CONSTRUCTOR, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = {})
@Pattern(regexp = "^[a-z0-9`!#$%^&*'{}?/+=|_~-]+(\\.[a-z0-9`!#$%^&*'{}?/+=|" +
        "_~-]+)*@([a-z0-9]([a-z0-9-]*[a-z0-9])?)+(\\.[a-z0-9]" +
        "([a-z0-9-]*[a-z0-9])?)*$", flags = {Pattern.Flag.CASE_INSENSITIVE})
@ReportAsSingleViolation
public @interface Email
{
    String message() default "{com.wrox.site.validation.Email.message}";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    @Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE,
            ElementType.CONSTRUCTOR, ElementType.PARAMETER})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    static @interface List {
        Email[] value();
    }
}

```
###  自定义验证器 
```

@SuppressWarnings("unused")
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE,
        ElementType.CONSTRUCTOR, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = {NotBlankValidator.class})
@NotNull
@ReportAsSingleViolation
public @interface NotBlank
{
    String message() default "{com.wrox.site.validation.NotBlank.message}";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    @Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE,
            ElementType.CONSTRUCTOR, ElementType.PARAMETER})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    static @interface List {
        NotBlank[] value();
    }
}


```
```

public class NotBlankValidator
        implements ConstraintValidator<NotBlank, CharSequence>
{
    @Override
    public void initialize(NotBlank annotation)
    {

    }

    @Override
    public boolean isValid(CharSequence value, ConstraintValidatorContext context)
    {
        if(value instanceof String)
            return ((String) value).trim().length() > 0;
        return value.toString().trim().length() > 0;
    }
}

```
##  主要的Hibernate-Validate验证注解: 
![](/data/dokuwiki/spring/pasted/20150812-084641.png)![](/data/dokuwiki/spring/pasted/20150812-084710.png)
更多信息请参考官方文档：http://docs.jboss.org/hibernate/validator/4.3/reference/en-US/html/validator-usingvalidator.html

