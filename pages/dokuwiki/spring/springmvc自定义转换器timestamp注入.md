title: springmvc自定义转换器timestamp注入 

#  SpringMVC自定义转换器实现Timestamp参数注入 
默认情况下如果接受前台数据的Vo类或者Dto类中有Timestamp类型属性,那么请求时SpringMVC会直接返回400代码,而不会执行Controller方法.这是因为无法解析参数导致的.这种情况下我们需要自定义转换器.
```

public class TimestampConverter implements Converter<String, Timestamp>{
 
        @Override
    public Timestamp convert(String timeStr) {
        // TODO Auto-generated method stub
        Timestamp t=null;
        if(StringUtils.checkNotEmpty(timeStr)){
            long time=Long.valueOf(timeStr);
            t=new Timestamp(time);
        }
        return t;
    }
 
 
}

```
```

<mvc:annotation-driven  conversion-service="conversionService"/> 
<bean id="conversionService"
            class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="net.xby1993.springmvc.util.TimestampConverter"></bean>
            </list>
        </property>
    </bean>

```
```

@RequestMapping("/saveOrUpdate")
    @ResponseBody
    public Map<String,Object> update(HttpServletRequest req,SiteNoticeDto dto,String noticeId){
}

```
```

public class SiteNoticeDto {
    private Timestamp publishTime;
    private String noticeTitle;
    private String noticeTitleColor;
    private String noticeContent;
    private String noticeAuthor;
................................
}

```