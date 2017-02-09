title: java进阶202 

#  Java Bridge Method (桥接方法) 
```

abstract class A<T> {  
    abstract T get(T t);  
}  
  
class B extends A<String> {  
    @Override  
    String get(String s) {  
        return "";  
    }  
}  
  
public class TestBridge {  
    public static void main(String[] args) {  
        Class<B> clazz = B.class;  
        Method[] methods = clazz.getDeclaredMethods();  
        for (int i = 0; i < methods.length; i++) {  
            Method m = methods[i];  
            System.out.println(getMethodInfo(m) + " is Bridge Method? " + m.isBridge());  //用Method.isBridge()来进行判断
        }  
    }  
      
    public static String getMethodInfo(Method m){  
        StringBuilder sb = new StringBuilder();  
        sb.append(m.getReturnType()).append(" ");  
        sb.append(m.getName());  
        Class[]params = m.getParameterTypes();  
        for (int i = 0; i < params.length; i++) {  
            sb.append(params[i].getName()).append(" ");  
        }  
        return sb.toString();  
    }  
}  

```
运行上面的代码，可以看到输出结果如下： 
[class java.lang.String get] is Bridge Method? false 
[class java.lang.Object get] is Bridge Method? true 

**或许你现在诧异了，怎么B类中只定义了一个方法，现在怎么出现了两个呢？而且返回类型和参数类型还不同。话说这个是java5中的泛型所带来的结果了。**针对上面的这段代码分析下： 
在java5之前，你可以往一个集合里扔任何你想扔的对象。往集合中放对象的人很爽了。但是从集合中去对象的人就头大了。你不知道你下个取到的对象将会是什么类型的。不知道转成什么类型，你也就只能使用所有Object的方法了，这样就毫无意义了。所以在java5中提供了泛型这一新特性。程序员在写代码的时候可以指定集合可以存放对象的类型。然后将这些类型检查的事情交给编译器去做，减少了程序员的工作。 
上面代码中<>中的T和String就是指定类的参数类型。T代表一种泛型，告诉编译器，一旦有类指定了T这个参数的实际类，那么get方法返回的类型也必须为同一个类（当然也可以是这个类的子类；这个也是java5中的协变式返回新特性），如果不是，就必须报错提示；将原来的运行时可能出现的错误提前到编译期了。**那么，假设你是java5编译器的设计者，你会如何来设计让编译器能实现这个特性，同时能保证编译出来的字节码可以在老版本的jdk中运行呢？java5编译器中作了个很巧妙的设计——桥接方法。** 
下面来说说java5编译器是如何来编译上面的代码： 
```

abstract class A<T> {  
    abstract T get(T t);  
} 

``` 
**对于A类，编译器看到<>中指定的T参数后，会用Object把类中的其他T参数替换。因为在jdk中根本就不存在T这个类嘛。**替换后就成了下面这样子。 
```

abstract class A {  
    abstract Object get(Object obj);  
}

```  
上面这个过程称为` 类型擦除 `。对于B类，它继承了A类，指定了T参数为String。java5编译器在编译的时候做了些手脚。**当编译器发现你指定了类型参数，便会在编译的字节码中添加一个桥接方法。**这个可以查看B的反编译代码就知道了。 
```

class B extends A {  
    //编译器添加的方法  
    Object get(Object s) {  
        return (Object) get((String) s);  
    }  
  
    String get(String s) {  
        return "";  
    }  
} 

``` 
下面再说说协变式返回，什么是协变式返回呢？先可以**对比下java1.4和java5中对于重写(Override)的定义**： 
In Java 1.4, and earlier, one method can override another if the signatures match exactly. 
In Java 5, a method can override another if the arguments match exactly but the return type of the overriding method, if it is a subtype of the return type of the other method. 
也就是说，**对于重写的判断放宽了条件，子类中方法返回的类型是父类中方法返回类型的子类也是重写**，听起来有点绕，看下面这段代码： 
```

class A{  
    Father get(){}  
}  
class B extends A{  
    @Override  
    Son get(){}  
}  
class Father{}  
class Sun extends Father{} 

``` 
也就是B中的get()方法返回Son类也是重写。如何实现的呢？同样是使用桥接方法。 

参考:http://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html
http://berdy.iteye.com/blog/810488
http://stackoverflow.com/questions/289731/what-java-lang-reflect-method-isbridge-used-for
http://aruld.info/synthetic-methods-in-java/