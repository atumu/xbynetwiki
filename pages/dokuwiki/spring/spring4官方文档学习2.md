title: spring4官方文档学习2 

#  Spring4官方文档学习2Spring4新特性 
**4.0新特性:**
  * 全面支持Java8新特性。最低支持Java6
  * 支持Servlet3.x
  * 新增@RestController注解，而不用在每个rest方法前面都加@ResponseBody了。
  * AsyncRestTemplate用于支持异步非阻塞rest客户端开发
  * 新增spring-websocket模块，提供websocket支持。
  * 新增spring-messaging模块，提供adds support for STOMP as the WebSocket sub-protocol.and processing STOMP messages from WebSocket clients. As a result an @Controller can now contain both @RequestMapping and @MessageMapping methods for handling HTTP requests and messages from WebSocket-connected clients.
  * 测试模块强化。
**4.1新特性**：
  * JMS支持强化
  * 缓存支持强化：1、Caches can be resolved at runtime using a CacheResolver. As a result the value argument defining the cache name(s) to use is no longer mandatory.More operation-level customizations: cache resolver, cache manager, key generator2、一个新的类级别的注解@CacheConfig允许定义通用的配置。3、通过使用` CacheErrorHandler `得到缓存方法更好的异常处理。4、CacheInterface 新增 putIfAbsent方法
  * 新的HttpMessageConverter配置选项:1、Gson支持，2、Google Protocol Buffers 支持，

**4.2新特性**：
全面支持Hibernate5.org.springframework.orm.hibernate5
测试增强:
1、SpringJUnit4ClassRunner.
2、新增@Commit注解用来替换@Rollback(false).
3、@Rollback支持类及别
