title: js高级程序设计3rd学习11 

#  JS高级程序设计3rd学习11最佳实践 
##  1、尊重对象所有权： 
  * 不要为实例或原型添加属性
  * 不要为实例或原型添加方法
  * 不要重定义已存在的方法。
##  2、避免全局量 
避免使用多个全局变量，**可以使用一个全局对象包括所有配置。**例如
```

var MyApplication={
	name:"",
	method1:function(){
	}
}

```
而不是定义多个全局变量。
##  3、使用常量 
比如location.href="http://localhost/index.php"
可以改为使用常量：
```

var Constants={
	LOCAL_HOST:"http://localhost/index.php"
}
location.href=Constants.LOCAL_HOST

```
##  4、使用innerHTML 
略。。。
##  5、使用JSLint进行验证 

##  6、压缩JS代码文件 
使用YUI压缩。

```

<!-- 正式生产环境 -->
		<profile>
			<id>product</id>
			<properties>
				<profiles.active>product</profiles.active>
			</properties>
			<build>
			<plugins>
			  <!-- YUI Compressor Maven压缩插件 -->			
		      <plugin>
		        <groupId>net.alchim31.maven</groupId>
		        <artifactId>yuicompressor-maven-plugin</artifactId>
		        <version>1.5.1</version> 
				<executions>
					<execution>
						<phase>compile</phase>
						<goals>
							<goal>compress</goal>
						</goals>
					</execution>
				</executions>		        
				<configuration>
					<!-- 读取js,css文件采用UTF-8编码 -->
					<encoding>UTF-8</encoding>
					<!-- 不显示js可能的错误 -->
					<jswarn>false</jswarn>
					<!-- 若存在已压缩的文件，会先对比源文件是否有改动有改动便压缩，无改动就不压缩 -->
					<force>false</force>
					<!-- 在指定的列号后插入新行 -->
					<linebreakpos>-1</linebreakpos>
					<!-- 压缩之前先执行聚合文件操作 -->
					<preProcessAggregates>false</preProcessAggregates>
					<!-- 压缩后保存文件后缀 -->
					<!--<suffix>.min</suffix> -->
					<nosuffix>true</nosuffix>
					<!-- 源目录，即需压缩的根目录 -->
					<sourceDirectory>${project.basedir}/src/main/webapp/static</sourceDirectory>
					<!-- 压缩js和css文件 -->
					<includes>
						<include>**/*.js</include>
					</includes>
					<!-- 以下目录和文件不会被压缩 -->
					<excludes>
						<exclude>**/*.min.js</exclude>
						<exclude>**/*.debug.js</exclude>
						<exclude>**/*.min.css</exclude>
						<exclude>**/*.css</exclude>
					</excludes>
					<!-- 压缩后输出文件目录 -->
					<outputDirectory>${project.basedir}/target/js/static</outputDirectory>
				</configuration>
		      </plugin>	
			  <plugin>
			    <groupId>org.apache.maven.plugins</groupId>
			    <artifactId>maven-war-plugin</artifactId>
			    <version>2.6</version>				    
			    <configuration>
					<webResources>  
			            <resource>  
			                <directory>target/js</directory>  
			            </resource>  
			        </webResources> 
			    </configuration>
			  </plugin>			      
		      </plugins>		
			</build>
		</profile>

```
