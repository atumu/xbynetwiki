title: java反射初步 

#  java反射初步 
先看一下反射的概念：
主要是指程序可以访问，检测和修改它本身状态或行为的一种能力，并能根据自身行为的状态和结果，调整或修改应用所描述行为的状态和相关的语义。
反射是java中一种强大的工具，能够使我们很方便的创建灵活的代码，这些代码可以再运行时装配，无需在组件之间进行源代码链接。但是反射使用不当会成本很高！
Reflection 是Java被视为动态（或准动态）语言的一个关键性质。这个机制允许程序在运行时透过Reflection APIs取得任何一个已知名称的class的内部信息，包括其modifiers（诸如public, static 等等）、superclass（例如Object）、实现之interfaces（例如Serializable），也包括fields和methods的所有信息，并可于运行时改变fields内容或调用methods。
**为什么要用反射机制？**直接创建对象不就可以了吗，这就涉及到了动态与静态的概念， 
    静态编译：在编译时确定类型，绑定对象,即通过。 
    动态编译：运行时确定类型，绑定对象。动态编译最大限度发挥了java的灵活性，体现了多 
    态的应用，有以降低类之间的藕合性。 
    一句话，反射机制的优点就是可以实现动态创建对象和编译，体现出很大的灵活性，特别是在J2EE的开发中 
    它的灵活性就表现的十分明显。比如，一个大型的软件，不可能一次就把把它设计的很完美，当这个程序编 
    译后，发布了，当发现需要更新某些功能时，我们不可能要用户把以前的卸载，再重新安装新的版本，假如 
    这样的话，这个软件肯定是没有多少人用的。采用静态的话，需要把整个程序重新编译一次才可以实现功能 
    的更新，而采用反射机制的话，它就可以不用卸载，只需要在运行时才动态的创建和编译，就可以实现该功 
    能。 
       它的缺点是对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它 
    满足我们的要求。这类操作总是慢于只直接执行相同的操作。 

**反射机制的作用：**
  * 反编译：.class-->.java
  * 通过反射机制访问java对象的属性，方法，构造方法等；
**Java 反射机制主要提供了以下功能：**
  * 在运行时判断任意一个对象所属的类。
  * 在运行时构造任意一个类的对象。
  * 在运行时判断任意一个类所具有的成员变量和方法。
  * 在运行时调用任意一个对象的方法。


在这里先看一下sun为我们提供了那些反射机制中的类：
java.lang.Class;                
java.lang.reflect.Constructor; 
java.lang.reflect.Field;        
java.lang.reflect.Method;
java.lang.reflect.Modifier;
Array类：提供了动态创建数组，以及访问数组的元素的静态方法。

 很多反射中的方法，属性等操作我们可以从这四个类中查询。还是哪句话要学着不断的查询API，那才是我们最好的老师。

**利用反射机制能获得什么信息** 
         一句话，类中有什么信息，它就可以获得什么信息，不过前提是得知道类的名字，要不就没有后文了 
    首先得根据传入的类的全名来创建Class对象。 
    Class c=Class.forName("className");注明：className必须为全名，也就是得包含包名，比如，cn.netjava.pojo.UserInfo; 
    Object obj=c.newInstance();//创建对象的实例 
    OK，有了对象就什么都好办了，想要什么信息就有什么信息了。   
    获得构造函数的方法 
    Constructor getConstructor(Class[] params)//根据指定参数获得public构造器
    Constructor[] getConstructors()//获得public的所有构造器
    Constructor getDeclaredConstructor(Class[] params)//根据指定参数获得public和非public的构造器
    Constructor[] getDeclaredConstructors()//获得public的所有构造器 
    获得类方法的方法 
    Method getMethod(String name, Class[] params),根据方法名，参数类型获得方法
    Method[] getMethods()//获得所有的public方法
    Method getDeclaredMethod(String name, Class[] params)//根据方法名和参数类型，获得public和非public的方法
    Method[] getDeclaredMethods()//获得所以的public和非public方法 
    获得类中属性的方法 
    Field getField(String name)//根据变量名得到相应的public变量
    Field[] getFields()//获得类中所以public的方法
    Field getDeclaredField(String name)//根据方法名获得public和非public变量
    Field[] getDeclaredFields()//获得类中所有的public和非public方法 
    常用的就这些，知道这些，其他的都好办…… 
 
**介绍一下Modifiers**
我们都知道在field或者Constructor,Method前面都含有若干修饰符，如：
public static final String name="corey";
等等，我们应用getModifiers()能够拿到这个修饰符的一个整形值，然后应用Modifier这个类的静态方法来进行判断；如：Modifier.isStatic(int)等等；
获取其他方法。具体查看其API

**Class类**是Reflection API 中的核心类，它有以下方法
getName()：获得类的完整名字。
getFields()：获得类的` public `类型的属性。
getDeclaredFields()：获得类的**所有**属性。  访问**私有**属性时记得` field.setAccessible(true); `
getMethods()：获得类的public类型的方法。
getDeclaredMethods()：获得类的所有方法。
getMethod(String name, Class[] parameterTypes)：获得类的特定方法，name参数指定方法的名字，parameterTypes 参数指定方法的参数类型。
getConstructors()：获得类的public类型的构造方法。
getConstructor(Class[] parameterTypes)：获得类的特定构造方法，parameterTypes 参数指定构造方法的参数类型。
newInstance()：通过类的不带参数的构造方法创建这个类的一个对象。
##  关于getDeclaredMethods与getMethods的区别 
从以下结果可以看出，getDeclaredMethods与getMethods都能返回public和非public的方法。与可见性无关。但是getMethods还能返回从父类继承的方法。
所以我们一般用` getDeclaredMethods() `
```

	class MyRun implements Runnable {
		@Override
		public void run() {

		}
		public void haha() {

		}
		void a(){
			
		}
	}
		Method[] dms=MyRun.class.getDeclaredMethods();
		Method[] mts=MyRun.class.getMethods();
		System.out.println(Arrays.toString(dms));输出 [public void Test$MyRun.run(), void Test$MyRun.a(), public void Test$MyRun.haha()]
		System.out.println(Arrays.toString(mts));输出：                 
[public void Test$MyRun.run(), public void Test$MyRun.haha(), public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException, public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException, public final void java.lang.Object.wait() throws java.lang.InterruptedException, public boolean java.lang.Object.equals(java.lang.Object), public java.lang.String java.lang.Object.toString(), public native int java.lang.Object.hashCode(), public final native java.lang.Class java.lang.Object.getClass(), public final native void java.lang.Object.notify(), public final native void java.lang.Object.notifyAll()]

```
###  检查对象是否存在默认构造函数 
```

public static boolean hasDefaultConstructor(Class<?> clazz) throws SecurityException {
    Class<?>[] empty = {};
    try {
        clazz.getConstructor(empty);
    } catch (NoSuchMethodException e) {
        return false;
    }
    return true;
}

```

##  具体功能实现： 
###  反射机制获取类 

有三种方法，我们来获取Employee类
```

//第一种方式：  
Classc1 = Class.forName("Employee");  
//第二种方式：  
//java中每个类型都有class 属性.  
Classc2 = Employee.class;  
   
//第三种方式：  
//java语言中任何一个java对象都有getClass 方法  
Employee e = new Employee();  
Class c3 = e.getClass(); //c3是运行时类 (e的运行时类是Employee)  

```
###  反射机制创建对象： 

获取类以后我们来创建它的对象，利用newInstance：
```

Class c =Class.forName("Employee");  
  
//创建此Class 对象所表示的类的一个新实例  
Objecto = c.newInstance(); //调用了Employee的无参数构造方法.  

```
**` 所以一般我们需要提供无参构造器 `**
###  反射机制获取属性 
分为所有的属性和指定的属性：
** a，先看获取所有的属性的写法**：
```

//获取整个类  
            Class c = Class.forName("java.lang.Integer");  
              //获取所有的属性?  
            Field[] fs = c.getDeclaredFields();  
       
                   //定义可变长的字符串，用来存储属性  
            StringBuffer sb = new StringBuffer();  
            //通过追加的方法，将每个属性拼接到此字符串中  
            //最外边的public定义  
            sb.append(Modifier.toString(c.getModifiers()) + " class " + c.getSimpleName() +"{\n");  
            //里边的每一个属性  
            for(Field field:fs){  
                sb.append("\t");//空格  
                sb.append(Modifier.toString(field.getModifiers())+" ");//获得属性的修饰符，例如public，static等等  
                sb.append(field.getType().getSimpleName() + " ");//属性的类型的名字  
                sb.append(field.getName()+";\n");//属性的名字+回车  
            }  
      
            sb.append("}");  
      
            System.out.println(sb);  

```
###  获取特定的属性 

`  要注意的是：setAccessible()方法可以设置是否访问和修改私有属性 `
```

 //获取类  
    Class c = Class.forName("User");  
    //获取id属性  
    Field idF = c.getDeclaredField("id");  
    //实例化这个类赋给o  
    Object o = c.newInstance();  
    //打破封装  
    idF.setAccessible(true); //使用反射机制可以打破封装性，导致了java对象的属性不安全。  
    //给o对象的id属性赋值"110"  
    idF.set(o, "110"); //set  
    //get  
    System.out.println(idF.get(o));  
}  

```
###  获取方法并运行： 

```

public class ReflectionTest {
    public static void main(String[] args) throws Exception {
        DisPlay disPlay = new DisPlay();
        // 获得Class
        Class<?> cls = disPlay.getClass();
        // 通过Class获得DisPlay类的show方法
        Method method = cls.getMethod("show", String.class);
        // 调用show方法
        method.invoke(disPlay, "Wanggc");
    }
}

class DisPlay {
    public void show(String name) {
        System.out.println("Hello :" + name);
    }
}

```
```

class Person {
    public void print(int i) {
        System.out.println("我在写数字： " + i);
    }
     
    public static void say(String str) {
        System.out.println("我在说： " + str);
    }
}
 
public class Demo {
    public static void main(String[] args) throws Exception {
        Person p = new Person();
        Class<?> c = p.getClass();
     
        //getMethod()方法需要传入方法名，和参数类型
        Method m1 = c.getMethod("print", int.class);
        //invoke()表示调用的意思，需要传入对象和参数
        m1.invoke(p, 10);
         
        Method m2 = c.getMethod("say", String.class);
        //这里的null表示不由对象调用，也就是静态方法
        m2.invoke(null, "你妹");
    }
}

```
###  取得类所实现的接口和继承的父类 
```

public class Demo {
    public static void main(String[] args) throws Exception {
        Class<?> c = null;
        try {
            c = Class.forName("java.lang.Boolean");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        Class<?>[] in = c.getInterfaces();
        System.out.println(Arrays.toString(in));
      
        Class<?> su = c.getSuperclass();
        System.out.println(su);
    }
}

```
###  取得类的构造方法 
```

public class Demo {
    //下面的几个方法抛出来的异常太多，为了代码的紧凑性，这里就直接抛给虚拟机了
    public static void main(String[] args) throws Exception {
        Class<?> c = null;
        try {
            c = Class.forName("java.lang.Boolean");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        //这里的getConstructors()方法返回的是一个Constructor数组
        Constructor<?>[] cons = c.getConstructors();
        //打印的方式你可以自己写，为了方便我用Arrays.toString()，凑合着看
        System.out.println(Arrays.toString(cons));
    }
}

```
###  反射获取枚举常量 

```

public static Object getEnumConstant(Class<?> clazz, String name) {
		if (clazz==null || name==null || name.isEmpty()) {
			return null;
		}
		return Enum.valueOf((Class<Enum>)clazz, name);
	}

```
###  反射获取返回值类型 
```

public static Class<?> getMethodReturnType(Class<?> clazz, String name) {
		if (clazz==null || name==null || name.isEmpty()) {
			return null;
		}
		
		name = name.toLowerCase();
		Class<?> returnType = null;
		
		for (Method method : clazz.getDeclaredMethods()) {
			if (method.getName().equals(name)) {
				returnType = method.getReturnType();
				break;
			}
		}
		
		return returnType;
	}

```
##  反射获取父类的方法和属性 
getDeclaredMethod只能获取本类中的方法。
父类的方法是**不能**getDeclaredMethod(String name, Class<?>... parameterTypes)获取到的。
**子类是不可以直接反射父类的方法的，需要一层层的查找父类方法。**
可以参考：http://931360439-qq-com.iteye.com/blog/938886
```

    /** 
     * 循环向上转型, 获取对象的 DeclaredMethod 
     * @param object : 子类对象 
     * @param methodName : 父类中的方法名 
     * @param parameterTypes : 父类中的方法参数类型 
     * @return 父类中的方法对象 
     */  
      
    public static Method getDeclaredMethod(Object object, String methodName, Class<?> ... parameterTypes){  
        Method method = null ;  
          
        for(Class<?> clazz = object.getClass() ; clazz != Object.class ; clazz = clazz.getSuperclass()) {  
            try {  
                method = clazz.getDeclaredMethod(methodName, parameterTypes) ; 
                return method ;  
            } catch (Exception e) {  
                //这里甚么都不要做！并且这里的异常必须这样写，不能抛出去。  
                //如果这里的异常打印或者往外抛，则就不会执行clazz = clazz.getSuperclass(),最后就不会进入到父类中了  
              
            }  
        }  
          
        return null;  
    }  
    /** 
     * 循环向上转型, 获取对象的 DeclaredField 
     * @param object : 子类对象 
     * @param fieldName : 父类中的属性名 
     * @return 父类中的属性对象 
     */  
      
    public static Field getDeclaredField(Object object, String fieldName){  
        Field field = null ;  
          
        Class<?> clazz = object.getClass() ;  
          
        for(; clazz != Object.class ; clazz = clazz.getSuperclass()) {  
            try {  
                field = clazz.getDeclaredField(fieldName) ;  
                return field ;  
            } catch (Exception e) {  
                //这里甚么都不要做！并且这里的异常必须这样写，不能抛出去。  
                //如果这里的异常打印或者往外抛，则就不会执行clazz = clazz.getSuperclass(),最后就不会进入到父类中了  
                  
            }   
        }  
      
        return null;  
    } 

```
##  用反射机制实现对数据库数据的增、查例子 
纤细参考：http://www.cnblogs.com/jqyp/archive/2012/03/29/2423112.html
基本原理；保存数据时，把需要保存的对象的属性值全部取出来再拼凑sql语句 ， 查询时，将查询到的数据全部包装成一个java对象。 
游戏规则：俗话说的好，无规矩不成方圆，特别是程序来说，它只能做有规则的事情，没有规则的它干不了，好，那就 
先定规则 
1）数据库的每一个表对象一个pojo类，表中的每一个字段对应pojo类的中的一个属性。 
并且pojo类的名字和表的名字相同，**属性名和字段名相同，大小写没有关系，因为数据库一般不区分大小写 ** 
2）为pojo类中的每一个属性添加标准的set和get方法。 
有了游戏规则，那么开始游戏吧。
```

public class NetJavaSession {
    /**
     * 解析出保存对象的sql语句
     *
     * @param object
     *            ：需要保存的对象
     * @return：保存对象的sql语句
     */
    public static String getSaveObjectSql(Object object) {
        // 定义一个sql字符串
        String sql = "insert into ";
        // 得到对象的类
        Class c = object.getClass();
        // 得到对象中所有的方法
        Method[] methods = c.getMethods();
        // 得到对象中所有的属性
        Field[] fields = c.getFields();
        // 得到对象类的名字
        String cName = c.getName();
        // 从类的名字中解析出表名
        String tableName = cName.substring(cName.lastIndexOf(".") + 1,
                cName.length());
        sql += tableName + "(";
        List<String> mList = new ArrayList<String>();
        List vList = new ArrayList();
        for (Method method : methods) {
            String mName = method.getName();
            if (mName.startsWith("get") && !mName.startsWith("getClass")) {
                String fieldName = mName.substring(3, mName.length());
                mList.add(fieldName);
                System.out.println("字段名字----->" + fieldName);
                try {
                    Object value = method.invoke(object, null);
                    System.out.println("执行方法返回的值：" + value);
                    if (value instanceof String) {
                        vList.add("\"" + value + "\"");
                        System.out.println("字段值------>" + value);
                    } else {
                        vList.add(value);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        for (int i = 0; i < mList.size(); i++) {
            if (i < mList.size() - 1) {
                sql += mList.get(i) + ",";
            } else {
                sql += mList.get(i) + ") values(";
            }
        }
        for (int i = 0; i < vList.size(); i++) {
            if (i < vList.size() - 1) {
                sql += vList.get(i) + ",";
            } else {
                sql += vList.get(i) + ")";
            }
        }
 
        return sql;
    }
 
    public static List getDatasFromDB(String tableName, int Id) {
 
        return null;
 
    }
 
    /**
     * 将对象保存到数据库中
     *
     * @param object
     *            ：需要保存的对象
     * @return：方法执行的结果;1:表示成功，0:表示失败
     */
    public int saveObject(Object object) {
        Connection con = Connect2DBFactory.getDBConnection();
        String sql = getSaveObjectSql(object);
        try {
            // Statement statement=(Statement) con.createStatement();
            PreparedStatement psmt = con.prepareStatement(sql);
            psmt.executeUpdate();
            return 1;
        } catch (SQLException e) {
            e.printStackTrace();
            return 0;
        }
    }
 
    /**
     * 从数据库中取得对象
     *
     * @param arg0
     *            ：对象所属的类
     * @param id
     *            ：对象的id
     * @return:需要查找的对象
     */
    public Object getObject(String className, int Id) {
        // 得到表名字
        String tableName = className.substring(className.lastIndexOf(".") + 1,
                className.length());
        // 根据类名来创建Class对象
        Class c = null;
        try {
            c = Class.forName(className);
 
        } catch (ClassNotFoundException e1) {
 
            e1.printStackTrace();
        }
        // 拼凑查询sql语句
        String sql = "select * from " + tableName + " where Id=" + Id;
        System.out.println("查找sql语句：" + sql);
        // 获得数据库链接
        Connection con = Connect2DBFactory.getDBConnection();
        // 创建类的实例
        Object obj = null;
        try {
 
            Statement stm = con.createStatement();
            // 得到执行查寻语句返回的结果集
            ResultSet set = stm.executeQuery(sql);
            // 得到对象的方法数组
            Method[] methods = c.getMethods();
            // 遍历结果集
            while (set.next()) {
                obj = c.newInstance();
                // 遍历对象的方法
                for (Method method : methods) {
                    String methodName = method.getName();
                    // 如果对象的方法以set开头
                    if (methodName.startsWith("set")) {
                        // 根据方法名字得到数据表格中字段的名字
                        String columnName = methodName.substring(3,
                                methodName.length());
                        // 得到方法的参数类型
                        Class[] parmts = method.getParameterTypes();
                        if (parmts[0] == String.class) {
                            // 如果参数为String类型，则从结果集中按照列名取得对应的值，并且执行改set方法
                            method.invoke(obj, set.getString(columnName));
                        }
                        if (parmts[0] == int.class) {
                            method.invoke(obj, set.getInt(columnName));
                        }
                    }
 
                }
            }
 
        } catch (Exception e) {
            e.printStackTrace();
        }
        return obj;
    }
}

```
###  四、动态创建和访问数组 
` java.lang.Array ` 类提供了动态创建和访问数组元素的各种静态方法。
例程ArrayTester1 类的main()方法创建了一个长度为10 的字符串数组，接着把索引位置为5 的元素设为“hello”，然后再读取索引位置为5 的元素的值
```

public class ArrayTester1 {
    public static void main(String args[]) throws Exception {
        Class<?> classType = Class.forName("java.lang.String");
        // 创建一个长度为10的字符串数组
        Object array = Array.newInstance(classType, 10);
        // 把索引位置为5的元素设为"hello"
        Array.set(array, 5, "hello");
        // 获得索引位置为5的元素的值
        String s = (String) Array.get(array, 5);
        System.out.println(s);
    }
}

public class Demo {
    public static void main(String[] args) throws Exception {
        int[] arr = {1,2,3,4,5};
        Class<?> c = arr.getClass().getComponentType();
         
        System.out.println("数组类型： " + c.getName());
        int len = Array.getLength(arr);
        System.out.println("数组长度： " + len);
        System.out.print("遍历数组： ");
        for (int i = 0; i < len; i++) {
            System.out.print(Array.get(arr, i) + " ");
        }
        System.out.println();
        //修改数组
        System.out.println("修改前的第一个元素： " + Array.get(arr, 0));
        Array.set(arr, 0, 3);
        System.out.println("修改后的第一个元素： " + Array.get(arr, 0));
    }
}

```
##  JDK动态代理： 
与静态代理类对照的是动态代理类，动态代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手工编写它的源代码。动态代理类不仅简化了编程工作，而且提高了软件系统的可扩展性，因为Java 反射机制可以生成任意类型的动态代理类。java.lang.reflect 包中的` Proxy类和InvocationHandler ` 接口提供了生成动态代理类的能力。
Java动态代理类位于java.lang.reflect包下，一般主要涉及到以下两个类： 
(1)Interface InvocationHandler：该接口中仅定义了一个方法 public object invoke(Object obj,Method method, Object[] args) 在实际使用时，第一个参数obj一般是指代理类，method是被代理的方法，这个抽象方法在代理类中动态实现。 
(2)Proxy：该类即为动态代理类

**1. 动态代理的步骤**
(1).创建一个实现接口InvocationHandler的类，它必须实现invoke方法
(2).创建被代理的类以及接口
(3).通过Proxy的静态方法` newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h) ` 创建一个代理
(4).通过代理调用方法

```

public interface Subject   
{   
  public void doSomething();   
}   
public class RealSubject implements Subject   
{   
  public void doSomething()   
  {   
    System.out.println( "call doSomething()" );   
  }   
}   
public class ProxyHandler implements InvocationHandler   
{   
  private Object proxied;   
     
  public ProxyHandler( Object proxied )   
  {   
    this.proxied = proxied;   
  }   
     
  public Object invoke( Object proxy, Method method, Object[] args ) throws Throwable   
  {   
    //在转调具体目标对象之前，可以执行一些功能处理

    //转调具体目标对象的方法
    return method.invoke( proxied, args);  
    
    //在转调具体目标对象之后，可以执行一些功能处理
  }    
}

```
```

public class DynamicProxy   
{   
  public static void main( String args[] )   
  {   
    RealSubject real = new RealSubject();   
    Subject proxySubject = (Subject)Proxy.newProxyInstance(Subject.class.getClassLoader(), 
     new Class[]{Subject.class}, 
     new ProxyHandler(real));
         
    proxySubject.doSomething();
  }      
}

```
##  反射与泛型 
　java 的泛型，只是编译时作为类型检查，一旦编译完成，泛型就会被擦除，在运行期间是得不到泛型的信息的，包括它的类型参数。有时候我们需要用到泛型的类型参数，
```

class Test {  
    public static void main(String args[]) throws Exception {  
        Method applyMethod=Test.class.getMethod("applyVector", Vector.class);  
        Type[] types=applyMethod.getGenericParameterTypes();  
        System.out.println(types[0].toString());  
    }  
    public static void applyVector(Vector<Date> v1){  
          
    }  
}  

```
输出：java.util.Vector<java.util.Date>
```

class Test {  
    public static void main(String args[]) throws Exception {  
        Method applyMethod=Test.class.getMethod("applyVector", Vector.class);  
        Type[] types=applyMethod.getGenericParameterTypes();  
        ParameterizedType pType=(ParameterizedType)types[0];  
        System.out.println(pType.getActualTypeArguments()[0]);  
        System.out.println(pType.getRawType());  
  
    }  
    public static void applyVector(Vector<Date> v1){  
          
    }  
}  

```
输出：
class java.util.Date
class java.util.Vector
可见，getActualTypeArguments 能取得实际类型参数，getRawType() 取得原本的类型。
```

Field field = Pair.class.getDeclaredField("myList"); //myList的类型是List 
Type type = field.getGenericType(); 
if (type instanceof ParameterizedType) {     
    ParameterizedType paramType = (ParameterizedType) type;     
    Type[] actualTypes = paramType.getActualTypeArguments();   

```
  。。。
```

public abstract class Foo<T> {
    private Class<T> tClass;    

    public Foo(Class<T> tClass) {
        this.tClass = tClass;
    }
    //content
}

public class FooChild extends Foo<Bar> {
    public FooChild() {
        super(FooChild.class);
    }
    //content
} 

```
```

public static Type[] getParameterizedTypes(Object object) {
    Type superclassType = object.getClass().getGenericSuperclass();
    if (!ParameterizedType.class.isAssignableFrom(superclassType.getClass())) {
        return null;
    }
    return ((ParameterizedType)superclassType).getActualTypeArguments();
}

```
##  反射与注解 
获取注解信息
**在框架开发中，注解加反射的组合使用是最为常见形式的。**定义注解时我们会通过` @Target ` 指定该注解能够作用的类型，看如下示例:

```

    @Target({
            ElementType.METHOD, ElementType.FIELD, ElementType.TYPE
    })
    @Retention(RetentionPolicy.RUNTIME)
    static @interface Test {
	
    }

```
上述注解的@target 表示该注解只能用在函数上，还有 Type、Field、PARAMETER 等类型，可以参考上述给出的参考资料。通过反射 api 我们也能够获取一个 Class 对象获取类型、属性、函数等相关的对象，通过这些对象的 ` getAnnotation ` 接口获取到对应的注解信息。 首先我们需要在目标对象上添加上注解，例如 :
```

@Test(tag = "Student class Test Annoatation")
public class Student extends Person implements Examination {
    // 年级
    @Test(tag = "mGrade Test Annotation ")
    int mGrade;

    // ......
}

```
然后通过相关的注解函数得到注解信息，如下所示 :
```

    private static void getAnnotationInfos() {
        Student student = new Student("mr.simple");
        Test classTest = student.getClass().getAnnotation(Test.class);
        System.out.println("class Annotatation tag = " + classTest.tag());

        Field field = null;
        try {
            field = student.getClass().getDeclaredField("mGrade");
            Test testAnnotation = field.getAnnotation(Test.class);
            System.out.println("属性的 Test 注解 tag : " + testAnnotation.tag());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

```
输出结果为 : >
class Annotatation tag = Student class Test Annoatation
属性的 Test 注解 tag : mGrade Test Annotation

**接口说明**
```

// 获取指定类型的注解
public <A extends Annotation> A getAnnotation(Class<A> annotationClass) ;
// 获取 Class 对象中的所有注解
public Annotation[] getAnnotations() ;

```
反射作为 Java 语言的重要特性，在开发中有着极为重要的作用。**很多开发框架都是基于反射来实现对目标对象的操作，而反射配合注解更是设计开发框架的主流选择，**因此深入了解反射的作用以及使用对于日后开发和学习必定大有益处。
参考资料：
http://codekk.com/open-source-project-analysis/detail/Android/Mr.Simple/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8F%8D%E5%B0%84%20Reflection
http://qussay.com/wp-content/uploads/2013/09/ReflectionUtil.java
http://blog.csdn.net/liujiahan629629/article/details/18013523
http://www.cnblogs.com/gulvzhe/archive/2012/01/27/2330001.html
http://www.cnblogs.com/jqyp/archive/2012/03/29/2423112.html
http://lavasoft.blog.51cto.com/62575/43218/
http://my.oschina.net/zookeeper/blog/179269
http://www.cnblogs.com/nerxious/archive/2012/12/24/2829446.html
http://www.infoq.com/cn/articles/cf-java-reflection-dynamic-proxy/
http://www.cnblogs.com/flyoung2008/archive/2013/08/11/3251148.html
http://www.cnblogs.com/whitewolf/p/4355541.html
