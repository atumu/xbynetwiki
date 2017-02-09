title: maven_skiptest 

#  maven skiptest 
http://maven.apache.org/surefire/maven-surefire-plugin/examples/skipping-test.html
To skip running the tests for a particular project, set the skipTests property to true.

<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.19.1</version>
        <configuration>
          <skipTests>true</skipTests>
        </configuration>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
You can also skip the tests via the command line by executing the following command:

mvn install -DskipTests
If you absolutely must, you can also use the maven.test.skip property to skip compiling the tests. maven.test.skip is honored by Surefire, Failsafe and the Compiler Plugin.

mvn install -Dmaven.test.skip=true