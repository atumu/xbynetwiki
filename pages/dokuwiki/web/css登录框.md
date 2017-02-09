title: css登录框 

#  登录框居中背景玻璃透明效果 

```

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
  top: 90px;
  left: 0;
  right: 0;
  text-align: center;
}
/* responsive */
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
/* responsive */
@media (max-width: 767px) {
  .login-wrapper .box {
    width: 350px;
  }
}
@media (max-width: 480px) {
  .login-wrapper .box {
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

```
```

<div class="login-wrapper">
	<div class="logo">&nbsp;</div>
    <div class="box">
        <div class="content-wrap">
            <h6>发布应用系统</h6>
            <form id="loginForm" action="">
            <div class="form-group">
            	 <input class="form-control" name="username" type="text" placeholder="登录名">
            </div>
           	<div class="form-group">
            	<input class="form-control" name="password" type="password" placeholder="密码">
            </div>
            <div class="row">
           	<div class="col-sm-7">
           		<div class="form-group">
           			<input id="kaptcha" name="code" class="form-control" type="text" placeholder="验证码">
           		</div>
           	</div>
           	<div class="col-sm-5">
           		<img src="/wyjg/captcha/image" id="kaptchaImage" align="left"/> 
           	</div>
           	</div>
            <div class="remember">
                <input id="remember-me" type="checkbox">
                <label for="remember-me">记住我</label>
            </div>
            <button class="btn-glow primary login" type="button" onclick="login()">登  录</button>
            </form>
        </div>
    </div>
</div>


```