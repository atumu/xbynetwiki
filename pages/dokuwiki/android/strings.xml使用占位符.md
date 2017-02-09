title: strings.xml使用占位符 

#  android:strings.xml使用占位符 
strings.xml中节点是支持占位符的，如下所示：
` <string name="data">整数型:%1$d，浮点型：%2$.2f，字符串:%3$s</string> `
其中%后面是占位符的位置，**从1开始**，
  * $ 后面是填充数据的类型
  *  %d：表示整数型；
  *  %f ：表示浮点型，其中f前面的.2 表示小数的位数
  *  %s：表示字符串
这些和C语言中输入输出函数的占位符很相似

在程序中我们可以通过下面的代码对字符串进行格式化，也就是填充占位符中的内容：
''String data = getResources().getString(R.string.data); 
data = String.format(data, 100, 10.3, "2011-07-01");''