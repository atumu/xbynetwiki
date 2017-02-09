title: apache_tiles使用 

#  apache tiles使用 
参考：http://my.oschina.net/jast90/blog/283254  http://aiilive.blog.51cto.com/1925756/1596059系列文章

 Tiles项目：http://tiles.apache.org/index.html

本文使用版本` Tiles 3.x `版本。
Apache Tiles 是什么？
一个免费开源的模板框架，为现代的Java应用程序。 
基于复合模式，它是建立在简化用户界面的开发。 
对于复杂的网站，但它仍然一起工作，任何的MVC技术的最简单，最优雅的方式。

复合式视图模式（The Composite View Pattern）
所有的网站都有一个共同点：它们是由具有类似结构的网页。页面共享相同的布局，而每个页面是由不同的独立的配件，但是始终摆在所有网站相同的位置。 
 复合视图模式正式化了这个典型的使用，它允许创建具有类似结构的页面，其中页面的每个部分在不同情况而有所不同的页面。

复合式视图模式怎么工作？
 为了理解这个模式，让我举个例子。从下图你可以看到一个典型的web网页结构。
![](/data/dokuwiki/javaweb/pasted/20150914-073211.png)
这个结构称为“典型布局（classic layout）”。这个模板按照这个布局组织页面，在需要的位置放上每一块，所以header部分在上，footer部分在下，等等。 将会发生如下情况，例如单击一个超链接跳转到另一个页面，另一个页面只需要改变其中的一部分，通常是body部分。     
![](/data/dokuwiki/javaweb/pasted/20150914-073247.png)
你能看到，这个页面虽然是不同的，但是他们的不同之处仅仅只有body部分。注意，虽然页面是不同的，但并不像在frameset中刷新frame。

**复合视图VS装饰器（decorator）**
**Tiles 是一个复合视图框架**：它允许在整个应用中复用页面。通过装饰器模式也可以实现复用页面。例如：**Sitemesh 就是一个基于装饰器模式的。**
通过创建一个模板（template）来组织各个页面到同一个页面，装饰器模式需要一个简单的HTML页面，在转换时添加缺失的部分（在我们的以上例子中，添加header，footer和menu） 最终呈现它。 
下面是两种模式的对比表
![](/data/dokuwiki/javaweb/pasted/20150914-073352.png)

##  安装 
```

  <dependency>
    <groupId>org.apache.tiles</groupId>
    <artifactId>tiles-extras</artifactId>
    <version>3.0.5</version>
  </dependency>

```
##  启动Tiles引擎 
通过在**web.xml**文件中配置适当的listener来加载tiles 容器，既然我们决定加载一切，我们将使用`  CompleteAutoTilesListener `:
```

<listener>
    <listener-class>org.apache.tiles.extras.complete.CompleteAutoloadTilesListener</listener-class>
</listener>

```

##  模板：Template 
在Tiles中，模板（Template）是一个页面的布局部分。你能将一个页面结构看成是由不同的需要填补空白组成。
![](/data/dokuwiki/javaweb/pasted/20150914-073829.png)   
你能够将该结构复制到一个新建的JSP页面中，如下所示 template.jsp
```

<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles" %>
<table>
  <tr>
    <td colspan="2">
      <tiles:insertAttribute name="header" />
    </td>
  </tr>
  <tr>
    <td>
      <tiles:insertAttribute name="menu" />
    </td>
    <td>
      <tiles:insertAttribute name="body" />
    </td>
  </tr>
  <tr>
    <td colspan="2">
      <tiles:insertAttribute name="footer" />
    </td>
  </tr>
</table>

```
**属性：Attribute**
属性是模板中的空白，它在你的应用程序中被填充到模板中。属性可以是以下三种类型：
  * string:属性是string的话，会将string直接呈现在页面。
  * template:属性是一个模板（Template），有无属性都行。如果有属性的话，你也要将他们填充后再呈现页面。
  * definition:它是一个可重复使用组成的页面，包含所有的属性来填充以呈现页面。 
**定义：definition**
定义是呈现给最终用户的组合物；本质上，一个定义是由一个模板和完全或部分填充的属性组成的。说白了就是：一个定义是由一个模板和属性组成的。

**如果所有的“属性”都填充了，它将可以呈现给最终用户。**
如果不是所有的属性都填充了，这个定义称为“抽象定义”（abastract definition）,它可以被用作“父定义”，让其他“定义”**继承**，失去的“属性”能在运行时填充。
例如，你可以按之前看过的“典型模板”创建创建一个页面，修改Tiles的配置文件，如下：
```

<definition name="myapp.homepage" template="/layouts/classic.jsp">
  <put-attribute name="header" value="/tiles/banner.jsp" />
  <put-attribute name="menu" value="/tiles/common_menu.jsp" />
  <put-attribute name="body" value="/tiles/home_body.jsp" />
  <put-attribute name="footer" value="/tiles/credits.jsp" />
</definition>

```

##  "模版定义"继承 
```

<tiles-definitions>
<!--<start id="tile_template"/>-->
   <definition name="template"                                 <!-- 定义名-->
               template="/WEB-INF/views/main_template.jsp">  <!-- template定义模版位置-->
     <put-attribute name="top" 
                    value="/WEB-INF/views/tiles/spittleForm.jsp" />
     <put-attribute name="side" 
                    value="/WEB-INF/views/tiles/signinsignup.jsp" />
   </definition>
<!--<end id="tile_template"/>-->
 
<!--<start id="tile_home"/>-->
   <definition name="home" extends="template">                <!--extends继承 -->
     <put-attribute name="content" value="/WEB-INF/views/home.jsp" />
   </definition>  

   <definition name="login" extends="template">
     <put-attribute name="content" value="/WEB-INF/views/login.jsp" />
     <put-attribute name="side" value="/WEB-INF/views/tiles/alreadyamember.jsp" />
   </definition>  
  </tiles-definitions>

```
**视图助手：View Preparer**
有时候一个定义在呈现之前需要“预处理”。例如，显示一个menu时，menu的结构必须被创建并且已经保存在request范围内。
为了达到“预处理 ”，视图助手将会被用到，视图助手将在呈现定义之前被调用，因此在将“定义”呈现所需的东西都会被正确的“预处理 ”。

##  创建和使用Tiles 页面 
```

<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles" %>
<html>
  <head>
    <title><tiles:getAsString name="title"/></title>
  </head>
  <body>
        <table>
      <tr>
        <td colspan="2">
          <tiles:insertAttribute name="header" />
        </td>
      </tr>
      <tr>
        <td>
          <tiles:insertAttribute name="menu" />
        </td>
        <td>
          <tiles:insertAttribute name="body" />
        </td>
      </tr>
      <tr>
        <td colspan="2">
          <tiles:insertAttribute name="footer" />
        </td>
      </tr>
    </table>
  </body>
</html>

```
在这个模板中有5个属性：title（string类型的属性），header，menu,body和footer。

创建一个定义
**默认情况，“定义”文件是/WEB-INF/tiles.xml。**如果你使用的是CompleteAutoloadTilesListener,tiles将会使用webapp目录下按/WEB-INF/tiles*.xml匹配或classpath下按/META-INF/tiles*.xml匹配的任何文件作为“定义 ”文件；如果发现多个，tiles将会合并这些文件到一起。
但现在，我们使用默认情况并创建` /WEN-INF/tiles.xml `文件，该文件下包含一个“定义”。
```

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">
<tiles-definitions>
  <definition name="myapp.homepage" template="/layouts/classic.jsp">
    <put-attribute name="title" value="Tiles tutorial homepage" />
    <put-attribute name="header" value="/tiles/banner.jsp" />
    <put-attribute name="menu" value="/tiles/common_menu.jsp" />
    <put-attribute name="body" value="/tiles/home_body.jsp" />
    <put-attribute name="footer" value="/tiles/credits.jsp" />
  </definition>
</tiles-definitions>

```
**渲染定义**
创建完定以后，你就能渲染它了。
下面是几种方式：
1、通过使用` <tiles:insertDefinition /> `标签，将定义插入一个JSP页面。
```

<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles" %>
<tiles:insertDefinition name="myapp.homepage" />

```
2、 其他情况，你可以直接使用response来渲染定义，通过使用Tiles 容器
```

TilesContainer container = TilesAccess.getContainer(
        request.getSession().getServletContext());
container.render("myapp.homepage", request, response);

```
3、通过使用Tiles提供的Rendering Utilities。例如，如果你已经配置了TilesDispatchServlet，你能通过请求：http://example.com/webapp/myapp.homepage.tiles来渲染“定义”。
4、通过使用支持的框架（struts,spring等）来渲染“定义”。 


**总结一下使用步骤：**
  * 添加Maven依赖
  * web.xml配置加入tiles listener配置
  * 在/WEB-INF/目录下创建一个tiles.xml文件，添加配置：
  * 创建相关目录以及模版
  * 创建属性(attribute)，一般为JSP页面
  * tiles.xml中添加定义(difinition)
  * 插入定义到jsp页面

##  SpringMVC整合Tiles 
```

<dependency>
    <groupId>org.apache.tiles</groupId>
    <artifactId>tiles-extras</artifactId>
    <version>3.0.5</version>
  </dependency>

```

```

<!-- Tiles3配置-->
 <bean class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
     <property name="definitions">
       <list>
         <value>/WEB-INF/views/**/views.xml</value> <!-- Ant风格路径-->
       </list>
     </property>
   </bean>
     <!-- Tiles视图解析器--> 
<bean class="org.springframework.web.servlet.view.tiles3.TilesViewResolver"/>

```
非常简单仅此而已。

##  乱码问题解决与实战 
使用tiles结合的` **每一个** `jsp包括template.jsp都需要在头部声明
```

<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>

```
例如：
views.xml:
```

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE tiles-definitions PUBLIC
       "-//Apache Software Foundation//DTD Tiles Configuration 3.0//EN"
       "http://tiles.apache.org/dtds/tiles-config_3_0.dtd">
<tiles-definitions>
<definition name="admin.index" template="/WEB-INF/views/admin/template.jsp">
  <put-attribute name="header" value="/WEB-INF/views/admin/header.jsp" />
  <put-attribute name="menu" value="/WEB-INF/views/admin/sider.jsp" />
  <put-attribute name="body" value="/WEB-INF/views/admin/index.jsp" />
  <put-attribute name="footer" value="/WEB-INF/views/admin/footer.jsp" />
</definition>
</tiles-definitions>

```

template.jsp:头部声明编码
```

<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://tiles.apache.org/tags-tiles" prefix="tiles" %>

      <tiles:insertAttribute name="header" />
      <tiles:insertAttribute name="menu" />
      <tiles:insertAttribute name="body" />
      <tiles:insertAttribute name="footer" />


```
header.jsp:头部声明编码
```

<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta http-equiv="content-type" content="text/html;charset=utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>小小懒羊羊管理平台</title>
  <!-- Tell the browser to be responsive to screen width -->
  <meta content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" name="viewport">
  <!-- Bootstrap 3.3.5 -->
  <link href="static/modules/AdminLTE/dist/css/bootstrap.min.css" rel="stylesheet">
  <!-- Font Awesome -->
  <link rel="stylesheet" href="static/modules/AdminLTE/dist/css/font-awesome.min.css">
  <!-- Ionicons -->
  <link rel="stylesheet" href="static/modules/AdminLTE/dist/css/ionicons.min.css">
  <!-- jvectormap -->
  <!-- Theme style -->
  <link rel="stylesheet" href="static/modules/AdminLTE/dist/css/AdminLTE.min.css">
  <!-- AdminLTE Skins. Choose a skin from the css/skins
       folder instead of downloading all of them to reduce the load. -->
  <link rel="stylesheet" href="static/modules/AdminLTE/dist/css/skins/_all-skins.min.css">
</head>
<body class="hold-transition skin-blue sidebar-mini">
<div class="wrapper">
<header class="main-header">

    <!-- Logo -->
    <a href="admin/index" class="logo">
      <!-- logo for regular state and mobile devices -->
      <span ><b>小小懒羊羊管理平台</b></span>
    </a>

    <!-- Header Navbar: style can be found in header.less -->
    <nav class="navbar navbar-static-top" role="navigation">
      <!-- Sidebar toggle button-->
      <a href="#" class="sidebar-toggle" data-toggle="offcanvas" role="button">
        <span class="sr-only">Toggle navigation</span>
      </a>
      <!-- Navbar Right Menu -->
      <div class="navbar-custom-menu">
        <ul class="nav navbar-nav">
          <!-- Messages: style can be found in dropdown.less-->
          <li class="dropdown messages-menu">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown">
              <i class="fa fa-envelope-o"></i>
              <span class="label label-success">4</span>
            </a>
            <ul class="dropdown-menu">
              <li class="header">You have 4 messages</li>
              <li>
                <!-- inner menu: contains the actual data -->
                <ul class="menu">
                  <li><!-- start message -->
                    <a href="#">
                      <div class="pull-left">
                        <img src="static/modules/AdminLTE/dist/img/huba.jpg" class="img-circle" alt="User Image">
                      </div>
                      <h4>
                        Support Team
                        <small><i class="fa fa-clock-o"></i> 5 mins</small>
                      </h4>
                      <p>Why not buy a new awesome theme?</p>
                    </a>
                  </li>
                  <!-- end message -->
                </ul>
              </li>
              <li class="footer"><a href="#">See All Messages</a></li>
            </ul>
          </li>
          <!-- Notifications: style can be found in dropdown.less -->
          <li class="dropdown notifications-menu">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown">
              <i class="fa fa-bell-o"></i>
              <span class="label label-warning">10</span>
            </a>
            <ul class="dropdown-menu">
              <li class="header">You have 10 notifications</li>
              <li>
                <!-- inner menu: contains the actual data -->
                <ul class="menu">
                  <li>
                    <a href="#">
                      <i class="fa fa-users text-aqua"></i> 5 new members joined today
                    </a>
                  </li>
                 
                </ul>
              </li>
              <li class="footer"><a href="#">View all</a></li>
            </ul>
          </li>
          <!-- Tasks: style can be found in dropdown.less -->
          <li class="dropdown tasks-menu">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown">
              <i class="fa fa-flag-o"></i>
              <span class="label label-danger">9</span>
            </a>
            <ul class="dropdown-menu">
              <li class="header">You have 9 tasks</li>
              <li>
                <!-- inner menu: contains the actual data -->
                <ul class="menu">
                  <li><!-- Task item -->
                    <a href="#">
                      <h3>
                        Design some buttons
                        <small class="pull-right">20%</small>
                      </h3>
                      <div class="progress xs">
                        <div class="progress-bar progress-bar-aqua" style="width: 20%" role="progressbar" aria-valuenow="20" aria-valuemin="0" aria-valuemax="100">
                          <span class="sr-only">20% Complete</span>
                        </div>
                      </div>
                    </a>
                  </li>
                  <!-- end task item -->
  
                </ul>
              </li>
              <li class="footer">
                <a href="#">View all tasks</a>
              </li>
            </ul>
          </li>
          <!-- User Account: style can be found in dropdown.less -->
          <li class="dropdown user user-menu">
            <a href="#" class="dropdown-toggle" data-toggle="dropdown">
              <img src="static/modules/AdminLTE/dist/img/huba.jpg" class="user-image" alt="User Image">
              <span class="hidden-xs">Admin</span>
            </a>
            <ul class="dropdown-menu">
              <!-- User image -->
              <li class="user-header">
                <img src="static/modules/AdminLTE/dist/img/huba.jpg" class="img-circle" alt="User Image">

                <p>
                 Admin
                  <small> 2015</small>
                </p>
              </li>
              <!-- Menu Body -->
              <li class="user-body">
                <div class="row">
                  <div class="col-xs-4 text-center">
                    <a href="#">Followers</a>
                  </div>
                  <div class="col-xs-4 text-center">
                    <a href="#">Sales</a>
                  </div>
                  <div class="col-xs-4 text-center">
                    <a href="#">Friends</a>
                  </div>
                </div>
                <!-- /.row -->
              </li>
              <!-- Menu Footer-->
              <li class="user-footer">
                <div class="pull-left">
                  <a href="#" class="btn btn-default btn-flat">配置</a>
                </div>
                <div class="pull-right">
                  <a href="#" class="btn btn-default btn-flat">登出</a>
                </div>
              </li>
            </ul>
          </li>
          <!-- Control Sidebar Toggle Button -->
          <li>
            <a href="#" data-toggle="control-sidebar"><i class="fa fa-gears"></i></a>
          </li>
        </ul>
      </div>

    </nav>
  </header>

```
footer.jsp:头部声明编码
```

 <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
 <!-- /.content-wrapper -->

  <footer class="main-footer">
    <div class="pull-right hidden-xs">
      <b>Version</b> 1.0
    </div>
    <strong>Copyright &copy; 2014-2015 <a href="http://wiki.xby1993.net">小小懒羊羊管理平台</a>.</strong> All rights
    reserved.
  </footer>

</div>

<!-- jQuery 2.1.4 -->
<script src="static/modules/jquery/jquery.min.js"></script>
<!-- Bootstrap 3.3.5 -->
<script src="static/modules/bootstrap/js/bootstrap.min.js"></script>
<!-- FastClick -->
<script src="static/modules/fastclick/fastclick.js"></script>
<!-- AdminLTE App -->
<script src="static/modules/AdminLTE/dist/js/app.min.js"></script>

<!-- SlimScroll 1.3.0 -->
<script src="static/modules/slimScroll/jquery.slimscroll.min.js"></script>
<!-- AdminLTE dashboard demo (This is only for demo purposes) -->
</body>
</html>

```