title: sql组装类 

#  mybatis:SQL组装类 

#  SelectBuilder 

一个 Java 程序员面对的最痛苦的事情之一就是在 Java 代码中嵌入 SQL 语句。 正如你已经 看到的,MyBatis 在它的 XML 映射特性中有处理生成动态 SQL 的很强大的方案。然而,有 时必须在 Java 代码中创建 SQL 语句的字符串。这种情况下,MyBatis 有另外一种特性来帮 助你,在减少典型的加号,引号,新行,格式化问题和嵌入条件来处理多余的逗号或 AND 连接词之前,事实上,在 Java 代码中动态生成 SQL 就是一个噩梦。
SelectBuilder 使用了静态引入和 ThreadLocal 变量的组合来开 启简洁的语法可以很容易地用条件进行隔行扫描,而且为你保护所有 SQL 的格式。它允许 你创建这样的方法:
先静态导入
```

import static org.apache.ibatis.jdbc.SelectBuilder.*;  
[java] view plaincopy
public String selectBlogsSql() {  
  BEGIN(); // Clears ThreadLocal variable  
  SELECT("*");  
  FROM("BLOG");  
  return SQL();  
}  

private String selectPersonSql() {  
  BEGIN(); // Clears ThreadLocal variable  
  SELECT("P.ID, P.USERNAME, P.PASSWORD, P.FULL_NAME");  
  SELECT("P.LAST_NAME, P.CREATED_ON, P.UPDATED_ON");  
  FROM("PERSON P");  
  FROM("ACCOUNT A");  
  INNER_JOIN("DEPARTMENT D on D.ID = P.DEPARTMENT_ID");  
  INNER_JOIN("COMPANY C on D.COMPANY_ID = C.ID");  
  WHERE("P.ID = A.ID");  
  WHERE("P.FIRST_NAME like ?");  
  OR();  
  WHERE("P.LAST_NAME like ?");  
  GROUP_BY("P.ID");  
  HAVING("P.LAST_NAME like ?");  
  OR();  
  HAVING("P.FIRST_NAME like ?");  
  ORDER_BY("P.ID");  
  ORDER_BY("P.FULL_NAME");  
  return SQL();  
}  
使用条件语句
private String selectPersonLike(Person p){  
  BEGIN(); // Clears ThreadLocal variable  
  SELECT("P.ID, P.USERNAME, P.PASSWORD, P.FIRST_NAME, P.LAST_NAME");  
  FROM("PERSON P");  
  if (p.id != null) {  
  WHERE("P.ID like #{id}");  
  }  
  if (p.firstName != null) {  
  WHERE("P.FIRST_NAME like #{firstName}");  
  }  
  if (p.lastName != null) {  
  WHERE("P.LAST_NAME like #{lastName}");  
  }  
  ORDER_BY("P.LAST_NAME");  
  return SQL();  
}  

```
SelectBuilder 对理解哪里放置“WHERE” ,哪里应该使用“AND”还有所有的字符串 连接都是很小心的。
有两个方法会吸引你的眼球:BEGIN()和 SQL()。总之,每个 SelectBuilder 方法应该以 调用 BEGIN()开始,以调用 SQL()结束。当然你可以在中途提取方法来打断你执行的逻 辑,但是 SQL 生成的范围应该以 ` BEGIN()方法开始而且以 SQL()方法结束。BEGIN()方法清 理 ThreadLocal 变量,来确保你不会不小心执行了前面的状态,而且 SQL()方法会基于这些 调用, 从最后一次调用 BEGIN()开始组装你的 SQL 语句。 ` 注意 BEGIN()有一个称为 RESET() 的代替方法,它们所做的工作相同,只是 RESET()会在特定上下文中读取的更好。

#  SqlBuilder 

和 SelectBuilder 相似,MyBatis 也包含一个一般性的 SqlBuilder。它包含 SelectBuilder 的所有方法,还有构建 insert,update 和 delete 的方法。
```

public String deletePersonSql() {  
  BEGIN(); // Clears ThreadLocal variable  
  DELETE_FROM("PERSON");  
  WHERE("ID = ${id}");  
  return SQL();  
}  
  
public String insertPersonSql() {  
  BEGIN(); // Clears ThreadLocal variable  
  INSERT_INTO("PERSON");  
  VALUES("ID, FIRST_NAME", "${id}, ${firstName}");  
  VALUES("LAST_NAME", "${lastName}");  
  return SQL();  
}  
  
public String updatePersonSql() {  
  BEGIN(); // Clears ThreadLocal variable  
  UPDATE("PERSON");  
  SET("FIRST_NAME = ${firstName}");  
  WHERE("ID = ${id}");  
  return SQL();  
}  

```