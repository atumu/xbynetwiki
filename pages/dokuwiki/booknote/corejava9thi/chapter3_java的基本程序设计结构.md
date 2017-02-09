title: chapter3_java的基本程序设计结构 


#  Chapter3 Java的基本程序设计结构 
Created Saturday 07 March 2015
System.out.println; System.out.print

##  注释 
```

// ; /* */; /** */

```
##  数据类型 
Java是一种强类型语言。8种基本类型(primitive type):short int long byte ; char; boolean; double float
注意几点：java中字符串不是基本类型。java中整型的范围与运行java代码的机器无关。
表示溢出或错误情况的三个特殊浮点数：
	正无穷大Double.POSITIVE_INFINITY
	负无穷大Double.NEGATIVE_INFINITY
	NaN：Double.NaN; 这里需要注意NaN的判断：Double.isNaN(4);
注意：浮点数值不能用于直接比较是否相等。只能取定范围后比较是否相等。
	浮点数值不能用于金融或银行业。这些地方禁止出现舍入误差。应该使用java.math.BigDecimal类。

##  变量与常量 
千万不要使用没有初始化的变量。
常量采用final定义。

##  数学函数和常量 
Math.sqrt(x);Math.pow(x,a);....
Math.PI;Math.E
静态导入：import static java.lang.Math.*

##  数值类型之间的转换 
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-061659.png)

##  强制类型转换 

##  枚举类型 
适用于变量的取值只在一个有限的集合内。枚举类型包括有限个命名的值。例如：
enum Size{ BIG,MEDIUM,SMALL}
Size s=Size.BIG;

##  字符串与String 
**注意：字符串一旦被创建就是不可变的，同时共享在堆区的常量共享池中，故而一般不能通过==比较字符串，而应该采用String类的equals()方法**

##  空串与null 
判断字符串非空：注意以下的判断顺序，否则程序会出错。
```

if(str!=null && str.length()!=0){}//千万不要使用str!="",取而代之使用str.length()!=0或者!(str.equals(""))

```
##  构建字符串 
高效率：StringBuilder（线程安全版为StringBuffer）

##  输入输出 
```

读取输入：java.util.Scanner
Scanner in=new Scanner(System.in);
System.out.println("What's your name?");
String name=in.nextLine();
System.out.println("How old are you?");
int old=in.nextInt();
读取密码：Console
Console c=System.console();
String name=c.readLine("name?");
char[] pass=c.readPassword("pass?");

```
###  格式化输出 
System.out.printf()
![](/data/dokuwiki/booknote/corejava9thi/pasted/20150521-061728.png)

###  文件输入与输出 
```

Scanner scan=new Scanner(Paths.get("C:**\\**xby.txt"));//注意windows路径中的双斜杠。
写入文件：PrintWriter writer=new PrintWriter("xby.txt");
获取虚拟机启动路径：String path=System.getProperty("user.dir");

```
##  控制流程 
switch语句：从java7开始允许字符串常量的判断。
```

switch(choice){ //choice允许为char, int, short byte及其包装类型;枚举常量；从java7开始允许字符串字面量。
	case 1:
		..
		break;
	case 2:
		...
		break;
	defalut:
		..
		break;

}
带标签的break
read:
while(){
....
break read;//跳出read标记的循环。执行该循环语句之后的代码。
}
...

```
##  大数值 
BigInteger, BigDecimal
BigInteger bInt=BigInteger.valueOf(100);

##  数组 
数组是一种数据结构，用来存储同一类型的值的集合。
int[] a=new int[100];

###  for each循环 
for(int element : collection) statement 循环集合中的每一个元素。element变量只是在每一次迭代中暂存变量的值。
技巧：Arrays.toString(a);可以将数组转换为字符串以便打印。

###  数组拷贝 
Arrays.copyOf()返回新数组

###  数组排序 
Arrays.sort()

##  随机数 
Math.random()返回0-1不包括1之间的浮点数。
Math.random()*100返回0-99之间的浮点数
int r =(int)(Math.random()*100)返回0-99之间的整数。

###  多维数组 
注意：for each循环不能迭代多维数组，只能使用多个for each循环嵌套迭代。

##  命令行参数 
args[0]代表第一个参数，而不是允许命令java


##  需要学习的API： 
java.math.BigInteger
java.math.BigDecimal
java.lang.String
java.lang.StringBuilder
java.util.Scanner
java.lang.System
java.io.Console
java.io.PrintWriter
java.nio.file.Paths 从JDK7开始提供

