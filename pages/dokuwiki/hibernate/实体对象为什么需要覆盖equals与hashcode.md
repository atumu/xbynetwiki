title: 实体对象为什么需要覆盖equals与hashcode 

#  实体对象为什么需要覆盖equals与hashCode 
让我设想以下情景：
```

Session s1=sessionFactory.openSession();
s1.beginTransaction();
Object a=s1.get(Item.class,new Long(123));
Object a=s1.get(Item.class,new Long(123));

// a==b为true，a、b指向相同的内存地址。

s1.getTransaction.commit();
s1.close();

Session s2=sessionFactory.openSession();
s2.beginTransaction();
Object c=s2.get(Item.class,new Long(123));

//a==c返回false，a,c指向不同的内存地址

s2.getTransaction.commit();
s2.close();

/**现在a,b,c都处于脱管状态*/
Set set=new HashSet();
set.add(a);
set.add(b);
set.add(c);

/**输出什么呢？1?,2?,3?*/
System.out.println(set.size())

```
我们知道set是通过调用集合元素的equals方法来确保元素唯一性的。而Object的equals方法默认就是通过obj1==obj2来通过内存地址判断同一性。
那我们现在知道了把。**上述会输出2。但是我们知道这两个元素都是代表数据裤中的同一行。所以这个时候我们就需要覆盖equals与hashCode方法了。建议我们对所有的实体类都覆盖这两个方法，**因为我们无法保证这种情况不会发生。

覆盖实体类的equals方法可以选择两种方式：
  * 1、通过判断业务键而非数据库主键的相等性。如果业务键是唯一的话，推荐此方式。
  * 2、通过判断对象的所有属性的相等性。

```

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

```