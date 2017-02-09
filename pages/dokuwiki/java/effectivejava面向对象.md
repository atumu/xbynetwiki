title: effectivejava面向对象 

#  effective Java笔记 
主动清除过期的对象引用。
使类和成员的可访问性最小化，将局部变量的作用域最小化
优先使用for-each循环
使可变性最小化
复合优先于继承
接口优先于抽象类，变量声明，方法等面向接口编程
接口优先于反射
用命令对象表示策略
优先考虑泛型 
注解优先于命名模式或命名约定
检查参数的有效性，并在必要时抛出非检查异常
谨慎设计方法签名：
  * 谨慎地选择方法名称
  * 避免过长的参数列表，可使用辅助类或Builder模式替代
  * 对于参数类型，要优先使用接口而不是类

慎用重载
慎用可变参数
谨慎地进行优化
返回零长度的数组或集合，而不是null
为所有导出的API元素编写文档注释
如需精确的答案，请避免使用float和double
当心字符串连接的性能。用StringBuilder代替String连接
对可恢复的情况使用受检异常，对编程错误使用运行时异常RuntimeException的子类
executor优先于线程
并发工具优先于wait和notify
避免过度使用同步
线程安全的文档化

##  慎用延迟初始化： 
```

// Initialization styles - Pages 282-284
public class Initialization {
	正常初始化
    // Normal initialization of an instance field - Page 282
    private final FieldType field1 = computeFieldValue();
	采用同步方法的延迟初始化
    // Lazy initialization of instance field - synchronized accessor - Page 282
    private FieldType field2;
    synchronized FieldType getField2() {
        if (field2 == null)
            field2 = computeFieldValue();
        return field2;
    }
	采用Holder模式的延迟初始化
    // Lazy initialization holder class idiom for static fields - Page 283
    private static class FieldHolder {
        static final FieldType field = computeFieldValue();
    }
    static FieldType getField3() { return FieldHolder.field; }

	双检查的延迟初始化，记得变量标记为volatile
    // Double-check idiom for lazy initialization of instance fields - Page 283
    private volatile FieldType field4;
    FieldType getField4() {
        FieldType result = field4;
        if (result == null) {  // First check (no locking)
            synchronized(this) {
                result = field4;
                if (result == null)  // Second check (with locking)
                    field4 = result = computeFieldValue();
            }
        }
        return result;
    }

    // Single-check idiom - can cause repeated initialization! - Page 284
    private volatile FieldType field5;
    private FieldType getField5() {
        FieldType result = field5;
        if (result == null) 
            field5 = result = computeFieldValue();
        return result;
    }

    private static FieldType computeFieldValue() {
        return new FieldType();
    }
}

class FieldType { }


```
##  第1条，在某种情况下如工具类可以考虑使用静态工厂方法代替构造器。 

优势：
1、有名称；
2、不必没次调用他们的时候都创建一个新对象（有利于预先构建或缓存实例）如Boolean.valueOf()；
3、可以利用面向接口与多态特性，返回原返回类型的任何子类型。如DriverManager.registerDriver();
4、在创建参数化类型实例的时候，使代码变得更加简洁。如public static <K,V> HashMap<K,V> new Instance(){ return new HashMap<K,V>()}

##  第2条：遇到多个构造器参数时要考虑用构建器（也就是XxxBuilder模式） 

多个构造器参数，且大多数参数都是选的。
通常采用的方式为：多个重载的构造器方法。有一个包含必要参数，其他几个构造器包含不同的可选参数，最后一个包含所有参数。
```

public class NutritionFacts {
    private final int servingSize;   // (mL)            required
    private final int servings;      // (per container) required
    private final int calories;      //                 optional
    private final int fat;           // (g)             optional
    private final int sodium;        // (mg)            optional
    private final int carbohydrate;  // (g)             optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings,
            int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
            int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
            int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings,
           int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola =
            new NutritionFacts(240, 8, 100, 0, 35, 27);
    }
}

```
但是这样带来的是客户端代码会很难编写，或者难以被理解。
或许你会认为可以采用JavaBean的形式来代替。但是这造成另一个结果，那就是将构造过程被分到了多个setter方法中，很可能造成不一致的状态。同时阻止了该类成为不可变类。

我们可以换种思路。` 采用Builder构建器模式。 `
```

public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int carbohydrate  = 0;
        private int sodium        = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
            { calories = val;      return this; }
        public Builder fat(int val)
            { fat = val;           return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val;  return this; }
        public Builder sodium(int val)
            { sodium = val;        return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).
            calories(100).sodium(35).carbohydrate(27).build();
    }
}

```

##  第4条 覆盖equals时请遵守约定 

 1.对象内容的比较，需要使用equals()方法，若是对于已经重写该方法的类，例如String类，就无需再重写；若是没有重写，例如自定义的User类，就需要重写。
 2.Java语言中的“==”对于基本数据类型就是比较其值，而对于对象就是比较对象的引用。
 3.**重写equals方法后最好重写hashCode方法**，否则两个等价对象可能得到不同的hashCode,这在集合框架中使用可能产生严重后果
1 在改写equals时要遵守通用约定
1.1 不必改写equals的情况：
  * 1）一个类的每个实例本质上都是惟一的。
  * 2）不关心一个类是否提供了“逻辑相等”的测试功能。
  * 3）超类已经改写的equals，从超类继承的行为对于子类也是合适的。
  * 4）一个类是私有的，或者是包级私有的，并且可以确定它的equals方法永远也不会被调用。
1.2 需要改写Object.equals的情况：
当一个类有自己特有的“逻辑相等”概念，而且父类也没有改写equals以实现期望的行为。通常适合于value class的情形。
改写equals也使得这个类的实例可以被用作map的key或者set的元素，并使map和set表现出预期的行为。
1.3 改写equals要遵守的通用约定（equals方法实现了等价关系）：
  * 1）自反性：x.equals(x)一定返回true
  * 2）对称性：x.equals(y)返回true当且仅当y.equals(x)
  * 3）传递性：x.equals(y)且y.equals(z)，则x.equals(z)为true
  * 4）一致性：若x.equals(y)返回true，则不改变x，y时多次调用x.equals(y)都返回true
  * 5）对于任意的非空引用值x，x.equals(null)一定返回false。
当重写完equals方法后，应该检查是否满足对称性、传递性、一致性。（自反性、null通常会自行满足）
```

public class User {
	private String name;
	private String pass;
	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + ((name == null) ? 0 : name.hashCode());
		result = prime * result + ((pass == null) ? 0 : pass.hashCode());
		return result;
	}
	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		User other = (User) obj;
		if (name == null) {
			if (other.name != null)
				return false;
		} else if (!name.equals(other.name))
			return false;
		if (pass == null) {
			if (other.pass != null)
				return false;
		} else if (!pass.equals(other.pass))
			return false;
		return true;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getPass() {
		return pass;
	}
	public void setPass(String pass) {
		this.pass = pass;
	}
}


```  

##  第5条 覆盖equals时总要覆盖hashCode 
```

	@Override
	public int hashCode() {
		final int prime = 31;
		int result = 1;
		result = prime * result + ((name == null) ? 0 : name.hashCode());
		result = prime * result + ((pass == null) ? 0 : pass.hashCode());
		return result;
	}

```
##  第6条 始终覆盖toString 
```

class Item implements Comparable<Item>  
{  
    public Item(String aDescription, int aPartNumber)  
    {  
        description=aDescription;  
        partNumber=aPartNumber;  
    }  
    public String getDescription()  
    {  
        return description;  
    }  
    public String toString()  
    {  
        return "[description="+description  
                +",partNumber="+partNumber+"]";  
    }  

```

##  用enum代替int常量 

```

public class Test {
	public static void main(String[] args) {
		System.out.println(TestEnum.JAVA==TestEnum.JAVA);
		System.out.println(TestEnum.JAVA.equals(TestEnum.JAVA));
		System.out.println(TestEnum.JAVA.name());
		System.out.println(TestEnum.JAVA.toString());
	}
}
true
true
JAVA
JAVA

```
