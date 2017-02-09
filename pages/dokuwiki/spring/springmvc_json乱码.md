title: springmvc_json乱码 

#  SpringMVC JSON乱码解决 
参考：http://blog.csdn.net/os4key/article/details/38729345?utm_source=tuicool
SpringMVC的` @ResponseBody `注解可以将请求方法返回的对象直接转换成JSON对象，
但是**当返回值是String的时候，中文会乱码**，原因是因为其中**字符串转换和对象转换用的是两个转换器**，而**String的转换器中固定了转换编码为"ISO-8859-1"**

1.org.springframework.http.converter.` StringHttpMessageConverter `类是处理请求或相应字符串的类，并且` 默认字符集为ISO-8859-1 `，所以**在当返回json中有中文时会出现乱码。**
2. StringHttpMessageConverter的父类里有个List<MediaType> ` supportedMediaTypes `属性，用来存放 StringHttpMessageConverter支持需特殊处理的 MediaType 类型，如果需处理的 MediaType 类型不在 supportedMediaTypes列表中，则采用默认字符集。
3.解决办法，只需在配置文件中加入如下代码：
```

<!-- springmvc传json值时的乱码解决 -->
  <mvc:annotation-driven>
      <mvc:message-converters>
          <bean class="org.springframework.http.converter.StringHttpMessageConverter">
              <property name="supportedMediaTypes">
                  <list>
                    <value>text/plain;charset=UTF-8</value>
                  </list>
              </property>
          </bean>
      </mvc:message-converters>
  </mvc:annotation-driven>

```
4.如果需要处理其他 MediaType 类型，可在list标签中加入其他value标签

（也可以一个一个地配置，不过有点麻烦。如下
只需要在@RequestMapping中加上` produces="text/plain;charset=UTF-8" `即可
```

@RequestMapping(value="/me",produces="text/plain;charset=UTF-8")
	public @ResponseBody String getMeiNvImg(String num){
		String httpUrl = "http://baidu.com";
		String httpArg = "num="+num;
		String jsonResult = request(httpUrl, httpArg);
		return jsonResult;
		
	}

```
）