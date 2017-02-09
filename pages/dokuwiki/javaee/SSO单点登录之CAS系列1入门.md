createAt:2017-02-08 15:42:20
author:xbynet
modifyAt:2017-02-08 15:43:33
location:dokuwiki/javaee/SSO单点登录之CAS系列1入门
title:SSO单点登录之CAS系列1入门

CAS （ Central Authentication Service ） 是 Yale 大学发起的一个企业级的、开源的项目，旨在为 Web 应用系统提供一种可靠的单点登录解决方法（属于 Web SSO ）。
CAS 开始于 2001 年， 并在 2004 年 12 月正式成为 JA-SIG 的一个项目。
主要特性
1、   开源的、多协议的 SSO 解决方案； Protocols ： Custom Protocol 、 CAS 、 OAuth 、 OpenID 、 RESTful API 、 SAML1.1 、 SAML2.0 等。
2、   支持多种认证机制： Active Directory 、 JAAS 、 JDBC 、 LDAP 、 X.509 Certificates 等；
3、   安全策略：使用票据（ Ticket ）来实现支持的认证协议；
4、   支持授权：可以决定哪些服务可以请求和验证服务票据（ Service Ticket ）；
5、   提供高可用性：通过把认证过的状态数据存储在 TicketRegistry 组件中，这些组件有很多支持分布式环境的实现，如： BerkleyDB 、 Default 、 EhcacheTicketRegistry 、 JDBCTicketRegistry 、 JBOSS TreeCache 、 JpaTicketRegistry 、 MemcacheTicketRegistry 等；
6、   支持多种客户端： Java 、 .Net 、 PHP 、 Perl 、 Apache, uPortal 等。

# SSO 单点登录原理
单点登录（ Single Sign-On , 简称 SSO ）是目前比较流行的服务于企业业务整合的解决方案之一， SSO 使得在多个应用系统中，用户只需要 登录一次 就可以访问所有相互信任的应用系统。
SSO 体系中的角色
1、 User （多个）
2、 Web 应用（多个）
3、 SSO 认证中心（ 1 个 ）

SSO 实现模式的原则
1、   所有的认证登录都在 SSO 认证中心进行；
2、   SSO 认证中心通过一些方法来告诉 Web 应用当前访问用户究竟是不是已通过认证的用户；
3、   SSO 认证中心和所有的 Web 应用建立一种信任关系，也就是说 web 应用必须信任认证中心。（单点信任）

SSO 主要实现方式
SAML(Security Assertion Markup Language ，安全断言标记语言）的出现大大简化了 SSO ，并被 OASIS 批准为 SSO 的执行标准 。开源组织 OpenSAML 实现了 SAML 规范。
其余，略。

# CAS 的基本原理
3.1.  结构体系
从结构体系看， CAS 包括两部分： CAS Server 和 CAS Client 。
CAS Server 负责完成对用户的认证工作 , 需要独立部署 , CAS Server 会处理用户名 / 密码等凭证(Credentials) 。
CAS Client 负责处理对客户端受保护资源的访问请求，需要对请求方进行身份认证时，重定向到 CAS Server 进行认证。（原则上，客户端应用不再接受任何的用户名密码等 Credentials ）。
CAS Client 与受保护的客户端应用部署在一起，以 Filter 方式保护受保护的资源。

基础模式 SSO 访问流程主要有以下步骤：
1. 访问服务： SSO 客户端发送请求访问应用系统提供的服务资源。
2. 定向认证： SSO 客户端会重定向用户请求到 SSO 服务器。
3. 用户认证：用户身份认证。
4. 发放票据： SSO 服务器会产生一个随机的 Service Ticket 。
5. 验证票据： SSO 服务器验证票据 Service Ticket 的合法性，验证通过后，允许客户端访问服务。
6. 传输用户信息： SSO 服务器验证票据通过后，传输用户认证结果信息给客户端。
下面是 CAS 最基本的协议过程：
![8fa4e106-edd1-11e6-a476-00163e2eed34.png](/data/upload/8fa4e106-edd1-11e6-a476-00163e2eed34.png)

如上图： CAS Client 与受保护的客户端应用部署在一起，以 Filter 方式保护 Web 应用的受保护资源，过滤从客户端过来的每一个 Web 请求，同时， CAS Client 会分析 HTTP 请求中是否包含请求 Service Ticket( ST 上图中的 Ticket) ，如果没有，则说明该用户是没有经过认证的；于是 CAS Client 会重定向用户请求到 CAS Server （ Step 2 ），并传递 Service （要访问的目的资源地址）。 Step 3 是用户认证过程，如果用户提供了正确的 Credentials ， CAS Server 随机产生一个相当长度、唯一、不可伪造的 Service Ticket ，并缓存以待将来验证，并且重定向用户到 Service 所在地址（附带刚才产生的 Service Ticket ） , 并为客户端浏览器设置一个** Ticket Granted Cookie （ TGC ）** ； CAS Client 在拿到 Service 和新产生的 Ticket 过后，在 Step 5 和 Step6 中与 CAS Server 进行身份核实，以确保 Service Ticket 的合法性。
在该协议中，所有与 CAS Server 的交互均采用 SSL 协议，以确保 ST 和 TGC 的安全性。协议工作过程中会**有 2 次重定向** 的过程。但是 CAS Client 与 CAS Server 之间进行 Ticket 验证的过程对于用户是透明的（使用 HttpsURLConnection ）。

 CAS 请求认证时序图如下：
 ![be68abda-edd1-11e6-9b73-00163e2eed34.png](/data/upload/be68abda-edd1-11e6-9b73-00163e2eed34.png)
 
## CAS 代理模式
 略。
 
## 术语解释
CAS 系统中设计了 5 中票据： TGC 、 ST 、 PGT 、 PGTIOU 、 PT 。
Ø     Ticket-granting cookie(TGC) ：存放用户身份认证凭证的 cookie ，在浏览器和 CAS Server 间通讯时使用，并且只能基于安全通道传输（ Https ），是 CAS Server 用来明确用户身份的凭证；
Ø   Service ticket(ST) ：服务票据，服务的惟一标识码 , 由 CAS Server 发出（ Http 传送），通过客户端浏览器到达业务服务器端；一个特定的服务只能有一个惟一的 ST ；
Ø   Proxy-Granting ticket （ PGT ）：由 CAS Server 颁发给拥有 ST 凭证的服务， PGT 绑定一个用户的特定服务，使其拥有向 CAS Server 申请，获得 PT 的能力；
Ø   Proxy-Granting Ticket I Owe You （ PGTIOU ） : 作用是将通过凭证校验时的应答信息由 CAS Server 返回给 CAS Client ，同时，与该 PGTIOU 对应的 PGT 将通过回调链接传给 Web 应用。 Web 应用负责维护 PGTIOU 与 PGT 之间映射关系的内容表；
Ø   Proxy Ticket (PT) ：是应用程序代理用户身份对目标程序进行访问的凭证；

其它说明如下：
Ø   Ticket Granting ticket(TGT) ：票据授权票据，由 KDC 的 AS 发放。即获取这样一张票据后，以后申请各种其他服务票据 (ST) 便不必再向 KDC 提交身份认证信息 (Credentials) ；
Ø   Authentication service(AS) --------- 认证用服务，索取 Credentials ，发放 TGT ；
Ø   Ticket-granting service (TGS) --------- 票据授权服务，索取 TGT ，发放 ST ；
Ø   KDC( Key Distribution Center ) ---------- 密钥发放中心；

## CAS 安全性
CAS 的安全性仅仅依赖于 SSL 。使用的是 secure cookie 。
4.1.  TGC/PGT 安全性
对于一个 CAS 用户来说，最重要是要保护它的 TGC ，如果 TGC 不慎被 CAS Server 以外的实体获得， Hacker 能够找到该 TGC ，然后冒充 CAS 用户访问 所有 授权资源。 PGT 的角色跟 TGC 是一样的。
从基础模式可以看出， TGC 是 CAS Server **通过 SSL 方式**发送给终端用户，因此，要截取 TGC 难度非常大，从而确保 CAS 的安全性。
TGT 的存活周期默认为 120 分钟。

4.2.  ST/PT 安全性
ST （ Service Ticket ）是通过 Http 传送的，因此网络中的其他人可以 Sniffer 到其他人的 Ticket 。 CAS 通过以下几方面来使 ST 变得更加安全（事实上都是可以配置的）：
1、  ** ST 只能使用一次**
CAS 协议规定，无论 Service Ticket 验证是否成功， CAS Server 都会清除服务端缓存中的该Ticket ，从而可以确保一个 Service Ticket 不被使用两次。
2、   ST 在一段时间内失效
CAS 规定 ST 只能存活一定的时间，然后 CAS Server 会让它失效。默认有效时间为 5 分钟。
3、   ST 是基于随机数生成的
ST 必须足够随机，如果 ST 生成规则被猜出， Hacker 就等于绕过 CAS 认证，直接访问 对应的服务。


参考：http://www.cnblogs.com/gxbk629/p/4473569.html