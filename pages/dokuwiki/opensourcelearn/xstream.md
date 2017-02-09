title: xstream 

#  XStream解析XML为POJO 
可供参考的文档：
http://www.oschina.net/code/snippet_116183_14202
http://www.cnblogs.com/hoojo/archive/2011/04/22/2025197.html
特点: 
简化的API; 
无映射文件; 
高性能,低内存占用; 
整洁的XML; 
不需要修改对象;支持内部私有字段,不需要setter/getter方法,final字段;非公有类,内部类;类不需要默认构造器,完全对象图支持.维护对象引用计数,循环引用. i 
提供序列化接口; 
自定义转换类型策略; 
详细的错误诊断; 
快速输出格式;当前支持 JSON 和 morphing. 

使用场景 
Transport 转换 
Persistence 持久化对象 
Configuration 配置 
Unit Tests 单元测

**安装：**
```

		<dependency>
			<groupId>com.thoughtworks.xstream</groupId>
			<artifactId>xstream</artifactId>
			<version>1.4.7</version>
		</dependency>

```
2.Xstream注解常用知识： 
@XStreamAlias("message") 别名注解  
作用目标: 类,字段  
@XStreamImplicit 隐式集合  
@XStreamImplicit(itemFieldName="part")  
作用目标: 集合字段  
@XStreamConverter(SingleValueCalendarConverter.class) 注入转换器  
作用目标: 对象  
@XStreamAsAttribute 转换成属性  
作用目标: 字段  
@XStreamOmitField 忽略字段  
作用目标: 字段  
Auto-detect Annotations 自动侦查注解   
` xstream.autodetectAnnotations(true);  ` 
自动侦查注解与XStream.processAnnotations(Class[] cls)的区别在于性能.自动侦查注解将缓存所有类的类型.  
3.案例分析： （1）同一标签下多个同名元素； 
（2）同一标签下循环多个对象；
```

<person name="haha">
  <firstName>chen</firstName>
  <lastName>youlong</lastName>
  <telphone>
    <code>137280</code>
    <number>137280968</number>
  </telphone>
  <faxphone>
    <code>20</code>
    <number>020221327</number>
  </faxphone>
  <friends>
    <name>A1</name>
    <name>A2</name>
    <name>A3</name>
  </friends>
  <pets>
    <pet>
      <name>doly</name>
      <age>2</age>
    </pet>
    <pet>
      <name>Ketty</name>
      <age>2</age>
    </pet>
  </pets>
</person>

```
```

@XStreamAlias("person")
public class PersonBean {
  @XStreamAsAttribute //属性
        private String name;	
    @XStreamAlias("firstName")  //子元素
    private String firstName;
    @XStreamAlias("lastName")
    private String lastName;
     
    @XStreamAlias("telphone")
    private PhoneNumber tel;
    @XStreamAlias("faxphone")
    private PhoneNumber fax;
     
    //测试一个标签下有多个同名标签
    @XStreamAlias("friends")
    private Friends friend;
     
    //测试一个标签下循环对象
    @XStreamAlias("pets")
    private Pets pet;
     
     
    //省略setter和getter
}

```
```

@XStreamAlias("phoneNumber")
    public  class PhoneNumber{
        @XStreamAlias("code")
        private int code;
        @XStreamAlias("number")
        private String number;
         
            //省略setter和getter
         
    }

```
```

/**
     * 用Xstream注解的方式实现：一个标签下有多个同名标签 
     *@ClassName:Friends
     *@author: chenyoulong  Email: chen.youlong@payeco.com
     *@date :2012-9-28 下午4:32:24
     *@Description:TODO 5个name 中国，美国，俄罗斯，英国，法国
     *http://blog.csdn.net/menhuanxiyou/article/details/5426765
     */
    public static class Friends{
        @XStreamImplicit(itemFieldName="name")   //itemFieldName定义重复字段的名称，
        /*<friends>                               <friends>
            <name>A1</name>                         <String>A1</String>
            <name>A2</name>    如果没有，则会变成    =====>       <String>A1</String>
            <name>A3</name>                         <String>A1</String>
        </friends>                                </friends>
      */
        private List<String> name;
 
        public List<String> getName() {
            return name;
        }
 
        public void setName(List<String> name) {
            this.name = name;
        }
    }

```
```

//测试同一标签下循环某一对象
    public  class Animal{
        @XStreamAlias("name")
        private String name;
        @XStreamAlias("age")
        private int age;
        public Animal(String name,int age){
            this.name=name;
            this.age=age;
        }
         
              //省略setter和getter
    }

```
```

/**
     * 测试同一标签下循环某一对象
     *@ClassName:Pets
     *@author: chenyoulong  Email: chen.youlong@payeco.com
     *@date :2012-9-28 下午6:26:01
     *@Description:TODO
     */
    public class Pets{
        @XStreamImplicit(itemFieldName="pet")
        private List<Animal> animalList;
         
        public List<Animal> getAnimalList() {
            return animalList;
        }
 
        public void setAnimalList(List<Animal> animalList) {
            this.animalList = animalList;
        }
         
    }

```
```

public static void main(String[] args) {
        // TODO Auto-generated method stub
         
        PersonBean per=new PersonBean();
        per.setFirstName("chen");
        per.setLastName("youlong");
         
        PhoneNumber tel=new PhoneNumber();
        tel.setCode(137280);
        tel.setNumber("137280968");
         
        PhoneNumber fax=new PhoneNumber();
        fax.setCode(20);
        fax.setNumber("020221327");
        per.setTel(tel);
        per.setFax(fax);
         
         
        //测试一个标签下有多个同名标签
        List<String> friendList=new ArrayList<String>();
        friendList.add("A1");
        friendList.add("A2");
        friendList.add("A3");
        Friends friend1=new Friends();
        friend1.setName(friendList);
        per.setFriend(friend1);
         
        //测试一个标签下循环对象
        Animal dog=new Animal("Dolly",2);
        Animal cat=new Animal("Ketty",2);
        List<Animal> petList=new ArrayList<Animal>();
        petList.add(dog);
        petList.add(cat);
        Pets pet=new Pets();
        pet.setAnimalList(petList);
        per.setPet(pet);
         
                    //java对象转换成xml
        String xml=XmlUtil.toXml(per);
        System.out.println("xml==="+xml);
         
    }

```
```

/**
     * 输出xml和解析xml的工具类
     *@ClassName:XmlUtil
     *@author: chenyoulong  Email: chen.youlong@payeco.com
     *@date :2012-9-29 上午9:51:28
     *@Description:TODO
     */
    public class XmlUtil{
        /**
         * java 转换成xml
         * @Title: toXml 
         * @Description: TODO 
         * @param obj 对象实例
         * @return String xml字符串
         */
        public static String toXml(Object obj){
            XStream xstream=new XStream();
//          XStream xstream=new XStream(new DomDriver()); //直接用jaxp dom来解释
//          XStream xstream=new XStream(new DomDriver("utf-8")); //指定编码解析器,直接用jaxp dom来解释
             
            ////如果没有这句，xml中的根元素会是<包.类名>；或者说：注解根本就没生效，所以的元素名就是类的属性
            xstream.processAnnotations(obj.getClass()); //通过注解方式的，一定要有这句话
            return xstream.toXML(obj);
        }
         
        /**
         *  将传入xml文本转换成Java对象
         * @Title: toBean 
         * @Description: TODO 
         * @param xmlStr
         * @param cls  xml对应的class类
         * @return T   xml对应的class类的实例对象
         * 
         * 调用的方法实例：PersonBean person=XmlUtil.toBean(xmlStr, PersonBean.class);
         */
        public static <T> T  toBean(String xmlStr,Class<T> cls){
            //注意：不是new Xstream(); 否则报错：java.lang.NoClassDefFoundError: org/xmlpull/v1/XmlPullParserFactory
            XStream xstream=new XStream(new DomDriver());
            xstream.processAnnotations(cls);
            T obj=(T)xstream.fromXML(xmlStr);
            return obj;         
        } 
 
       /**
         * 写到xml文件中去
         * @Title: writeXMLFile 
         * @Description: TODO 
         * @param obj 对象
         * @param absPath 绝对路径
         * @param fileName  文件名
         * @return boolean
         */
       
        public static boolean toXMLFile(Object obj, String absPath, String fileName ){
            String strXml = toXml(obj);
            String filePath = absPath + fileName;
            File file = new File(filePath);
            if(!file.exists()){
                try {
                    file.createNewFile();
                } catch (IOException e) {
                    log.error("创建{"+ filePath +"}文件失败!!!" + Strings.getStackTrace(e));
                    return false ;
                }
            }// end if 
            OutputStream ous = null ;
            try {
                ous = new FileOutputStream(file);
                ous.write(strXml.getBytes());
                ous.flush();
            } catch (Exception e1) {
                log.error("写{"+ filePath +"}文件失败!!!" + Strings.getStackTrace(e1));
                return false;
            }finally{
                if(ous != null )
                    try {
                        ous.close();
                    } catch (IOException e) {
                        log.error("写{"+ filePath +"}文件关闭输出流异常!!!" + Strings.getStackTrace(e));
                    }
            }
            return true ;
        }
         
        /**
         * 从xml文件读取报文
         * @Title: toBeanFromFile 
         * @Description: TODO 
         * @param absPath 绝对路径
         * @param fileName 文件名
         * @param cls
         * @throws Exception 
         * @return T
         */
        public static <T> T  toBeanFromFile(String absPath, String fileName,Class<T> cls) throws Exception{
            String filePath = absPath +fileName;
            InputStream ins = null ;
            try {
                ins = new FileInputStream(new File(filePath ));
            } catch (Exception e) {
                throw new Exception("读{"+ filePath +"}文件失败！", e);
            }
             
            String encode = useEncode(cls);
            XStream xstream=new XStream(new DomDriver(encode));
            xstream.processAnnotations(cls);
            T obj =null;
            try {
                obj = (T)xstream.fromXML(ins);
            } catch (Exception e) {
                // TODO Auto-generated catch block
                throw new Exception("解析{"+ filePath +"}文件失败！",e);
            }
            if(ins != null)
                ins.close();
            return obj;         
        } 
         
    }

```