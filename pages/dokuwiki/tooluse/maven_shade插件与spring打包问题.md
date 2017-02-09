title: maven_shade插件与spring打包问题 

#  Maven Shade插件与Spring打包报错问题 
报错：Unable to locate Spring NamespaceHandler for XML schema namespace
Exception in thread "main" org.springframework.beans.factory.parsing.BeanDefini
ionParsingException: Configuration problem: Unable to locate Spring NamespaceHa
dler for XML schema namespace [http://www.springframework.org/schema/aop]
Offending resource: class path resource [applicationContext.xml]
解决思路：
问题的所在，在我的jar包下的META-INF目录下，有两个跟spring相关的文件：spring.handlers、spring.schemas，打开这两个文件一看，里面都只包含了spring-tx的配置
里面并没有spring-aop的schema和handler配置，所以会报错。
那么问题的根源是什么呢，在stackoverflow上找到了答案：http://stackoverflow.com/questions/1937767/spring-3-0-unable-to-locate-spring-namespacehandler-for-xml-schema-namespace

因为我使用了maven-shade-plugin这个maven打包插件，主要原因是插件配置不当导致，我原来的配置如下：
```

<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-shade-plugin</artifactId>  
    <version> 1.7.1</version>  
    <executions>  
        <execution>  
            <phase>package</phase>  
            <goals>  
                <goal>shade</goal>  
            </goals>  
            <configuration>  
                <transformers>  
                    <transformer  
                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">  
                        <mainClass>com.chenzhou.test.Main</mainClass>  
                    </transformer>  
                </transformers>  
            </configuration>  
        </execution>  
    </executions>  
</plugin> 

``` 
由于没有配置META-INF/spring.handlers和META-INF/spring.schemas所以如果工程中依赖了Spring的多个依赖，在打包时后面的会把前面的覆盖，使得这两个文件中永远只保存最后一个spring依赖的schema和handler。
解决方法就是在里面加上META-INF/spring.handlers和META-INF/spring.schemas的配置：
```

<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-shade-plugin</artifactId>  
    <version> 1.7.1</version>  
    <executions>  
        <execution>  
            <phase>package</phase>  
            <goals>  
                <goal>shade</goal>  
            </goals>  
            <configuration>  
                <transformers>  
                    <transformer  
                        implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
                        <resource>META-INF/spring.handlers</resource>  
                    </transformer>  
                    <transformer  
                        implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
                        <resource>META-INF/spring.schemas</resource>  
                    </transformer>  
                    <transformer  
                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">  
                        <mainClass>com.chenzhou.test.Main</mainClass>  
                    </transformer>  
                </transformers>  
            </configuration>  
        </execution>  
    </executions>  
</plugin> 

``` 
和第一段配置对比，主要增加了：
```

<transformer  
    implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
    <resource>META-INF/spring.handlers</resource>  
</transformer>  
<transformer  
    implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">  
    <resource>META-INF/spring.schemas</resource>  
</transformer> 

``` 
这段配置意思是把spring.handlers和spring.schemas文件以append方式加入到构建的jar包中。
修改完后，再次打包，此时就会把工程依赖的所有的spring依赖的schema和handler都加载到spring.handlers和spring.schemas里面。加载applicationContext.xml时就不会报错了。
参考：http://chenzhou123520.iteye.com/blog/1971322
http://chenzhou123520.iteye.com/blog/1706242