title: equals与hashcode方法的正确重写方式 

#  equals与hashCode方法的正确重写方式 
参考：http://www.oschina.net/question/82993_75533
目录：
  * hashCode()和equals()的重写
  * 使用Eclipse自动生成equals()和hashCode()
  * 使用Apache Commons Lang包重写hashCode()和equals()
  * 当使用ORM的时候特别要注意的
##  hashCode()和equals()的重写 
可以看到，我们对每个参与equals与hashCode的` **属性都进行了非null判断** `。我们往往忽略了这一点，但这是很关键的，**你总不希望在添加实列进集合时调用equals或hashCode抛出NullPointException把。**
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
##  使用Eclipse自动生成equals()和hashCode() 
![](/data/dokuwiki/java/pasted/20150912-152206.png)
##  使用Commons Lang包重写hashCode()和equals()方法 
Apache Commons 包提供了两个非常优秀的类来生成hashCode()和equals()方法。看下面的程序。 
```

public class Employee
{
 private Integer id;
 private String firstname;
 private String lastName;
 private String department;
public Integer getId() {
    return id;
 }
 public void setId(Integer id) {
    this.id = id;
 }
 public String getFirstname() {
    return firstname;
 }
 public void setFirstname(String firstname) {
    this.firstname = firstname;
 }
 public String getLastName() {
    return lastName;
 }
 public void setLastName(String lastName) {
    this.lastName = lastName;
 }
 public String getDepartment() {
    return department;
 }
 public void setDepartment(String department) {
    this.department = department;
 }
@Override
 public int hashCode()
 {
    final int PRIME = 31;
    return new HashCodeBuilder(getId()%2==0?getId()+1:getId(), PRIME).
           toHashCode();
 }
@Override
 public boolean equals(Object o) {
    if (o == null)
       return false;
    if (o == this)
       return true;
    if (o.getClass() != getClass())
       return false;
    Employee e = (Employee) o;
       return new EqualsBuilder().
              append(getId(), e.getId()).
              isEquals();
    }
 }

```
##  当使用ORM的时候特别要注意的 
如果你使用ORM处理一些对象的话，你要确保在hashCode()和equals()对象中使用getter和setter而不是直接引用成员变量。
因为在ORM中有的时候成员变量会被**延时加载**，**这些变量只有当getter方法被调用的时候才真正可用。**
例如在我们的例子中，如果我们使用e1.id == e2.id则可能会出现这个问题，但是我们使用e1.getId() == e2.getId()就不会出现这个问题。