title: jmockit框架 

#  Jmockit框架 
JMockit 是一个轻量级的mock框架是用以帮助开发人员编写测试程序的一组工具和API，该项目完全基于 Java 5 SE 的 java.lang.instrument 包开发，内部使用 ASM 库来修改Java的Bytecode。
如果junit版本是4.x, 需要4.8以上的版本, 而且在设置classpath时jmockit.jar的路径要设置在junit.jar前. 这样保证使用jmockit的runner加载junit
官网：http://jmockit.org/gettingStarted.html
![](/data/dokuwiki/opensourcelearn/pasted/20151230-155251.png)

```

<dependency>
    <groupId>org.jmockit</groupId>
    <artifactId>jmockit</artifactId>
    <version>1.21</version>
   <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.jmockit</groupId>
    <artifactId>jmockit-coverage</artifactId>
    <version>1.21</version>
   <scope>test</scope>
</dependency>

```
##  JMockit使用实例  
关键词：如何mock一个类的方法、Expectations 
```

public class DateUtil {  
  
    private int type;  
  
    public static final String getCurrentDateStr() {  
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
        return sdf.format(DateUtil.now());  
    }  
  
    public static final String getCurrentDateStrByFormatType(int type) {  
        if (type == 1) {  
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");  
            return sdf.format(DateUtil.now());  
        } else {  
            return DateUtil.getCurrentDateStr();  
        }  
    }  
  
    public static final Date now() {  
        return new Date();  
    }  
  
    public int getType() {  
        return type;  
    }  
  
    public void setType(int type) {  
        this.type = type;  
    }  
  
}  

```
```

public class DateUtilTest {  
  
    /** 
     * Mock某个类方法 
     */  
    @Test  
    public void testGetCurrentDateStr() {  
        //DateUtil.class,要Mock的类  
        new Expectations(DateUtil.class) {  
            {  
              //要Mock的方法now,其他方法DateUtil.class  
                DateUtil.now();  
              //期望方法返回的结果  
                result = mockDate();  
            }  
        };  
        Assert.assertEquals("2010-07-22 15:52:55", DateUtil.getCurrentDateStr());  
    }  
  
    /** 
     * Mock 某个类方法根据不同参数返回不同值 
     */  
    @Test  
    public void testGetCurrentDateStrByFormatType() {  
        new Expectations(DateUtil.class) {  
            {  
                DateUtil.getCurrentDateStrByFormatType(anyInt);  
                result = new Delegate() {  
                    public String getCurrentDateStrByFormatType(int type) {  
                        if (type == 1) {  
                            return "2010/07/22 15:52:55";  
                        } else {  
                            return "2010-07-22 15:52:55";  
                        }  
                    }  
                };  
            }  
        };  
        Assert.assertEquals("2010-07-22 15:52:55", DateUtil.getCurrentDateStrByFormatType(2));  
  
    }  
  
    public static Date mockDate() {  
        Calendar c = Calendar.getInstance();  
        c.set(2010, 6, 22, 15, 52, 55);  
        return c.getTime();  
    }  
  
}

```
小结 
Expectations：一个Expectations块是给定测试方法中将会涉及到的mock项中，预期将要被调用的方法或构造函数。一个Expectations可以包含多个预期的要执行方法(mock)，但不必包含所有预期会被调用的方法。在Expectations中；除了可以指定预期的方法外，还可以指定方法的参数的精确值或约束行为（满足某个断言）；同时Expectations中还可以指定该方法预期的返回值（如果有）或预期抛出的异常。Expectations(.class){}这种方式只会模拟区域中包含的方法，这个类的其它方法将按照正常的业务逻辑运行，上面的例子，定义了一个mock类DateUtil，同时在Expectation中定义了预期会被调用的方法now，以及now方法的返回值，这种方式还有种等价实现方式，使用@Mocked标签 

case-2 
关键词：service dao @Mocked 
```

public interface P4psettleDayService {  
    public List<DaySyncVerifyDO> getDaySyncVerifyNotSuccessList();  
  
    public void updataStatusById(Integer id,String status);  
} 

```
```

@TestCaseInfo(contextKey = "P4PSettleServicesLocator", defaultRollBack = false)  
public class P4psettleDayServiceImplTest extends BaseTestCase {  
  
    private P4psettleDayService p4psettleDayService;  
  
    public void setP4psettleDayService(P4psettleDayService p4psettleDayService) {  
        this.p4psettleDayService = p4psettleDayService;  
    }  
  
    /** 
     * 此例子仅为说明如何Mock我们现有service或dao的部分方法 
     */  
    @Test  
    public void testUpdataStatusById() {  
            new MockUp<P4psettleDayServiceImpl>(){  
              @Mock  
              public List<DaySyncVerifyDO> getDaySyncVerifyNotSuccessList(){  
                List<DaySyncVerifyDO> list = new ArrayList<DaySyncVerifyDO>();  
                DaySyncVerifyDO mockDO = new DaySyncVerifyDO();  
                mockDO.setId(111111);  
                list.add(mockDO);  
                return list;  
              }  
             };  
        //这里将返回我们mock的数据  
        List<DaySyncVerifyDO> list = p4psettleDayService  
                .getDaySyncVerifyNotSuccessList();  
        if (list != null && list.size() > 0) {  
            for (DaySyncVerifyDO item : list) {  
                //这里执行原有的方法  
                p4psettleDayService.updataStatusById(item.getId(),  
                        DayEnum.DAYSYNCVERIFY_STATUS_SUCCESS.getValue());  
            }  
        }  
    }  
  
}  

```
case-3 
关键词：mock private方法 invoke 
源类清单 
还是第一个例子，Now方法是私有的 
```

public class DateUtil {  
    ......  
    private static final Date now() {  
        return new Date();  
    }  
    ......  
  
}

```
```

public class DateUtilTest {  
    /** 
     * Mock某个类私有方法 
     */  
    @Test  
    public void testGetCurrentDateStr() {  
        //DateUtil.class,要Mock的类  
        new Expectations(DateUtil.class) {  
            {  
              //执行DateUtil的now方法  
                invoke(DateUtil.class,"now");  
              //期望方法返回的结果  
                result = mockDate();  
            }  
        };  
        Assert.assertEquals("2010-07-22 15:52:55", DateUtil.getCurrentDateStr());  
    }  

```
小结 
mock 某个类的私有方法，用invoke(mock的类或实例，方法名，方法的参数列表) 
case-4 
关键词：Verifications 想验证被Mock的类的某个方法是否被调用 
```

/** 
 * 演示验证被Mock的类的某个方法是否被调用 
 * 
 * @author LeiSir 
 */  
public class ServiceTest {  
  
    @Mocked  
    Remote remote;  
  
    @Test  
    public void testDoFuncYes() {  
        Service service = new Service();  
        service.doFunc(true, 1);  
        new Verifications() {  
            {  
                remote.doSomething(anyInt);//表示这个方法会被执行  
                //remote.doSomething(1);//表示这个方法会被执行，而且参数是1；在当前case，会通过  
                //remote.doSomething(2);//表示这个方法会被执行，而且参数是2；在当前case，这个会不被通过  
  
            }  
        };  
  
    }  
  
    @Test  
    public void testDoFuncNo() {  
        Service service = new Service();  
        service.doFunc(false, 1);  
        new Verifications() {  
            {  
                remote.doSomething(anyInt);  
                times = 0;//调用次数，0表示上面方法不会被调用  
            }  
        };  
    }  
  
    private static class Remote {  
        public void doSomething(int a) {  
        }  
    }  
  
    private static class Service {  
  
        private Remote remote = new Remote();  
  
        public void doFunc(boolean flag, int a) {  
            if (flag) {  
                remote.doSomething(a);  
            }  
        }  
    }  
  
}  

```
有时候我们想验证某个类的方法是否被正确调用的时候，上述Verifications就派上用场了 

```

public class UserServiceAPITest {

    @BeforeClass
    static public void beforeClass() {
        Mockit.setUpMocks();

    }

    // 代表要打桩的类，在UserService中用户了UserManager类，对此类进行打桩
    @Mocked
    private UserManager userManager = null;
    final String account = "13810481296";

    // 待测试类的实例
    private UserService userService = new UserService();

    @Test
    public void testAuthorizeUser() throws ApiCallException {
  // 也可以是NonStrictExpectations//非严格的，所有声明的调用，声明的次数，返回的结果不用完全匹配  
   // 这种是严格的，所有声明的调用，声明的次数，返回的结果都会完全匹配  
        // 开始打桩
        new Expectations() {
            {
                // 期望被mock的调用，以及被调用时返回的结果
                userManager.getUser(account);
                UserEntity userEntity;
                userEntity = new UserEntity();
                userEntity.setUserId(13L);
                userEntity.setPassword(MD5Util.MD5(account));
                userEntity.setMsisdn(account);
                userEntity.setMsisdnStatus(1);
                result = userEntity; // 也可以是returns(false);

                // 总共可以调用的次数
                times = 1;
            }
        };

        // 验证数据，与打桩数据相同
        UserInfo userInfo = new UserInfo();
        userInfo.userId = 13L;
        userInfo.password = MD5Util.MD5(account);
        userInfo.msisdn = account;
        userInfo.msisdnStatus = 1;
//      userService.authorizeUser(account, account);

        // 步骤二、replay 在此阶段，录制的方法被调用  
        Assert.assertEquals(userInfo.getUserId(), userService.authorizeUser(account, account)
                .getUserId());
    }
｝
//测试样例2，有Spring依赖及void方法测试
public class TagServiceTest {

    @BeforeClass
    static public void beforeClass() {
        Mockit.setUpMocks();

    }

    @Tested
    private TagServiceImpl tagServiceImplService = new TagServiceImpl();

    @Injectable
    TagManager tagManager;

    private String name = "test";

    @Test
    public void testFindByName() throws Exception {

        // 开始打桩
        new Expectations() {
            {
                // 期望被mock的调用，以及被调用时返回的结果
//              tagManager = new TagManager();
                tagManager.findByName(name, true);
                List<TagEntity> tagList = new ArrayList<TagEntity>();
                TagEntity tagEntity = new TagEntity();
                tagEntity.setId(1);
                tagEntity.setName("test");
                tagEntity.setCategory(1);
                tagEntity.setDesc("desc");
                tagList.add(tagEntity);
                result = tagList; // 也可以是returns(false);

                // 总共可以调用的次数
                times = 1;
            }
        };

        // 验证数据，与打桩数据相同
        List<TagEntity> tagList = new ArrayList<TagEntity>();
        TagEntity tagEntity = new TagEntity();
        tagEntity.setId(1);
        tagEntity.setName("test");
        tagEntity.setCategory(1);
        tagEntity.setDesc("desc");
        tagList.add(tagEntity);

        List<TagEntity> assertList = new ArrayList<TagEntity>();
        assertList.add(tagEntity);
        List<TagEntity> returnList = tagServiceImplService.findByName(name, true);
        Assert.assertEquals(assertList.get(0).getId(), returnList.get(0).getId());

    }

    @Test
    public void testDelete() throws Exception {

        // 开始打桩
        new Expectations() {
            {
                // 期望被mock的调用，以及被调用时返回的结果
//              tagManager = new TagManager();
                tagManager.delete(1);

                times = 1;
            }
        };

        // 验证数据，与打桩数据相同
        tagServiceImplService.delete(1);

    }

}

```
##  MockUp类应用 
Jmockit提供了多种mock方式共开发者使用, 但给我感觉最有用的一个类就是MockUp. 用它几乎能完成所有需要mock的操作:
mock接口. 使用MockUp.getMockInstance()方便mock接口, 特别是定义了多个方法的接口. 手工打桩需要写一个此接口的假实现, 但测试中只调用了接口的一个方法, 造成了其他没调用方法还要写一堆没用的实现. 用了MockUp打桩就只关注需要mock的方法即可:
```

public interface IService {
    void doSth();

    void doOtherthing();
}

```
接口中有两个方法, 使用Mock返回一个mock对象时只关注被调用的方法doSth()即可:
```

import mockit.Mock;
import mockit.MockUp;
import mockit.Mockit;

import org.junit.Test;

public class TestCase {

    @Test
    public void testSth() throws Exception {
        IService service = new MockUp<IService>() {
            // 需要mock哪个方法就只写哪个方法的mock实现, 其他方法都可以忽略
            @Mock
            public void doSth() {
                System.out.println("this is mock implement");
            }
        }.getMockInstance();

        // 调用mock方法
        service.doSth();

        // 最后做还原动作, 确保用例之间不相互影响. 也可以放到test case的teardown方法中
        Mockit.tearDownMocks();
    }
}

```
mock final类或静态方法. 对于final类或静态方法, 其他mock工具基本没有好的办法, 但用MockUp就非常简单:
```

public class Utils {

    public static String getFormatStr()throws Exception{
        return "YYYY-mm-dd HH:MM:SS";
    }
}

package cn.outofmemory.demo.jmockit;

import mockit.Mock;
import mockit.MockUp;
import mockit.Mockit;

import org.junit.After;
import org.junit.Test;

public class TestCase {
    @Test
    public void testMockStaticMethod() {
        // 打印mock前返回值
        System.out.println(Utils.getFormatStr());

        new MockUp<Utils>() {
                     //除了static关键字, 其他方法定义内容保持与被mock方法一致(包括异常定义)
            @Mock
            public String getFormatStr() throws Exception {
                return "OtherFormat: dd-mm-YYYY";
            }
        };
        // 打印mock后返回值
        System.out.println(Utils.getFormatStr());
    }

    @After
    public void tearDown() {
        // 最后做还原动作, 确保用例之间不相互影响.
        Mockit.tearDownMocks();
    }
}

```
参考：http://outofmemory.cn/code-snippet/3275/Jmockit-tips

##  完整的Mock步骤 
```

import jmockit.target.OfferPostAction;  
import jmockit.target.WinportUrlServiceImpl;  
import junit.framework.Assert;  
import mockit.Mocked;  
import mockit.NonStrictExpectations;  
import mockit.Verifications;  

import org.junit.Test;  

/** 
 * 一个完整的Mock会有三个步骤，步骤一、record （录制）；步骤二、replay 在此阶段，录制的方法可能会被调用；步骤三、验证。如果是Expectations就没有必要做Verifications了。 
 * @author Ginge 
 * 
 */  
public class RecordReplayVerificationTest {  

    @Mocked  
    private WinportUrlServiceImpl winportUrlService = null;  

    private OfferPostAction offerPostAction = new OfferPostAction();  

    @Test  
    public void testofferPostActionExecute() {  
        final String memberId = "test2009";  
        // 步骤一、record （录制）  
        new NonStrictExpectations() {  
            {  
                // 期望被mock的调用，以及被调用时返回的结果  
                winportUrlService.hasWinport(memberId);  
                result = false; // 也可以是returns(false);  
                // 总共可以调用的次数  
                times = 1;  
            }  
        };  

        // 步骤二、replay 在此阶段，录制的方法可能会被调用  
        Assert.assertEquals(false, offerPostAction.hasWinport(memberId));  

        try{  
            offerPostAction.getWinportUrlThrowException(memberId);  
        }catch(Exception e){  
            //fully mock，默认完全被mock，到这里就注定失败  
            Assert.fail();  
        }  

        // 步骤三、验证步骤二中，mock方法是否被调用，本步骤可以省略  
        new Verifications() {  
            {  
                winportUrlService.hasWinport(withAny(""));  
                times = 1;  
            }  
        };  
    }  
}  

```
```

public class OfferPostAction {  
  
    private WinportUrlServiceImpl winportUrlService = new WinportUrlServiceImpl();  
  
    public boolean hasWinport(String memberId) {  
        return winportUrlService.hasWinport(memberId);  
    }  
      
    public String getWinportUrlThrowException(String memberId){  
        return winportUrlService.getWinportUrlThrowException(memberId);  
    }  
      
    public long getPostedOfferCounts(String memberId){  
        return winportUrlService.getPostedOfferCounts(memberId);  
    }  
}  

```
##  基于行为的Mock方式 
非常类似与EasyMock和PowerMock的工作原理，基本步骤为：
1.录制方法预期行为。
2.真实调用。
3.验证录制的行为被调用。
JMockit也可以分类为非局部模拟与局部模拟，区分在于Expectations块是否有参数，有参数的是局部模拟，反之是非局部模拟。
而Expectations块一般由Expectations类和NonStrictExpectations类定义，类似于EasyMock和PowerMock中的Strict Mock和一般性Mock。
**用Expectations类定义的，则mock对象在运行时只能按照 Expectations块中定义的顺序依次调用方法，不能多调用也不能少调用，所以可以省略掉Verifications块；**

**而用NonStrictExpectations类定义的，则没有这些限制，所以如果需要验证，则要添加Verifications块。**
###  Mock私有成员与静态成员 
1、模拟静态方法：
```

@Test  
public void testMockStaticMethod() {  
    new NonStrictExpectations(ClassMocked.class) {  
        {  
            ClassMocked.getDouble(1);//也可以使用参数匹配：ClassMocked.getDouble(anyDouble);  
            result = 3;  
        }  
    };  
  
    assertEquals(3, ClassMocked.getDouble(1));  
  
    new Verifications() {  
        {  
            ClassMocked.getDouble(1);  
            times = 1;  
        }  
    };  
}  

```
2、模拟私有方法：
如果ClassMocked类中的getTripleString(int)方法指定调用一个私有的multiply3(int)的方法，我们可以使用如下方式来Mock：
```

@Test  
public void testMockPrivateMethod() throws Exception {  
    final ClassMocked obj = new ClassMocked();  
    new NonStrictExpectations(obj) {  
        {  
            this.invoke(obj, "multiply3", 1);//如果私有方法是静态的，可以使用：this.invoke(null, "multiply3")  
            result = 4;  
        }  
    };  
  
    String actual = obj.getTripleString(1);  
    assertEquals("4", actual);  
  
    new Verifications() {  
        {  
            this.invoke(obj, "multiply3", 1);  
            times = 1;  
        }  
    };  
} 

```
3、设置Mock对象私有属性的值：
```

@Test  
public void testMockPrivateProperty() throws IOException {  
    final ClassMocked obj = new ClassMocked();  
    new NonStrictExpectations(obj) {  
        {  
            this.setField(obj, "name", "name has bean change!");  
        }  
    };  
  
    assertEquals("name has bean change!", obj.getName());  
}  

```
4、使用JMockit设置静态私有属性：
```

@Test  
public void testMockPrivateStaticProperty() throws IOException {  
    new NonStrictExpectations(Class3Mocked.class) {  
        {  
            this.setField(ClassMocked.class, "className", "className has bean change!");  
        }  
    };  
  
    assertEquals("className has bean change!", ClassMocked.getClassName());  
} 

```
##  基于状态的Mock方式：改写方法 
JMockit上面的基于行为Mock方式和传统的EasyMock和PowerMock流程基本类似，相当于把被模拟的方法当作黑盒来处理，而JMockit的基于状态的Mock可以直接改写被模拟方法的内部逻辑，更像是真正意义上的白盒测试，下面通过简单例子介绍JMockit基于状态的Mock。
```

public class StateMocked {  
      
    public static int getDouble(int i){  
        return i*2;  
    }  
      
    public int getTriple(int i){  
        return i*3;  
    }  
}  

```
改写普通方法内容：
```

@Test  
public void testMockNormalMethodContent() throws IOException {  
    StateMocked obj = new StateMocked();  
    new MockUp<StateMocked>() {//使用MockUp修改被测试方法内部逻辑  
        @Mock  
      public int getTriple(int i) {  
            return i * 30;  
        }  
    };  
    assertEquals(30, obj.getTriple(1));  
    assertEquals(60, obj.getTriple(2));  
    Mockit.tearDownMocks();//注意：在JMockit1.5之后已经没有Mockit这个类，使用MockUp代替，mockUp和tearDown方法在MockUp类中  
} 

```
修改静态方法的内容：
基于状态的JMockit改写静态/final方法内容和测试普通方法没有什么区别，需要注意的是在MockUp中的方法除了不包含static关键字以外，其他都和被Mock的方法签名相同，并且使用@Mock标注，测试代码如下：
```

@Test  
    public void testGetTriple() {  
        new MockUp<StateMocked>() {  
            @Mock    
            public int getDouble(int i){    
                return i*20;    
            }  
        };    
        assertEquals(20, StateMocked.getDouble(1));    
        assertEquals(40, StateMocked.getDouble(2));   
    }  

```
后续参考：http://blog.csdn.net/ultrani/article/details/8993364
http://blog.csdn.net/chjttony/article/details/17838693
http://ginge.iteye.com/blog/841203
http://blog.csdn.net/songun/article/details/12233721
参考：http://cduym.iteye.com/blog/1905037