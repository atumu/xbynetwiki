title: bcrypt加密用户密码 

#  BCrypt加密用户密码 
官网：http://www.mindrot.org/projects/jBCrypt/
```

<dependency>
    <groupId>org.mindrot</groupId>
    <artifactId>jbcrypt</artifactId>
    <version>0.3m</version>
</dependency>

```
使用：
```

// Hash a password for the first time
String hashed = BCrypt.hashpw(password, BCrypt.gensalt());

// gensalt's log_rounds parameter determines the complexity
// the work factor is 2**log_rounds, and the default is 10
String hashed = BCrypt.hashpw(password, BCrypt.gensalt(12)); //至少为4，值为4-31. 不然报错

// Check that an unencrypted password matches one that has
// previously been hashed
if (BCrypt.checkpw(candidate, hashed))  //比较时不用传入盐。它会自动识别。
	System.out.println("It matches");
else
	System.out.println("It does not match");

```
用户表的密码通常使用MD5等不可逆算法加密后存储，为防止彩虹表破解更会先使用一个特定的字符串（如域名）加密，然后再使用一个随机的salt（盐值）加密。
特定字符串是程序代码中固定的，salt是每个密码单独随机，一般给用户表加一个字段单独存储，比较麻烦。
**` BCrypt算法将salt随机并混入最终加密后的密码，验证时也无需单独提供之前的salt，从而无需单独处理salt问题。 `**(看看加密后哈希值的前几位，与盐对比一下，你就会发现哈希值里面带盐了)

##  Spring Security中使用BCrypt 
API:http://docs.spring.io/spring-security/site/docs/3.1.x/apidocs/org/springframework/security/crypto/bcrypt/BCrypt.html


org.springframework.security.crypto.bcrypt.BCrypt
```

String pw_hash = BCrypt.hashpw(plain_password, BCrypt.gensalt()); 
if (BCrypt.checkpw(candidate_password, stored_hash))
    System.out.println("It matches");
else
    System.out.println("It does not match");

```

The gensalt() method takes an optional parameter (log_rounds) that determines the computational complexity of the hashing:
```

String strong_salt = BCrypt.gensalt(10)
String stronger_salt = BCrypt.gensalt(12)

```
The amount of work increases exponentially (2* *log_rounds), so each increment is twice as much work. ` The default log_rounds is 10, and the valid range is 4 to 31. `