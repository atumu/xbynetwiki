title: web登录 

##  Web登录 
##  login.jsp 
主要注意:验证码异步校验。bootstrap按钮btn-block。 css样式与@Media. 
```

<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<!DOCTYPE html>
<html>
<head>
<c:set var="ctx" value="${pageContext.request.contextPath}" />
<title>登录页面</title>
<meta http-equiv="X-UA-Compatible" content="IE=edge" />
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<link rel="stylesheet" type="text/css"
	href="static/modules/bootstrap/css/bootstrap.min.css" />
<link rel="stylesheet" type="text/css"
	href="static/modules/bootstrap/css/bootstrap-theme.min.css" />
<link rel="stylesheet" type="text/css"
	href="static/modules/easyui/themes/bootstrap/easyui.css">
<!– 使用bootstrap主题 –>
<link rel="stylesheet" type="text/css"
	href="static/modules/easyui/themes/icon.css">
<!– 图标样式–>
<script src="static/modules/jquery/jquery.min.js"></script>
<script src="static/modules/bootstrap/js/bootstrap.min.js"></script>
<script src="static/modules/easyui/jquery.easyui.min.js"></script>
<!–easyui主js–>
<script src="static/modules/easyui/locale/easyui-lang-zh_CN.js"></script>
<!–使用中文语言包–>

<script type="text/javascript" src="static/modules/jquery/plugins/validate/jquery.validate.min.js"></script>
	<script type="text/javascript" src="static/modules/jquery/plugins/validate/additional-methods.min.js"></script>
	<script type="text/javascript" src="static/modules/jquery/plugins/validate/localization/messages_zh.min.js"></script>
<script>
	function getContext() {
		return '';
	}
	
</script>

<style>

body {
	MARGIN: 0px;
	background-image: url();
	background: rgba(0,102,153,0.5);
}
.error{
	 font-size:14px; 
	 font-weight:bold;
	 color:red;
}
input{
	border:0;
	font-size:18px;	
	}
.login-wrapper {
  position: absolute;
  top: 20px;
  left: 0;
  right: 0;
  text-align: center;
}
/* responsive #99C7EF */
@media (max-height: 900px) and (min-device-height:900px) {
  .login-wrapper {
    top: 150px;
  }
}
@media (max-device-height: 900px) {
  .login-wrapper {
    top: 5px;
  }
}
.login-wrapper .box {
    margin: 0 auto;
    padding: 35px 0 30px;
    float: none;
    width: 400px;
    box-shadow: 0 0 6px 2px rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    background: rgba(255, 255, 255, 0.65);
}
.login-wrapper .box h6 {
  text-transform: uppercase;
  margin: 0 0 30px 0;
  font-size: 18px;
  font-weight: 600;
}
/* responsive */
@media (max-width: 767px) {
  .login-wrapper .box {
    width: 350px;
  }
}
@media (max-width: 480px) {
  .login-wrapper  {
    width: 90%;
  }
}    

.login-wrapper .box .content-wrap {
    width: 82%;
    margin: 0 auto;
}
form {
    display: block;
    margin-top: 0em;
}

</style>
</head>
<body>

<div class="login-wrapper">
	<div class="logo">&nbsp;</div>
    <div class="box">
				<div class="content-wrap">

					<h6>小小懒羊羊备份管理工具</h6>

					<!--<c:out value=''/>login-->
					<form method="POST" action="login" id="loginform">
					<input type="hidden" name="isLoginForm" id="isLoginForm" value="true"/> 
					<input type="hidden" id="loginId"
							name="loginId" value="<c:out value='${sessionScope.loginId}'/>" />
						<div class="form-group">
							<span class="error">${error}</span>
						</div>

						<div class="form-group">
							
			
								<input type="text" class="form-control"
									id="name" name="name" placeholder="用户名"
									value="${user.name }" >
						
						</div>
						<div class="form-group">
						
								<input type="password" class="form-control "
									name="password" id="password" placeholder="密码 "
									>
					
						</div>
						 <div class="row">
							<div class="col-md-7 col-sm-7">
								<input name="captchaCode" class="form-control" type="text"
									id="captchaCode" maxlength="4" placeholder="验证码" />
							</div>
							<div class="col-md-5 col-sm-5">
								<img src="captcha" id="kaptchaImage"  />
							</div>
						</div>
						<div style="margin-top: 10px"> 
						<button type="submit" class="btn btn-primary btn-block" >登录</button>
						</div>
					</form>
				</div>
			</div>
		</div>


	<script>
		$(function() {

			$("#loginform").validate({
				rules:{
					captchaCode:{
		               required:true,
		               remote:{ 
		                  type:"POST",  
		                  url: "captcha/valid", 
		                  data:{  
		                	  captchaCode:function(){return $("#captchaCode").val();}  
		                  }  
		               }
		       	    },
		       	 name:{
			       		required:true
			       	},
			       	password:{
			       		required:true
			       	}
				},
			 	messages: {
			 		captchaCode:{
			     		remote:"验证码错误"
			     	}
			 	}
			});
		$("#loginform").submit(function() {
			var isValid = $("#loginform").valid();

			return isValid;
		}); 
			$('#kaptchaImage').click(
					function() {//生成验证码  
						$(this).hide().attr('src',
								'captcha?' + Math.floor(Math.random() * 100))
								.fadeIn();
					})
			$("input").each(function(index){
				$(this).blur(function(){
					$(".error").text("");
				});
			});

		});
	</script>
</body>
</html>

```
##  CaptchaImageController.java 
```

@Controller
public class CaptchaImageController {
	private Producer captchaProducer = null;  
	@Autowired  
    public void setCaptchaProducer(Producer captchaProducer) {  
        this.captchaProducer = captchaProducer;  
    }  
  
    @RequestMapping("/captcha")  
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {  
  
        response.setDateHeader("Expires", 0);  
        // Set standard HTTP/1.1 no-cache headers.  
        response.setHeader("Cache-Control", "no-store, no-cache, must-revalidate");  
        // Set IE extended HTTP/1.1 no-cache headers (use addHeader).  
        response.addHeader("Cache-Control", "post-check=0, pre-check=0");  
        // Set standard HTTP/1.0 no-cache header.  
        response.setHeader("Pragma", "no-cache");  
        // return a jpeg  
        response.setContentType("image/jpeg");  
        // create the text for the image  
        String capText = captchaProducer.createText();  
        // store the text in the session  
        request.getSession().setAttribute(Constants.KAPTCHA_SESSION_KEY, capText);  
        // create the image with the text  
        BufferedImage bi = captchaProducer.createImage(capText);  
        ServletOutputStream out = response.getOutputStream();  
        // write the data out  
        ImageIO.write(bi, "jpg", out);  
        try {  
            out.flush();  
        } finally {  
            out.close();  
        }  
        return null;  
    }  
    @RequestMapping("/captcha/valid")
    @ResponseBody
    public String validCaptcha(HttpServletRequest request, HttpServletResponse response,@RequestParam("captchaCode")String captchaCode){
    	HttpSession session = request.getSession();
    	String code = (String)session.getAttribute(com.google.code.kaptcha.Constants.KAPTCHA_SESSION_KEY);
		if(StringUtils.checkEquals(code, captchaCode)){
			return "true";
		}
    	return "false";
    }
}

```
##  LoginController.java 
```

@Controller
public class LoginController {
	@Autowired
	private BaseDAO dao;
//	@Autowired
//	private BaseService service;

	@RequestMapping("/login")
	public String login(HttpServletRequest request,HttpServletResponse resp, @ModelAttribute("user") UserDto uDto) {

		HttpSession session = request.getSession();
		// 获取表单隐藏域
//		String loginId = request.getParameter("loginId");
		
		// 如果早就登录了，那么直接跳转
		if (session.getAttribute(GeneUtil.SESSION_USERTYPE_KEY) != null) {
//			return "home";
//			return "redirect:home";
			ControllerUtil.redirectURL(resp, "/home");
		
			return null;
		}
		//如果没有这个字段，说明不是提交表单，而是申请显示表单。
		String isLoginForm =request.getParameter("isLoginForm");
		if(!StringUtils.checkString(isLoginForm)){
			return "login";
		}
//		// 如果表单非重复提交提交，则移除
//		if (StringUtils.checkString(loginId) && loginId.equals(session.getAttribute("loginId"))) {
//			session.removeAttribute("loginId");
//		} else {
//			session.setAttribute("loginId", UUID.randomUUID().toString());
//			request.setAttribute("error", "表单重复提交");
//			return "login";
//		}
//		if (!StringUtils.checkStringArray(new String[] { uDto.getName(), uDto.getPassword() })) {
//			request.setAttribute("error", "用户名或密码不能为空");
//			session.setAttribute("loginId", UUID.randomUUID().toString());
//			return "login";
//		}

		User u = new User();

		// int isFirst = (int) em.createNativeQuery("select is_first from
		// init").getSingleResult();

		Map map1 = dao.findOneBySql("select is_first from init", null);
		if (map1 != null) {
			int isFirst = (int) (map1.get("isFirst"));
			// 如果是第一次使用系统，则输入为管理员帐号，并保存。
			if (isFirst == 0) {
				uDto.setUserType(UserType.ADMIN.name());
				final User u1 = new User();
				Dto2Entity.transalte(uDto, u1);
				dao.executeMultiInTransaction(new ExeInTransactionWrapper() {
					@Override
					public void execute() {
						// TODO Auto-generated method stub
						dao.saveOrUpdate(u1);
						dao.updateBySql("update init set is_first=1", null);
					}
				});
				
				session.setAttribute(GeneUtil.SESSION_USERTYPE_KEY, "ADMIN");
				ControllerUtil.redirectURL(resp, "/home");
				return null;
			} else {

				// Query q = em.createQuery("select u from user u where
				// u.name=:name").setParameter("name",
				// uDto.getName());
				// u=(User)q.getSingleResult();
				Map<String, Object> map = new HashMap<>();
				map.put("name", uDto.getName());
				u = dao.findOneByJpql(User.class, "select u from user u where u.name=:name", map);
				if (u != null) {
					if (u.getPassword().equals(uDto.getPassword()) && u.getUserType().equals(UserType.ADMIN.name())) {

						session.setAttribute(GeneUtil.SESSION_USERTYPE_KEY, "ADMIN");
						ControllerUtil.redirectURL(resp, "/home");
						return null;
					}
				}

			}
		}

		session.setAttribute("loginId", UUID.randomUUID().toString());
		request.setAttribute("error", "用户名或密码错误");
		return "login";

	}

	@RequestMapping("/loginOut")
	public void loginOut(HttpServletRequest request,HttpServletResponse resp) {
		HttpSession session = request.getSession();
		session.removeAttribute(GeneUtil.SESSION_USERTYPE_KEY);
		ControllerUtil.redirectURL(resp, "/login");
	}

	@RequestMapping("/home")
	public ModelAndView success(HttpServletRequest request) {
		ModelAndView mv=new ModelAndView();
		HttpSession session = request.getSession();
		Object key = session.getAttribute(GeneUtil.SESSION_USERTYPE_KEY);
		if (key != null) {
//			return "success";
			mv.setViewName("success");
			
		}else{
			session.setAttribute("loginId", UUID.randomUUID().toString());
			mv.setViewName("login");
		}
		
		return mv;
	}
}

```
```

public class ControllerUtil {
	public static final void redirectURL(HttpServletResponse resp,String target){
		resp.setStatus(302);
		resp.setHeader("Location", FileUtil.getPropertyFromCommon("host")+target);
	}
}

```