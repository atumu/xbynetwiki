title: mybatis中传参there_is_no_getter_for_property_named_xxx_in_class_java.lang.string 

#  Mybatis中传参There is no getter for property named 'XXX' in 'class java.lang.String' 
一、发现问题
```

<select id="queryStudentByNum" resultType="student" parameterType="string">  
select num,name,phone from student  
<where> 
<if test = " num!=null and num!='' ">
AND num = #{num}
</if>
</where>
</select>

``` 
Mybatis查询传入一个字符串传参数，报There is no getter for property named 'num' in 'class java.lang.String'。


二、解决问题
```

<select id="queryStudentByNum" resultType="student" parameterType="string">  
select num,name,phone from student  
<where> 
<if test = " _parameter!=null and_parameter!='' ">
AND num = #{_parameter}
</if>
</where>
</select>

```
` 无论参数名，都要改成"_parameter"。 `


三、原因分析
Mybatis默认**采用ONGL解析参数**，所以会自动采用对象树的形式取string.num值，引起报错。` 也可以public List methodName(@Param(value="num") String num)的方法说明参数值 `

参考博客：
http://blog.csdn.net/woshixuye/article/details/8820387
http://blog.sina.com.cn/s/blog_86e49b8f010191hw.html
http://txin0814.iteye.com/blog/1533645
