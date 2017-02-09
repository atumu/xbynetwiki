title: hibernate数据类型转换器 

#  Hibernate数据类型转换器 
Maven JPA实现
```

<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${org.hibernate.version}</version>
		</dependency>

```
JPA2.1增加了特性转化器` javax.persistence.AttributeConverter `类。用于自定义转换类型。
为了使转换器工作，还需要使用几个注解。@javax.persistence.Converter。一个实现了AttributeConverter的具体类要么必须标注上@Converter，要么必须在JPA映射文件中指定<converter>元素。**` 此外必须保证该类能被扫描到。 `**
**@Converter的autoApply特性（默认为false）表示JPA提供者是否应该自动将转换器应用在全局的匹配属性上。**
转化器写法：
```

@Converter
public class InstantConverter implements AttributeConverter<Instant, Timestamp>
{
    @Override
    public Timestamp convertToDatabaseColumn(Instant instant)
    {
        return instant == null ? null:new Timestamp(instant.toEpochMilli());
    }

    @Override
    public Instant convertToEntityAttribute(Timestamp timestamp)
    {
        return timestamp == null ? null:Instant.ofEpochMilli(timestamp.getTime());
    }
}


```
如果autoApply的值为false，那么你必须在对应实体的JPA属性上使用@javax.persistence.Converter注解的converter特性指定转换器类。表示转换器应该作用于该属性上。
```

public class MyEntity{
	@Converter(converter=InstantConverter.class) //方式一，直接写在field上
	private Instant dateCreate;
	......
	@Converter(converter=InstantConverter.class) //方式二，写在getXxx上
	public Instant getDateCreated(){...}

//方式三，可以直接写在类上方，略
}

```