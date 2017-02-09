title: spring单元测试 

#  Spring单元测试 
参考http://docs.spring.io/spring/docs/4.3.0.BUILD-SNAPSHOT/spring-framework-reference/htmlsingle/#integration-testing-annotations-spring
使用spring中对Junit框架的整合功能
除了junit4和spring的jar包，还需要**spring-test.jar**。引入如下依赖：
```

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>3.2.15.RELEASE</version>
   <scope>test</scope>
</dependency>

```
```

import static org.Junit.Assert.assertEquals; 
 
import org.Junit.Test; 
import org.Junit.runner.RunWith; 
import org.Springframework.beans.factory.annotation.Autowired; 
import org.Springframework.test.context.ContextConfiguration; 
import org.Springframework.test.context.Junit4.SpringJUnit4ClassRunner; 
import org.Springframework.transaction.annotation.Transactional; 
 
import domain.Account; 
 
@RunWith(SpringJUnit4ClassRunner.class) 
@ContextConfiguration({"classpath:applicationContext.xml"}) 
 
@Transactional
public class AccountServiceTest1 { 
    @Autowired
    private AccountService service; 
    @Before //在每个测试用例方法之前都会执行  
    public void init(){  
    }  
      
    @After //在每个测试用例执行完之后执行  
    public void destory(){  
    }  
    @Test
    public void testGetAcccountById() { 
Account acct = Account.getAccount(1, "user01", 18, "M"); 
        service.insertIfNotExist(acct); 
        Account acct2 = service.getAccountById(1); 
        assertEquals(acct,acct2); 
    } 
}

```
对这个类解释一下：
  * @RunWith 注释标签是 Junit 提供的，用来说明此测试类的运行者，这里用了 **SpringJUnit4ClassRunner**，这个类是一个针对 Junit 运行环境的自定义扩展，用来标准化在 Spring 环境中 Junit4.5 的测试用例，例如支持的注释标签的标准化
  * @ContextConfiguration 注释标签是 Spring test context 提供的，用来指定 Spring 配置信息的来源，支持指定 XML 文件位置或者 Spring 配置类名，这里我们指定 classpath 下的 /config/Spring-db1.xml 为配置文件的位置
  * ` @Transactional 注释标签是表明此测试类的事务启用，这样所有的测试方案都会自动的 rollback `，即您不用自己清除自己所做的任何对数据库的变更了
  * @Autowired 体现了我们的测试类也是在 Spring 的容器中管理的，他可以获取容器的 bean 的注入，您不用自己手工获取要测试的 bean 实例了
  * testGetAccountById 是我们的测试用例：注意和上面的 AccountServiceOldTest 中相同的测试方法的对比，这里我们不用再 try-catch-finally 了，**事务管理自动运行，当我们执行完成后，所有相关变更会被自动清**除
**` 测试时，默认回滚事务，如果想对某个方法提交事务，那么可以使用@Commit或者@Rollback(false) `**
```

@Commit
@Test
public void testProcessWithoutRollback() {
    // ...
}
@Rollback(false)
@Test
public void testProcessWithoutRollback() {
    // ...
}

```

方式二：手动加载spring的配置文件，并启动spring容器
```

public class ReadDaoImplTest {  
    public  static void main(String[] args){  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");    
        context.start();  
        ReadDao fqaService = (ReadDao) context.getBean("readDao");  
        System.out.println(fqaService);  
    }  
      
}  

```

##  SpringMVC单元测试 
```

@Controller  
@RequestMapping(value = "/spring")  
public class Action {  
  
    @Autowired  
    Teacher teacher;  
    // spring 支持restful的格式  
      
    @ResponseBody  
    @RequestMapping(value = "/rest/{ownerId}.do", method = RequestMethod.GET)  
    public String findOwner(@PathVariable String ownerId, Model model,  
            HttpServletResponse rep) throws IOException {  
        return ownerId;  
    }  
  
    @RequestMapping(value = "/test.do", method = RequestMethod.GET)  
    public String testa(Model model, HttpServletResponse rep)  
            throws IOException {  
        model.addAttribute("abc", "efd");  
        model.addAttribute(teacher);  
        return "a";  
    }  
  
    @ResponseBody  
    // 理论上可以@ResponseBody 支持直接返回teacher对象 但是3.2里有问题 我们还是老实返回字符串吧  
    @RequestMapping(value = "/testb.do", method = RequestMethod.GET)  
    public String testb(Model model, HttpServletResponse rep,  
            HttpServletRequest req, String ex) throws IOException {  
        // WEB中获得SPRING容器  
        WebApplicationContext wac = WebApplicationContextUtils  
                .getRequiredWebApplicationContext(req.getServletContext());  
        return new JSONObject(wac.getBean(Teacher.class)).toString();  
    }  
  
    @ResponseBody  
    @RequestMapping(value = "/post.do", method = RequestMethod.POST)  
    public String post(Model model, HttpServletResponse rep,  
            HttpServletRequest req, String ex) throws IOException {  
        return new JSONObject(req.getParameterMap()).toString();  
    }  
  
}  

```
```

import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.*;  
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.*;  
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;  
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestMethod;  
import org.springframework.web.context.WebApplicationContext;  

@RunWith(SpringJUnit4ClassRunner.class)  
@WebAppConfiguration  
@ContextConfiguration(locations = { "classpath:applicationContext.xml" })  
//记得要在XML文件中声明事务哦~~~我是采用注解的方式  
@Transactional  
public class ExampleTests {  
  
    @Autowired  
    private WebApplicationContext wac;  
  
    private MockMvc mockMvc;  
  
    @Before  
    public void setup() {  
        // webAppContextSetup 注意上面的static import  
        // webAppContextSetup 构造的WEB容器可以添加fileter 但是不能添加listenCLASS  
        // WebApplicationContext context =  
        // ContextLoader.getCurrentWebApplicationContext();  
        // 如果控制器包含如上方法 则会报空指针  
        this.mockMvc = webAppContextSetup(this.wac).build();  
    }  
  
    @Test  
        //有些单元测试你不希望回滚  
        @Rollback(false)  
    public void ownerId() throws Exception {  
        mockMvc.perform((get("/spring/rest/4.do"))).andExpect(status().isOk())  
                .andDo(print());  
    }  
  
    @Test  
    public void test() throws Exception {  
        mockMvc.perform((get("/spring/test.do"))).andExpect(status().isOk())  
                .andDo(print())  
                .andExpect(model().attributeHasNoErrors("teacher"));  
    }  
  
    @Test  
    public void testb() throws Exception {  
        mockMvc.perform((get("/spring/testb.do"))).andExpect(status().isOk())  
                .andDo(print());  
    }  
  
    @Test  
    public void getAccount() throws Exception {  
        mockMvc.perform((post("/spring/post.do").param("abc", "def")))  
                .andExpect(status().isOk()).andDo(print());  
    }  
      /*测试将数据以JSON格式写入请求体发送的请求*/  
    @Test  
    public void testGetAll() throws IOException, Exception {  
        List<User> list=new ArrayList<User>();  
        list.add(new User(23,"你爱我"));  
        list.add(new User(25,"我不爱你"));  
        mockMvc.perform(post("/user/getUsers").contentType(MediaType.APPLICATION_JSON).content(JsonUtil.convertObjectToJsonBytes(list))).andExpect(status().isOk());  
    }  
    /*测试文件上传发送的请求*/  
    @Test  
    public void testUpload() throws Exception{  
        MockMultipartFile file = new MockMultipartFile("file", "orig.txt", null, "bar".getBytes());  
        mockMvc.perform(fileUpload("/user/upload").file(file)).andExpect(status().isOk());  
/**      //文件上传  
byte[] bytes = new byte[] {1, 2};  
mockMvc.perform(fileUpload("/user/{id}/icon", 1L).file("icon", bytes)) //执行文件上传  
        .andExpect(model().attribute("icon", bytes)) //验证属性相等性  
        .andExpect(view().name("success")); //验证视图  
*/
    }  
}  
  
}  

```
@TransactionConfiguration在spring 4.2已过时,只使用@Transactional就OK,也可以继承AbstractTransactionalJUnit4SpringContextTests

参考：http://www.ibm.com/developerworks/cn/java/j-lo-springunitest/
http://blog.jobbole.com/40740/
http://jinnianshilongnian.iteye.com/blog/2004660
http://287854442.iteye.com/blog/734322
http://blog.csdn.net/a95473004/article/details/8926929