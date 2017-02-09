title: springmvc之转换器与格式化 

#  springmvc之转换器与格式化 
可供参考资料：
http://www.cnblogs.com/liukemng/p/3748137.html
http://jinnianshilongnian.iteye.com/blog/1729739

在系列（6）中我们介绍了如何验证提交的数据的正确性，当数据验证通过后就会被我们保存起来。保存的数据会用于以后的展示，这才是保存的价值。那么在展示的时候如何按照要求显示？（**比如：小数保留一定的位数，日期按指定的格式等**）。这就是本篇要说的内容—>格式化显示。
从Spring3.X开始，Spring提供了` Converter ` SPI类型转换和` Formatter ` SPI字段解析/格式化服务，其中Converter SPI实现**对象与对象之间**的相互转换，Formatter SPI实现**String与对象之间**的转换，Formatter SPI是对Converter SPI的封装并添加了对国际化的支持，其内部转换还是由Converter SPI完成。Formatter更适合web层，Converter更为通用。

下面是一个简单的请求与模型对象的转换流程：
![](/data/dokuwiki/spring/pasted/20150812-045229.png)
##  使用内置的注解进行字段级别的解析/格式化 

```

import java.util.Date;
import org.springframework.format.annotation.DateTimeFormat;
import org.springframework.format.annotation.NumberFormat;
import org.springframework.format.annotation.NumberFormat.Style;

public class FormatModel{
     @NumberFormat(style=Style.NUMBER, pattern="#,###")  
    private int totalCount;   
    @NumberFormat(style=Style.CURRENCY)
　　 private double money;
    @DateTimeFormat(pattern="yyyy-MM-dd HH:mm:ss")
    private Date date;
    
    public double getMoney(){
        return money;
    }
    public Date getDate(){
        return date;
    }
    
    public void setMoney(double money){
        this.money=money;
    }
    public void setDate(Date date){
        this.date=date;
    }
        
}

```
**@Number**：定义数字相关的解析/格式化元数据（通用样式、货币样式、百分数样式），参数如下：
style：用于指定样式类型，包括三种：Style.NUMBER（通用样式） Style.CURRENCY（货币样式） Style.PERCENT（百分数样式），默认Style.NUMBER；
pattern：自定义样式，如patter="#,###"；
 
**@DateTimeFormat**：定义日期相关的解析/格式化元数据，参数如下：
pattern：指定解析/格式化字段数据的模式，如”yyyy-MM-dd HH:mm:ss”
iso：指定解析/格式化字段数据的ISO模式，包括四种：ISO.NONE（不使用）  ISO.DATE(yyyy-MM-dd) ISO.TIME(hh:mm:ss.SSSZ)  ISO.DATE_TIME(yyyy-MM-dd hh:mm:ss.SSSZ)，默认ISO.NONE；
style：指定用于格式化的样式模式，默认“SS”，具体使用请参考Joda-Time类库的org.joda.time.format.DateTimeFormat的forStyle的javadoc；
优先级： pattern 大于 iso 大于 style。

##  自定义Formatter进行解析/格式化 
使用org.springframework.format.Formatter接口
public interface Formatter<T> T表示输入字符串要转换的目标类型。该接口有parse和print两个方法。parse将string转换为对象，而print与之相反。
```

import org.springframework.format.Formatter;

public class DateFormatter implements Formatter<Date> {

    private String datePattern;
    private SimpleDateFormat dateFormat;

    public DateFormatter(String datePattern) {   //构造器传入想要的日期格式，不依赖于Locale
        System.out.println("DateFormatter()5b### ");
        this.datePattern = datePattern;
        dateFormat = new SimpleDateFormat(datePattern);
        dateFormat.setLenient(false);
    }

    @Override
    public String print(Date date, Locale locale) {
        return dateFormat.format(date);
    }

    @Override
    public Date parse(String s, Locale locale) throws ParseException {
        try {
            return dateFormat.parse(s);
        } catch (ParseException e) {
            // the error message will be displayed when using <form:errors>  这段信息会被显示到<form:errors>
            throw new IllegalArgumentException(
                    "invalid date format. Please use this pattern\""
                            + datePattern + "\"");
        }
    }
}

```
**然后需要在springmvc配置文件中进行注册**：
```

<mvc:annotation-driven conversion-service="conversionService" />
<bean id="conversionService"
		class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="formatters">
			<set>
				<bean class="app06b.formatter.DateFormatter">
					<constructor-arg type="java.lang.String" value="MM-dd-yyyy" />
				</bean>
			</set>
		</property>
	</bean>

```
##  自定义Converter进行解析/格式化 
` Java标准的PropertyEditor `的核心功能是将一个**字符串转换为一个Java对象**，以便根据界面的输入或配置文件中的配置字符串构造出一个JVM内部的java对象。
如何注册自定义的属性编辑器：
1、实现PropertyEditor接口或者继承PropertyEditorSupport类
2、在Spring上下文中声明一个org.springframework.beans.factory.config.CustomEditorConfigurer的bean
```

<!--将bean1中的Date赋值2008-08-15，spring会认为2008-08-15是String，无法转换成Date，会报错！-->
<bean id="bean1" class="com.bjsxt.spring.Bean1">      
             <property name="dateValue">
                       <value>2008-08-15</value>
             </property>
</bean>

```
```

<!-- 于是定义属性编辑器 -->      
<bean id="customEditorConfigurer"           class="org.springframework.beans.factory.config.CustomEditorConfigurer">
           <property name="customEditors">
                 <map>  
                     <entry key="java.util.Date">           
                          <bean class="com.bjsxt.spring.UtilDatePropertyEditor">           
                              <!--干脆把format也注入，灵活处理格式-->
                              <property name="format" value="yyyy-MM-dd"/>
                             </bean>
                     </entry>
                 </map>
           </property>
</bean>

```
**但是Java原生的PropertyEditory存在一下不足：**
1、只能用于字符串和java对象的抓换，不适用于任意两个Java类型之间的转换。
2、PropertyEditor是线程不安全的，也就是有状态的，因此每次使用时都需要创建一个，不可重用；
3、PropertyEditor不是强类型的，setValue（Object）可以接受任意类型，因此需要我们自己判断类型是否兼容；
4、对源对象和目标对象上下文信息（如注解，所在宿主类的结构等）不敏感，在类型转换时**不能利用这些上下文信息实施高级转换逻辑。**
有鉴于此，**Spring 3.0核心模型中添加了一个通用的类型转换模块，类型转换模块位于` org.springframework.core.convert `中，**Spring希望用这个类型转换体系替换Java标准的PropertyEditor。但由于历史原因，会同时支持两者。
在Spring Web MVC环境中，数据类型转换、验证及格式化通常是这样使用的：
![](/data/dokuwiki/spring/pasted/20151130-221908.png)
一个有如下三种接口：
1、Converter：类型转换器，用于转换S类型到T类型，此接口的实现**必须是线程安全的且可以被共享。**
2、GenericConverter和ConditionalGenericConverter：GenericConverter接口实现能在多种类型之间进行转换，ConditionalGenericConverter是有条件的在多种类型之间进行转换。
3、ConverterFactory：工厂模式的实现，用于选择将一种S源类型转换为R类型的子类型T的转换器的工厂接口。
我们先来看一下 Converter 接口的定义：
```

public interface Converter<S, T> {
    T convert(S source);
}

```

` ConversionService `
它是Spring类型转换体系中**核心接口**，位于org.springframework.core.convert中，也是唯一一个接口。
可以利用` ConversionServiceFactorybean `在spring上下文中定义一个` ConversionService `。该factorybean提供了**大量内置的转换器**，同时也支持自定义转换器。
**自定义转换器**，可选择实现三者中任一种接口：
  * Converter<S,T>
  * GenericConverter
  * ConverterFactory
Spring mvc在支持新转换器框架同时，也支持JavaBeans的PropertyEditor，可以在控制器中使用@InitBinder添加自定义的编辑器，也可以通过WebBindingInitializer装配在全局范围内使用的编辑器。

**加载顺序**
那么，对于同一个类型对象来说，如果三个地方均配置了，该以哪个为主：
1、查询通过@Initbinder装配的自定义编辑器。
2、查询通过ConversionService装配的自定义转换器。
3、查询通过WebBindingInitializer装配的自定义编辑器。

**Converter<S, T> S代表源对象，T代表要转换的目标对象。** 
```

import org.springframework.core.convert.converter.Converter;

public class StringToDateConverter implements Converter<String, Date> {

    private String datePattern;

    public StringToDateConverter(String datePattern) {
        this.datePattern = datePattern;
    }

    @Override
    public Date convert(String s) {
        try {
            SimpleDateFormat dateFormat = new SimpleDateFormat(datePattern);
            dateFormat.setLenient(false);
            return dateFormat.parse(s);
        } catch (ParseException e) {
            // the error message will be displayed when using <form:errors>
            throw new IllegalArgumentException(
                    "invalid date format. Please use this pattern\""
                            + datePattern + "\"");
        }
    }
}


```
**然后在springmvc中进行注册**：
```

 <mvc:annotation-driven conversion-service="conversionService"/>
 <bean id="conversionService" 
            class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="app06a.converter.StringToDateConverter">
                    <constructor-arg type="java.lang.String" value="MM-dd-yyyy"/>
                </bean>
            </list>
        </property>
    </bean>

```
参考：http://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/validation.html#format-configuring-formatting-mvc
http://www.cnblogs.com/beiyeren/p/3904529.html
http://www.tuicool.com/articles/uUjaum