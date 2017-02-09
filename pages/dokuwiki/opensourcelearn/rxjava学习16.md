title: rxjava学习16 

#  RxJava学习之操作符学习十三字符串操作 
` StringObservable ` 类包含一些**用于处理字符串序列和流的特殊操作符**，如下：
  * byLine( ) — 将一个字符串的Observable转换为一个行序列的Observable，这个Observable将原来的序列当做流处理，然后按换行符分割
  * decode( ) — 将一个多字节的字符流转换为一个Observable，它按字符边界发射字节数组
  * encode( ) — 对一个发射字符串的Observable执行变换操作，变换后的Observable发射一个在原始字符串中表示多字节字符边界的字节数组
  * from( ) — 将一个字符流或者Reader转换为一个发射字节数组或者字符串的Observable
  * join( ) — 将一个发射字符串序列的Observable转换为一个发射单个字符串的Observable，后者用一个指定的字符串连接所有的字符串
  * split( ) — 将一个发射字符串的Observable转换为另一个发射字符串的Observable，后者使用一个指定的正则表达式边界分割前者发射的所有字符串
  * stringConcat( ) — 将一个发射字符串序列的Observable转换为一个发射单个字符串的Observable，后者连接前者发射的所有字符串

