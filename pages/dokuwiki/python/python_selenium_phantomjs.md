title: python_selenium_phantomjs 

#  Python_selenium_phantomjs动态抓取 
selenium:https://github.com/SeleniumHQ/selenium
当前版本3.0.1
A browser automation framework and ecosystem

phantomjs:http://phantomjs.org/ 
是一个服务器端的 JavaScript API 的 WebKit。也可以说是无界面浏览器。其支持各种Web标准： DOM 处理, CSS 选择器, JSON, Canvas, 和 SVG.

大部分的网页抓取用urllib都可以搞定，但是涉及到JavaScript及Ajax渲染的时候，urlopen就完全傻逼了，所以不得不用模拟浏览器，方法也有很多，此处采用的是selenium2+phantomjs
selenium2支持所有主流的浏览器和phantomjs这些无界面的浏览器。
安装：
```

pip install selenium

```
phantomjs不是python程序，去官网下载对应系统版本的安装即可。

```

from selenium import webdriver
import time

driver = webdriver.PhantomJS(executable_path=r'C:\Users\taojw\Desktop\pywork\phantomjs-2.1.1-windows\bin\phantomjs.exe')
driver.get("http://pythonscraping.com/pages/javascript/ajaxDemo.html")
time.sleep(3)
print(driver.find_element_by_id("content").text)
driver.close()

```
```

from selenium import webdriver

driver = webdriver.PhantomJS(executable_path='C:\Users\Gentlyguitar\Desktop\phantomjs-1.9.7-windows\phantomjs.exe')
driver.set_window_size(1120, 550)
driver.get("http://duckduckgo.com/")
driver.find_element_by_id('search_form_input_homepage').send_keys("Nirvana")
driver.find_element_by_id("search_button_homepage").click()
print(driver.current_url)
driver.close()

```
get方法会一直等到页面被完全加载，然后才会继续程序，但是对于ajax是无可奈何的。
send_keys就是填充input表单

##  等待页面渲染完成 
```

#等待页面渲染完成
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
 
dcap = dict(DesiredCapabilities.PHANTOMJS)
dcap["phantomjs.page.settings.userAgent"] = "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36"
driver = webdriver.PhantomJS(executable_path=r'C:\Users\taojw\Desktop\pywork\phantomjs-2.1.1-windows\bin\phantomjs.exe', desired_capabilities=dcap)
driver.get("http://pythonscraping.com/pages/javascript/ajaxDemo.html")
try:
    element = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.ID, "loadedButton")))
finally:
    print(driver.find_element_by_id("content").text)
    driver.close()

```
##  处理Javascript重定向 

```

#处理Javascript重定向
from selenium import webdriver
import time
from selenium.webdriver.remote.webelement import WebElement
from selenium.common.exceptions import StaleElementReferenceException

def waitForLoad(driver):
    elem = driver.find_element_by_tag_name("html")
    count = 0
    while True:
        count += 1
        if count > 20:
            print("Timing out after 10 seconds and returning")
            return
        time.sleep(.5)
        try:
            elem == driver.find_element_by_tag_name("html")
        #抛出StaleElementReferenceException异常说明elem元素已经消失了，也就说明页面已经跳转了。
        except StaleElementReferenceException:  
            return

driver = webdriver.PhantomJS(executable_path=r'C:\Users\taojw\Desktop\pywork\phantomjs-2.1.1-windows\bin\phantomjs.exe')
driver.get("http://pythonscraping.com/pages/javascript/redirectDemo1.html")
waitForLoad(driver)
print(driver.page_source)

```
##  设置PHANTOMJS的USER-AGENT 
有些网站的WebServer对User-Agent有限制，可能会拒绝不熟悉的User-Agent的访问。
设置PhantomJS的user-agent，是要设置“phantomjs.page.settings.userAgent”这个desired_capability.
```

from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
 
dcap = dict(DesiredCapabilities.PHANTOMJS)
dcap["phantomjs.page.settings.userAgent"] = "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36"

 
driver = webdriver.PhantomJS(executable_path='./phantomjs.exe', desired_capabilities=dcap)
driver.get("http://dianping.com/")
cap_dict = driver.desired_capabilities  #查看所有可用的desired_capabilities属性。
for key in cap_dict:
    print '%s: %s' % (key, cap_dict[key])
print driver.current_url
driver.quit()

```
##  示例：登陆知乎 
登陆知乎，然后能自动点击页面下方的“更多”，以载入更多的内容
```

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver import ActionChains
import time
import sys

driver = webdriver.PhantomJS(executable_path='C:\Users\Gentlyguitar\Desktop\phantomjs-1.9.7-windows\phantomjs.exe')
driver.get("http://www.zhihu.com/#signin")
#driver.find_element_by_name('email').send_keys('your email')
driver.find_element_by_xpath('//input[@name="password"]').send_keys('your password')
#driver.find_element_by_xpath('//input[@name="password"]').send_keys(Keys.RETURN)
time.sleep(2)
driver.get_screenshot_as_file('show.png')
#driver.find_element_by_xpath('//button[@class="sign-button"]').click()
driver.find_element_by_xpath('//form[@class="zu-side-login-box"]').submit()

try:
    #等待页面加载完毕
    dr=WebDriverWait(driver,5)
    dr.until(lambda the_driver:the_driver.find_element_by_xpath('//a[@class="zu-top-nav-userinfo "]').is_displayed())
except:
    print('登录失败')
    sys.exit(0)
driver.get_screenshot_as_file('show.png')
#user=driver.find_element_by_class_name('zu-top-nav-userinfo ')
#webdriver.ActionChains(driver).move_to_element(user).perform() #移动鼠标到我的用户名
loadmore=driver.find_element_by_xpath('//a[@id="zh-load-more"]')
actions = ActionChains(driver)
actions.move_to_element(loadmore)
actions.click(loadmore)
actions.perform()
time.sleep(2)
driver.get_screenshot_as_file('show.png')
print driver.current_url
print driver.page_source
driver.quit()

```
from selenium.webdriver.common.keys import Keys，keys这个类就是键盘上的键，文中的send_keys(Keys.RETURN)就是按一个回车
from selenium.webdriver.support.ui import WebDriverWait是为了后面一个等待的操作
from selenium.webdriver import ActionChains是导入一个动作的类
find_element推荐使用Xpath的方法，原因在于：优雅、通用、易学
Xpath表达式写法教程：http://www.ruanyifeng.com/blog/2009/07/xpath_path_expressions.html
值得注意的是，避免选择value带有空格的属性，譬如class = "country name"这种，不然会报错，大概compound class之类的错
检查用户密码是否输入正确的方法就是在填入后截屏看看

```

try:
    dr=WebDriverWait(driver,5)
    dr.until(lambda the_driver:the_driver.find_element_by_xpath('//a[@class="zu-top-nav-userinfo "]').is_displayed())
except:
    print '登录失败'
    sys.exit(0)

```
**是用来通过检查某个元素是否被加载来检查是否登录成功**，当个黑盒子用就可以了。**其中5的解释：5秒内每隔500毫秒扫描1次页面变化，直到指定的元素出现**
对于表单的提交，即可以选择登录按钮然后使用**click方法**，也可以选择表单然后使用**submit方法**，后者能应付没有登录按钮的情况，所以推荐使用submit()
对于一次点击，既可以使用click()，**也可以使用一连串的action来实现**。

**print driver.current_url
print driver.page_source**


参考：
http://www.cnblogs.com/chenqingyang/p/3772673.html
http://www.realpython.com/blog/python/headless-selenium-testing-with-python-and-phantomjs/#.U5FXUvmSziE
http://selenium-python.readthedocs.org/getting-started.html
http://www.cnblogs.com/paisen/p/3310067.html
http://smilejay.com/2013/12/set-user-agent-for-phantomjs/
更多参考：
[selenium webdriver的各种driver](http://blog.csdn.net/five3/article/details/19085303)