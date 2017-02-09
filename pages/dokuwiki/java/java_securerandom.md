title: java_securerandom 

#  java SecureRandom 
有以下两种请求 SecureRandom 对象的方法：仅指定算法名称，或者既指定算法名称又指定包提供程序。
如果仅指定算法名称，如下所示：
SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
 
系统将确定环境中是否有所请求的算法实现，是否有多个，是否有首选实现。
 
如果既指定了算法名称又指定了包提供程序，如下所示：
SecureRandom random = SecureRandom.getInstance("SHA1PRNG", "SUN");
 
系统将确定在所请求的包中是否有算法实现；如果没有，则抛出异常。
SecureRandom 实现尝试完全随机化生成器本身的内部状态，除非调用方在调用 getInstance 方法之后又调用了 setSeed 方法：
SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
random.setSeed(seed);
 
在调用方从 getInstance 调用中获得 SecureRandom 对象之后，它可以调用 nextBytes 来生成随机字节：
byte bytes[] = new byte[20];
random.nextBytes(bytes);
 
调用方还可以调用 generateSeed 方法来生成给定的种子字节数（例如，为其他随机数量生成器提供种子）：
byte seed[] = random.generateSeed(20);