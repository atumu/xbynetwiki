title: java继承时构造器调用问题 

#  java继承时构造器调用问题 
最近出现这个错误:Implicit super constructor MethodVisitor() is undefined. Must explicitly invoke another constructor
解决问题的关键就是当显示调用父类的构造函数时，` 必须在构造器的第一行调用super(xxx) `。
```

class Person { 
	protected String name; 
	protected int age; 
	//你已经定义了自动的构造函数，此时编译器不会为你创建默认的构造函数 
	public Person(String name,int age) { 
		this.name=name; 
		this.age=age; 
	} 
	public void print() { 
		System.out.println("Name:"+name+"/nAge:"+age); 
	}
}

``` 
```

/*由于父类的构造函数是有参的，所以编译不会为你自动调用默认的构造函数，此时，子类在自己的构造函数中必须显式的调用父类的构造函数 */
class Student extends Person { 
	public Student(){      //子类构造函数 
	//super();   不行，因为你的父类没有无参的构造函数 
	super("a",1); //显示调用父类的构造函数，而且必须是第一行调用，这就是问题的关键。
	} 
}

``` 