title: chapter999_关键技巧 

#  Chapter999 关键技巧 
Created Wednesday 11 March 2015

#  浮点数的比较 

* 使用Double.compare()用于比较浮点数。记住不能直接用==比较浮点数，

#  equals写法： 
```

 private String name;
   private double salary;
   private Date hireDay;
   /**
	* 这是equals比较全面的稳定的写法。先判断是否==如果否，,然后再判断是否为null为后续判断奠定基础，如果否，则需要判断是否属于同一个类型，
		如果为是，此时我们便可以进行转换。
	* 使用Objects.equals()可以防止出现null，
	* 使用Double.compare()用于比较浮点数。记住不能直接用==比较浮点数，
	* @param obj 
	* @return boolean
		*/
	   @Override
	   public boolean equals(Object obj){
		   if(this==obj) return true;
		   if(obj==null) return false;
		   if(getClass()!=obj.getClass()) return false;
		   Employee other=(Employee)obj;
		   
		   return Objects.equals(name, other.name)
				   &&(Double.compare(salary, other.salary)==0?true:false)
				   &&Objects.equals(hireDay, other.hireDay);
	   }

```
注意：对于数组类型的域，可以使用**Arrays.equals()**方法进行判断相等性。

#  hashCode写法 
```

 public int hashCode()
   {
	  return Objects.hash(name, salary, hireDay); 
   }


```