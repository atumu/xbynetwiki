title: maven资源文件路径 

#  maven资源文件路径 
<fc #ff0000>注意Maven中src/main/resource下的资源,
在执行mvn process-resources的时候会将此文件夹下的东西全部拷贝到classes（对于web工程在WEB-INF/classes）文件夹下.
也就是说打包成jar包时src/main/resource下的资源文件处于jar包根路径下。打包成war包时会在WEB-INF/classes路径下。</fc>
要想读取这些文件只能通过ClassLoader读取：
```

public static final String COMP_XML_CONFIG_PATH="component.xml";
private static final ClassLoader loader=Thread.currentThread().getContextClassLoader();//此处ClassLoader如不正确，则无法读取
InputStream  ins=loader.getResourceAsStream(COMP_XML_CONFIG_PATH);

```
**maven 编译项目是多个源目录（src/main/java、src/main/resources）同时输出到target/classes目录下**，经过maven编译后的src/main/java复制到相应的target/classes下，这里面全都是由src/main/java目录下的java文件编译过来classes字节码文件，**而src/main/java目录下的其它格式的文件，如（.xml、.property...）都不会被复制到target/classes目录，**所以如果在此时用class.getClassLoader().getSystemResourceAsStream("property.xml")是不可能找到配置文件的，本来无一物，何处觅文件？

所以，maven标准的目录结构里提供了src/main/resources目录，用以存放各种配置文件的，同样编译输出到target/classes目录下，所以，我们要找配置文件得在这个目录下去找。

