title: guava技巧总结 

#  Guava技巧总结 
##  guava 使用hash算法 
使用MD5算法获取摘要,并转换为字符串:
```

Hashing.md5().hashString("sdf");//881710b97e322568d6e8685aa3fbea63

``` 

使用sha256:
```

Hashing.sha256().hashString("sdf");//47e476c029f83f2e8fa32d6687956d3ae4db58815da964985a18b0fb4fe187ca 

``` 

使用sha512:
```

Hashing.sha512().hashString("sdf");//901a1d6fe52ff127c3d6c28a323a6567309a61395d1e132babe19091738c782a6f527c0f456cdf82c2a6b9029f5d13cf287a70123875eca075a8845385561912 

``` 
 
64位操作系统使用 SHA-512运算速度比SHA256更佳
```

HashCode hashCode= Hashing.md5().hashString("test_password", Charsets.UTF_8);  
String md5=hashCode.toString(); 

``` 