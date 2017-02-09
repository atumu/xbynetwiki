title: css_db 

#  css_db中软持久层层中间件 
基于Hibernate3.6与集成了memcached分布式内存缓存，提供**分页**功能**，屏蔽了开发者对内存缓存的直接操作**，并针对常用的几类模型进行抽象：**字典模型，树模型，关联模型**。
##  模型写法: 
情景：用户组与用户。
Group.java：
```

implements Serializable //实现序列化接口
	/** default constructor 提供一个默认构造器*/
	public Group() {
	}
	/**
	 * 定义关联用户列表
	 */
	private transient JoinList users = null;
	/**
	 * 查询当前组下的用户列表
	 */
	public JoinList getUsers() {
		if (users == null) {
			QueryCache qc = new QueryCache("select a.id from User a where a.groupId=:groupId") //hql语句，:groupId占位符，
					.setParameter("groupId", id);
			users = new JoinList(User.class, qc);
		}
		return users;
	}
	
//这是一个典型的树结构，通过parentId指向父节点的id
public Integer getParentId() {
		return this.parentId;
	}

	public void setParentId(Integer parentId) {
		this.parentId = parentId;
	}

```
User.java:
```

正常JavaBean
implements Serializable //实现序列化接口
public User(String name, Integer groupId, Integer sexId) {
        this.name = name;
        this.groupId = groupId;
        this.sexId = sexId;
    }
    public User() {
    }
//多了个指向group的id
public Integer getGroupId() {
        return this.groupId;
    }
    public void setGroupId(Integer groupId) {
        this.groupId = groupId;
    }

```
##  O/R Mapping 
Group.hbm.xml:
```

<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd" >
<hibernate-mapping>
<class 
    name="com.css.demo.model.Group" 
    table="user_group"
>
    <id
        name="id"
        type="java.lang.Integer"
        column="id"
    >
        <generator class="native" />
    </id>
    <property
        name="name"
        type="java.lang.String"
        column="name"
        length="50"
    />
    <property
        name="orderNo"
        type="java.lang.Integer"
        column="orderNo"
        length="1"
    />
    <property
        name="parentId"
        type="java.lang.Integer"
        column="parentId"
        length="11"
    />
</class>
</hibernate-mapping>

```
##  Hibernate.cfg.xml: 
```

<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC 
	"-//Hibernate/Hibernate Configuration DTD//EN"
	"http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>

	<session-factory>
		<property name="hibernate.connection.provider_class">org.hibernate.connection.ProxoolConnectionProvider</property>
		<property name="hibernate.proxool.pool_alias">proxool</property>
		<property name="hibernate.proxool.xml">proxool.xml</property>

		<property name="show_sql">true</property>
		<mapping resource="com/css/demo/model/GUser.hbm.xml" />
		<mapping resource="com/css/demo/model/Group.hbm.xml" />
		<mapping resource="com/css/demo/model/User.hbm.xml" />	
	
	</session-factory>
</hibernate-configuration>

```
proxool.xml:配置jdbc链接。
```

<?xml version="1.0" encoding="utf-8"?>
<something-else-entirely>
<proxool>
<alias>proxool</alias>

<driver-url>jdbc:mysql://localhost:3306/cssdemo</driver-url> 
<driver-class>com.mysql.jdbc.Driver</driver-class>
<driver-properties>
<property name="user" value="root"/>
<property name="password" value="gnodiew"/>
</driver-properties>

<maximum-connection-count>500</maximum-connection-count>
<minimum-connection-count>10</minimum-connection-count>
<house-keeping-sleep-time>30000</house-keeping-sleep-time>

<simultaneous-build-throttle>50</simultaneous-build-throttle>
<prototype-count>2</prototype-count>
<maximum-active-time>172800000</maximum-active-time>
<maximum-connection-lifetime>180000000</maximum-connection-lifetime>
<test-before-use>false</test-before-use> 
</proxool>
</something-else-entirely>

```
##  应用开发 
###  配置类ConfigurationManager.java: 

```

public class ConfigurationManager {
	public static final String MAPPING_FILE = "/hibernate.cfg.xml";
	public static int getCacheTime() {
		/** 缓存10秒，实际项目从系统配置的字典表里取 */
		return 10 * 1000;
	}
	public static int getPageSize() {
		/** 10条记录，实际项目从系统配置的字典表里取 */
		return 10;
	}
}

```
###  QueryCache: 
####  对象查询 

**通过主键查询：**
Group group=QueryCache.get(Group.class, 1);
User user=QueryCache.get(User.class, 2);
**通过语句查询：**
Integer userId =(Integer) new QueryCache("select a.id from User a where a.sexId like :sexId ").setParameter("sexId", 1).setMaxResults(1).uniqueResult();
User user = QueryCache.get(User.class, userId);
System.out.println(user.getName());
**说明：**
  *  qc.uniqueResultCache (); / /从缓存中读取，如果缓存中无数据则从数据库中读取，并自动缓存，缓存超时为系统默认时长，即配置类的getCacheTime() 
  *  qc.uniqueResultCache(time); / /从缓存中读取，如果缓存中无数据则从数据库中读取，并自动缓存，缓存超时为time为毫秒，如time=10*1000为10秒，如果time<0如-1则为永久缓存 
  *  qc.uniqueResult(); / /**不读缓存，直接从数据库中读取**

**列表查询：**
List<Integer> gId = new ArrayList<Integer>();
gId.add(11);
gId.add(12);
QueryCache qc = new QueryCache("select a.id from User a where a.sexId=:sexId and a.groupId in(:groups)") .setParameter("sexId", 1).setParameter("groups", gId);
List<Integer> userIds = qc.listCache();
List<User> users = QueryCache.` idToObj `(User.class, userIds);
for (User user : users)
System.out.println(user.getName());
说明：
 qc.listCache(); / /从缓存中读取，如果缓存中无数据则从数据库中读取，**并自动缓存，**缓存超时为系统默认时长，即配置类的getCacheTime()
 qc.listCache(time); / /从缓存中读取，如果缓存中无数据则从数据库中读取，并自动缓存，缓存超时为time为毫秒，如time=10*1000为10秒，如果time<0如-1则为永久缓存
 qc.list(); / /不读缓存，直接从数据库中读取
**分页查询**
```

Page page = new Page();
page.setCurrentPage(1); //设置当前页
page.setCountField("a.id"); //设置求和字段 即mysql count(a.id)
page.setOrderString("a.id");//设置排序字段,即mysql order by a.id
page.setOrderFlag(Page.OEDER_DESC);//设置排序方式 即mysql order by a.id desc
page.setPageSize(10);//设置每页记录数，不记录则取配置类参数 
StringBuffer sb = new StringBuffer("select a.id from Group a ");
if (parentId != null)
sb.append(" where a.parentId=:parentId ");
QueryCache qc = new QueryCache(sb.toString());
if (parentId != null)
qc.setParameter("parentId", parentId);
page = qc.pageCache(page); 注意这里使用的是pageCache方法。
page.setResults(QueryCache.idToObj(Group.class,page.getResults()));
System.out.println("页数:"+page.getTotalPages());
System.out.println("记录总数:"+page.getTotalRows());
for (Group group : (List<Group>)page.getResults())
System.out.println(group.getName());
}

```
说明：
 qc.pageCache(page); / /从缓存中读取，如果缓存中无数据则从数据库中读取，并自动缓存，缓存超时为系统默认时长，即配置类的getCacheTime()
 qc.pageCache(page,time); / /从缓存中读取，如果缓存中无数据则从数据库中读取，并自动缓存，缓存超时为time为毫秒，如time=10*1000为10秒，如果time<0如-1则为永久缓存
 qc.page(); / /不读缓存，直接从数据库中读取
**原生SQL查询：**
当通过 hql无法实现时，可以通过原生SQL语句查询。调用时将 ` new QueryCache(String hql, boolean naTive) `的** naTive 值设为 true，则表示用原生 SQL查询。**
示例 2：取实体类 a.*
```

QueryCache qc = new QueryCache(
"select a.* from user_info a left join user_group b on a.groupId=b.id where a.sexId=:sexId and b.name like :groupName",true);
qc.addEntity("a", User.class); 什么意思？暂时不明白
qc.setParameter("sexId", 1).setParameter("groupName", "%开发%");
List<Integer> userIds = qc.listCache();
List<User> users = QueryCache.idToObj(User.class, userIds);
for (User user : users)
System.out.println(user.getName());

```
###  TransactionCache 
**添加：**
TransactionCache tx = null;
tx = QueryCache.getTransaction();
tx.save(user);
tx.commit();
说明：
** 新添记录，保存到数据库，同时缓存到memcached**
 save(in)，in参数可以是一个数据模型的Object对象，也可以是一个包含多个数据模型对象的List对象
**修改：**
User user = QueryCache.get(User.class, uId);
user.setName("刘四_edit");
TransactionCache tx = null;
tx = QueryCache.getTransaction();
tx.update(user);
tx.commit();
说明：
 **更新记录，保存到数据库，同时更新缓存**
 update(in)，in参数可以是一个数据模型的Object对象，也可以是一个包含多个数据模型对象的List对象
**删除：**
User user = QueryCache.get(User.class, uId);
TransactionCache tx = null;
tx = QueryCache.getTransaction();
tx.delete(user);
tx.commit();
说明：
 **删除数据库中记录，同时删除缓存**
 update(in)，in参数可以是一个数据模型的Object对象，也可以是一个包含多个数据模型对象的List对象
##  多数据库配置 
1. 先完成 hibernate文件的配置工作
2. 继承 QueryCache类
```

public class QueryDict extends QueryCache {
	private static final String MAPPING = "/hibernatedict.cfg.xml"; 特定的hibernate配置文件/hibernatedict.cfg.xml,字段值固定为MAPPING
	public QueryDict(String hql, boolean naTive) { 注意这三个构造器的写法。
		super(MAPPING, hql, naTive);
	}
	public QueryDict(String hql) {
		super(MAPPING, hql);
	}
	public QueryDict() {
		super(MAPPING, null);
	}
	public static <T> T get(Class<T> clazz, Serializable id) { 注意这几个方法的写法。
		return new QueryDict().get(clazz, id);
	}
	public static List idToObj(Class clazz, List list) {
		return new QueryDict().idToObj(clazz, list);
	}
	public static TransactionCache getTransaction() {
		return new TransactionCache(MAPPING);
	}
}

```
##  常用模型 
###  数据字典模型： 
按框架的DictType类所规定的字段建字典表。
框架自带的` DictType `类：
```

public class DictType implements Serializable {
	private Integer id = null;
	private String description;
	private String remark;

	public DictType() {
		this.description = "";
		this.remark = "";
	}
  。。。

```
框架自带的` DictType.hbm.xml `：
```

<hibernate-mapping>
    <class name="com.css.core.dict.DictType" >
        <id
            name="id"
            column="id"
            type="java.lang.Integer" >

            <generator class="assigned" />
        </id>

        <property
            name="description"
            column="description"
            length="50"
            type="java.lang.String" />

        <property
            name="remark"
            column="remark"
            length="50"
            type="java.lang.String" />
    </class>
</hibernate-mapping>

```
框架自带的` DictMain `类:
List<DictType> getDictType(String table)根据字典表名自动生成对应的DictType对象列表
List<DictType> getDictListQuery(String table)
返回：字典表所对应的列象为 第一个值 为“”
List<DictType> getDictListSel(String table)：
返回：字典表所对应的列，象为，第一个值为“请选择”
DictType getDictType(String table, Serializable value)：参数：字典类据库表名称，**主键值**
###  关联（join）模型 
常用场景：得到一个 group ，想得到当前 group下的用户列表。
关联系：用户类（ User User）通过 groupId与部门 /组别类（ Group ）关联。
可以在 Group类中，通过定义一个 类中，通过定义一个 JoinList users属性变量，得到当前组下 的用户列表：
框架自带的com.css.db.query.` JoinList `:
public JoinList(Class clazz, QueryCache qc) ;
public List getListById();返回用户id列表
public List getList(); 返回用户列表
public void add(Serializable id);
public void removeAll();
public void remove(Serializable id);
```

public class Group implements Serializable {
	/**
	 * 定义关联用户列表
	 */
	private transient JoinList users = null;
	/**
	 * 查询当前组下的用户列表
	 */
	public JoinList getUsers() {
		if (users == null) {
			QueryCache qc = new QueryCache("select a.id from User a where a.groupId=:groupId")
					.setParameter("groupId", id);
			users = new JoinList(User.class, qc);
		}
		return users;
	}

```
###  树（Tree）模型 
继成com.css.db.tree.` Tree `:（**基于memcached通用缓存树模型**）
```

public class GroupTree extends Tree {
	private static GroupTree instance = null;
	private GroupTree() {
	}
	protected Class getTreeClass() {
		return Group.class;
	}
	public static synchronized GroupTree getInstance() {
		if (instance == null)
			instance = new GroupTree();
		instance.reloadTreeCache();
		return instance;
	}
	public TreeNode transform(Object info) {
		Group group = (Group) info;
		TreeNode node = new TreeNode();
		node.setNodeId(group.getId().toString());
		node.setParentId(group.getParentId() == null ? "" : group.getParentId().toString());
		node.setOrderNo(group.getOrderNo() == null ? 0 : group.getOrderNo());
		return node;
	}
	public void reloadTree() {
		List nodes = new QueryCache("select a.id from Group a ").list();
		nodes = QueryCache.idToObj(Group.class, nodes);
		Group root = new Group();
		root.setParentId(null);
		root.setId(0);
		root.setOrderNo(0);
		nodes.add(root);
		super.reload(nodes);
	}
}


```
示例：
```

GroupTree gt = GroupTree.getInstance();
		System.out.println("-------------------根节点下的所有节点：");
		List<TreeNode> lst1 = gt.getRootNode().getAllChildren();
		for (TreeNode item : lst1)
			printGroup(item);
		System.out.println("-------------------根节点下一层的节点：");
		List<TreeNode> lst2 = gt.getRootNode().getChildren();
		for (TreeNode item : lst2)
			printGroup(item);
		System.out.println("-------------------id=1技术部下层节点：");
		TreeNode tn = gt.getTreeNode("1");// 获取技术部所在组id="1"的树节点TreeNode对象
		List<TreeNode> lst3 = tn.getChildren();
		List<Group> groups = gt.getList(lst3);
		for (Group group : groups)
			System.out.println(group.getId() + ":" + group.getName());
		System.out.println("-------------------id=1技术部下层节点id：");
		List<Integer> groupIds = gt.getListById(lst3, true); // id类型String,Integer, true表示是Integer，省略或false表示String，
		for (Integer id : groupIds)
			System.out.println(id);

public static void printGroup(TreeNode item) {
		Group group = QueryCache.get(Group.class, Integer.parseInt(item.getNodeId()));
		if (group != null)
			System.out.println(group.getId() + ":" + group.getName());
	}

```
###  常用TreeNode方法： 
List<TreeNode> getChildren()
boolean isLeaf() 
List<TreeNode> getAllChildren()
。。。。。