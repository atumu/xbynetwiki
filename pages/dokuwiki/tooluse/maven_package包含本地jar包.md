title: maven_package包含本地jar包 

#  maven package包含本地jar包 
```

<build>
        <plugins>
            <plugin>
              <artifactId>maven-compiler-plugin</artifactId>
              <configuration>
                  <source>1.6</source>
                  <target>1.6</target>
                  <encoding>UTF-8</encoding>
                  <compilerArguments>
                   <!-- <extdirs>src\main\webapp\WEB-INF\lib</extdirs> -->
                     <extdirs>${project.basedir}/src/main/webapp/WEB-INF/lib</extdirs>
                 </compilerArguments>
              </configuration>
            </plugin>
        </plugins>
    </build>

```

参考：http://bbs.csdn.net/topics/390379229
http://www.mamicode.com/info-detail-169419.html
http://l-x.iteye.com/blog/1853655