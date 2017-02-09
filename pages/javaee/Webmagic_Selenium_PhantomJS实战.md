createAt:2017-01-22 16:27:48
author:xbynet
modifyAt:2017-01-24 15:09:59
location:javaee/Webmagic_Selenium_PhantomJS实战
title:Webmagic_Selenium_PhantomJS实战

还是直接贴代码说明比较实在。
感觉webmagic-selenium这个模块有点鸡肋，但还是有可借鉴之处。借鉴它写了一个SeleniumDownloader,如下：

```
import org.openqa.selenium.By;
import org.openqa.selenium.Cookie;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import us.codecraft.webmagic.Page;
import us.codecraft.webmagic.Request;
import us.codecraft.webmagic.Site;
import us.codecraft.webmagic.Task;
import us.codecraft.webmagic.downloader.Downloader;
import us.codecraft.webmagic.selector.Html;
import us.codecraft.webmagic.selector.PlainText;
import us.codecraft.webmagic.utils.UrlUtils;

import java.util.Map;

/**
 * @author taojw
 *
 */
public class SeleniumDownloader  implements Downloader{
	private static final Logger log=LoggerFactory.getLogger(SeleniumDownloader.class);
	private int sleepTime=3000;//3s
	private SeleniumAction action=null;
	private WebDriverPool webDriverPool=new WebDriverPool();
	public SeleniumDownloader(){
	}
	public SeleniumDownloader(int sleepTime,WebDriverPool pool){
		this(sleepTime,pool,null);
	}
	public SeleniumDownloader(int sleepTime,WebDriverPool pool,SeleniumAction action){
		this.sleepTime=sleepTime;
		this.action=action;
		if(pool!=null){
			webDriverPool=pool;
		}
	}
	public SeleniumDownloader setSleepTime(int sleepTime) {
		this.sleepTime = sleepTime;
		return this;
	}
	public void setOperator(SeleniumAction action){
		this.action=action;
	}
	@Override
	public Page download(Request request, Task task) {
		WebDriver webDriver;
		try {
			webDriver = webDriverPool.get();
		} catch (InterruptedException e) {
			log.warn("interrupted", e);
			return null;
		}
		log.info("downloading page " + request.getUrl());
		Page page = new Page();
		try {
			webDriver.get(request.getUrl());
			Thread.sleep(sleepTime);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (Exception e) {
			webDriverPool.close(webDriver);
			page.setSkip(true);
			return page;
		}
//		WindowUtil.changeWindow(webDriver);
		WebDriver.Options manage = webDriver.manage();
		Site site = task.getSite();
		if (site.getCookies() != null) {
			for (Map.Entry<String, String> cookieEntry : site.getCookies()
					.entrySet()) {
				Cookie cookie = new Cookie(cookieEntry.getKey(),
						cookieEntry.getValue());
				manage.addCookie(cookie);
			}
		}
		manage.window().maximize();
		if(action!=null){
			action.execute(webDriver);
		}
		SeleniumAction reqAction=(SeleniumAction) request.getExtra("action");
		if(reqAction!=null){
			reqAction.execute(webDriver);
		}

		WebElement webElement = webDriver.findElement(By.xpath("/html"));
		String content = webElement.getAttribute("outerHTML");
		
		page.setRawText(content);
		page.setHtml(new Html(UrlUtils.fixAllRelativeHrefs(content,
				webDriver.getCurrentUrl())));
		page.setUrl(new PlainText(webDriver.getCurrentUrl()));
		page.setRequest(request);
		webDriverPool.returnToPool(webDriver);
		return page;
	}

	@Override
	public void setThread(int thread) {
		
	}

}
```

功能：
支持在Spider.setDownloader的时候添加钩子SeleniumAction来实现自定义selenium的通用操作。加强了灵活性
支持对每个请求添加action参数，参数值为SeleniumAction对象，进而可以对每个请求实现自定义selenium操作.加强了灵活性

```
import org.openqa.selenium.WebDriver;

/**
 * @author taojw
 *
 */
public interface SeleniumAction {
	void execute(WebDriver driver);
}
```

WebDriverPool实现：注意对WebDriver的池化来保证性能
也是参考webmagic-selenium作了些修改。

```
import com.fh.util.FileUtil;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.phantomjs.PhantomJSDriver;
import org.openqa.selenium.phantomjs.PhantomJSDriverService;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.concurrent.BlockingDeque;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @author taojw
 */
public class WebDriverPool {
	private Logger logger = LoggerFactory.getLogger(getClass());

	private int CAPACITY = 5;
	private AtomicInteger refCount = new AtomicInteger(0);
	private static final String DRIVER_PHANTOMJS = "phantomjs";

	/**
	 * store webDrivers available
	 */
	private BlockingDeque<WebDriver> innerQueue = new LinkedBlockingDeque<WebDriver>(
			CAPACITY);

	private static String PHANTOMJS_PATH;
	private static DesiredCapabilities caps = DesiredCapabilities.phantomjs();
	static {
		PHANTOMJS_PATH = FileUtil.getCommonProp("phantomjs.path");
		caps.setJavascriptEnabled(true);
		caps.setCapability(
				PhantomJSDriverService.PHANTOMJS_EXECUTABLE_PATH_PROPERTY,
				PHANTOMJS_PATH);
		caps.setCapability("takesScreenshot", true);
		caps.setCapability(
				PhantomJSDriverService.PHANTOMJS_PAGE_CUSTOMHEADERS_PREFIX
						+ "User-Agent",
				"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/50.0.2661.102 Safari/537.36");
		caps.setCapability(PhantomJSDriverService.PHANTOMJS_CLI_ARGS,
				"--load-images=no");

	}

	public WebDriverPool() {
	}

	public WebDriverPool(int poolsize) {
		this.CAPACITY = poolsize;
		innerQueue = new LinkedBlockingDeque<WebDriver>(poolsize);
	}

	public WebDriver get() throws InterruptedException {
		WebDriver poll = innerQueue.poll();
		if (poll != null) {
			return poll;
		}
		if (refCount.get() < CAPACITY) {
			synchronized (innerQueue) {
				if (refCount.get() < CAPACITY) {

					WebDriver mDriver = new PhantomJSDriver(caps);
					// 尝试性解决：https://github.com/ariya/phantomjs/issues/11526问题
					mDriver.manage().timeouts()
							.pageLoadTimeout(60, TimeUnit.SECONDS);
					// mDriver.manage().window().setSize(new Dimension(1366,
					// 768));
					innerQueue.add(mDriver);
					refCount.incrementAndGet();
				}
			}
		}
		return innerQueue.take();
	}

	public void returnToPool(WebDriver webDriver) {
		// webDriver.quit();
		// webDriver=null;
		innerQueue.add(webDriver);
	}

	public void close(WebDriver webDriver) {
		refCount.decrementAndGet();
		webDriver.close();
		webDriver.quit();
		webDriver = null;
	}

	public void shutdown() {
		try {
			for (WebDriver driver : innerQueue) {
				close(driver);
			}
			innerQueue.clear();
		} catch (Exception e) {
//			e.printStackTrace();
			logger.warn("webdriverpool关闭失败",e);
		}
	}
}

```

修改后：
仅支持PhantomJS作为浏览器驱动。
增加phantomjs相关配置
修改队列大小控制逻辑


WindowUtil
注意这个loadAll方法的实现很巧妙哦，由于涉及滚动加载页面的时候，如果一下子滚到底部可能会造成中间部分没有加载出来，这样就不得不针对每个页面进行满满滚动。而loadAll采取的思路是直接获取页面可滚动大小，然后将浏览器窗口调成对应大小，刷新之后所有内容便加载出来了。
```
import org.apache.commons.io.FileUtils;
import org.openqa.selenium.*;

import java.io.File;
import java.io.IOException;

/**
 * @author taojw
 *
 */
public class WindowUtil {
	
	/**
	 * 滚动窗口。
	 * @param driver
	 * @param height
	 */
	public static void scroll(WebDriver driver,int height){
		((JavascriptExecutor)driver).executeScript("window.scrollTo(0,"+height+" );");	
	}
	/**
	 * 重新调整窗口大小，以适应页面，需要耗费一定时间。建议等待合理的时间。
	 * @param driver
	 */
	public static void loadAll(WebDriver driver){
		Dimension od=driver.manage().window().getSize();
		int width=driver.manage().window().getSize().width;
		//尝试性解决：https://github.com/ariya/phantomjs/issues/11526问题
        driver.manage().timeouts().pageLoadTimeout(60, TimeUnit.SECONDS); 
		long height=(Long)((JavascriptExecutor)driver).executeScript("return document.body.scrollHeight;");
		driver.manage().window().setSize(new Dimension(width, (int)height));
		driver.navigate().refresh();
	}
	public static void taskScreenShot(WebDriver driver,File saveFile){
		File src=((TakesScreenshot)driver).getScreenshotAs(OutputType.FILE);
		try {
			FileUtils.copyFile(src, saveFile);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	public static void changeWindow(WebDriver driver){
		// 获取当前页面句柄
		String handle = driver.getWindowHandle();
		// 获取所有页面的句柄，并循环判断不是当前的句柄，就做选取switchTo()
		for (String handles : driver.getWindowHandles()) {
			if (handles.equals(handle))
				continue;
			driver.switchTo().window(handles);
		}
	}
}
```

至此对爬虫框架的扩展高一段落。

# 实战部分

## 抓取淘宝店铺信息

```
/**
 * 店铺销售信息
 *
 * @author taojw
 */
@Scope("prototype")
@Component
public class TaoBaoShopInfoProcessor implements PageProcessor {
    private static final Logger log = LoggerFactory
            .getLogger(TaoBaoShopInfoProcessor.class);

    @Autowired
    private TaoBaoShopInfoService service;

    private Site site = Site
            .me()
            .setCharset("UTF-8")
            .setCycleRetryTimes(3)
            .setSleepTime(3 * 1000)
            .addHeader("Connection", "keep-alive")
            .addHeader("Cache-Control", "max-age=0")
            .addHeader("User-Agent",
                    "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:50.0) Gecko/20100101 Firefox/50.0");

    private AtomicBoolean isPageAdd = new AtomicBoolean(false);
    private static AtomicBoolean running = new AtomicBoolean(false);
    private WebDriverPool pool=new WebDriverPool();
    @Override
    public Site getSite() {
        return this.site;
    }

    @Override
    public void process(Page page) {
        if (islistPage(page)) {
            List<String> urls = page.getHtml()
                    .$("dl.item a.J_TGoldData", "href").all();
            List<String> targetUrls = new ArrayList<String>();
            for (String url : urls) {
                targetUrls.add(url.trim());
            }
            page.addTargetRequests(targetUrls);
            if (isPageAdd.compareAndSet(false, true)) {
                // 分页处理
                String pageinfo = page.getHtml()
                        .$(".pagination .page-info", "text").get();
                int pageCount = Integer.valueOf(pageinfo.split("/")[1]);
                String cururl = page.getUrl().get();
                //只抓前5页
                if(pageCount>5){
                    pageCount=5;
                }
                for (int i = 1; i < pageCount; i++) {
                    String tmp = cururl + "&pageNo=" + (i + 1);
                    page.addTargetRequest(tmp);
                }
            }
            return;
        }

        // 商品页面
        String curUrl = page.getUrl().get();
        boolean isTaoBao=curUrl.startsWith("https://item.taobao.com");
        boolean isTmall=curUrl.startsWith("https://detail.tmall.com");
        
        String tmpspm = curUrl.split("\\?")[1].split("&")[0];
        // spm码
        String spm = tmpspm.split("=")[1];
        // 网店地址
        String shopUrl="";
     // 商品名称
        String name="";
     // 价格
        double price =0; 
     // 30天交易总数
        int sellCount=0;
     // 交易总价
        double allPrice=0;
        if(isTaoBao){
        	shopUrl= page.getHtml()
                    .xpath("//div[@class='tb-shop-name']/dl/dd/strong/a/@href")
                    .get();
        	shopUrl = shopUrl.split("\\?")[0];
        	
        	name = page.getHtml().xpath("//*[@id='J_Title']/h3/text()")
                    .get();
        	try{
                price=Double.valueOf(page.getHtml()
                        .$("#J_PromoPriceNum", "text").get().split("-")[0].trim());
                }catch(Exception e){
                	
                	price=Double.valueOf(page.getHtml()
                            .$("#J_StrPrice .tb-rmb-num", "text").get().split("-")[0].trim());
                }
        	sellCount = Integer.valueOf(page.getHtml()
                    .$("#J_SellCounter", "text").get());
        	allPrice = Double.valueOf(price) * Double.valueOf(sellCount);
        }else if(isTmall){
        	shopUrl= page.getHtml()
                    .xpath("//*[@id='side-shop-info']/div/h3/div/a/@href")
                    .get();
        	shopUrl = shopUrl.split("\\?")[0];
        	
        	name = page.getHtml().$(".tb-detail-hd h1","text")
                    .get().trim();
        
            price=Double.valueOf(page.getHtml()
                        .$(".tm-price", "text").get().split("-")[0].trim());
                
        	sellCount = Integer.valueOf(page.getHtml()
                    .$(".tm-count", "text").get().trim());
        	allPrice = Double.valueOf(price) * Double.valueOf(sellCount);
        }

        // 采集日期
        // Timestamp recordDate=new Timestamp(new Date().getTime());
        String recordDate = DateUtil.formatDate(new Date(), "yyyy-MM-dd");

        log.debug(shopUrl + ":" + spm + ":" + name + ":" + price + ":"
                + sellCount + ":" + allPrice + ":" + recordDate);

        PageData pd = new PageData();
        pd.put("id", UUID.randomUUID().toString());
        pd.put("shopUrl", shopUrl);
        pd.put("spm", spm);
        pd.put("name", name);
        pd.put("price", price);
        pd.put("sellCount", sellCount);
        pd.put("allPrice", allPrice);
        pd.put("recordDate", recordDate);
        service.saveData(pd);
    }

    private boolean islistPage(Page page) {
        String tmp = page.getHtml().$("#J_PromoPrice").get();
        if (StringUtils.isBlank(tmp)) {
            return true;
        }
        return false;
    }

    public void start() {
        if (running.compareAndSet(false, true)) {
            try {
                service.emptyTable();
                List<String> urls = service.getShopUrl();
                if (urls == null) {
                    log.error("店铺url获取异常,终止抓取");
                }
                String[] urlStrs=null;
                int size=50;
//                int size=urls.size();
                if(urls.size()<size){
                	urlStrs=new String[urls.size()];
                	urlStrs=urls.toArray(urlStrs);
                	
                }else{
                	urlStrs=new String[size];
                	for(int i=0;i<size;i++){
                		urlStrs[i]=urls.get(i).trim();
                	}
                }
                log.info("准备抓取,需要抓取的店铺数为{}", urls.size());
               // String[] urlStrs = new String[urls.size()];
              //  "https://zhuzhuwo.taobao.com" urls.toArray(urlStrs)
                Spider spider = Spider.create(this)
                        .setDownloader(new SeleniumDownloader(5000, pool, new TestAction()))
                        .addUrl(urlStrs);
                      //  .addUrl("https://zhuzhuwo.taobao.com");

                spider.thread(5).run();
                log.info("淘宝店铺销售信息数据正常抓取完毕");
            } finally {
            	log.info("淘宝店铺销售信息数据抓取完毕,准备关闭webdriverpool");
            	pool.shutdown();
            	log.info("webdriverpool关闭完毕");
                running.set(false);
            }
        }
    }

    public static void main(String[] args) {
        new TaoBaoShopInfoProcessor().start();
    }

    private class TestAction implements SeleniumAction {

        @Override
        public void execute(WebDriver driver) {
            WebDriverWait wait = new WebDriverWait(driver, 10);
            // 商品页，避免加载过多无用图片信息。
            if (driver.getCurrentUrl().startsWith("https://item.taobao.com/")|| driver.getCurrentUrl().startsWith("https://detail.tmall.com")) {
              //  wait.until(ExpectedConditions.presenceOfElementLocated(By
              //          .cssSelector("#J_PromoPriceNum")));
                return;
            }
            // 店铺首页，点击所有分类
            if ((!(driver.getCurrentUrl().startsWith("https://item.taobao.com")))
                    && !(driver.getCurrentUrl().startsWith("https://detail.tmall.com")) && (!driver.getCurrentUrl().contains("search"))) {
                WebElement allcate = driver.findElement(By
                        .cssSelector(".all-cats-trigger a"));
                Actions action = new Actions(driver);
                action.click(allcate).perform();
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            String url = driver.getCurrentUrl();
            // 列表页，加载所有
            WindowUtil.loadAll(driver);
            url = driver.getCurrentUrl();
            try {
                Thread.sleep(3000);
//                WindowUtil.taskScreenShot(driver, new File("d:\\data\\tb\\" + UUID.randomUUID().toString() + ".png"));
                // wait.until(ExpectedConditions.presenceOfElementLocated(By.cssSelector(".pagination .page-info")));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}
```

## 抓取猫眼票房数据

由于猫眼票房数据采用加密字体图标，而且每个数字对应的加密码每次都变化。所以此次采用selenium加载页面，截图，抠图(给每个数字)，考虑到猫眼票房数据的规则性，结合google的 Tesseract-OCR 训练模型来识别我们抠出来的数字图片。

ImageUtil 负责抠图
```
import net.coobird.thumbnailator.Thumbnails;
import net.coobird.thumbnailator.geometry.Position;
import net.coobird.thumbnailator.geometry.Size;

/**
 * @author taojw
 *
 */
public class ImageUtil {
	public static void crop(String srcfile,String destfile,ImageRegion region){
		//指定坐标  
		try {
			Thumbnails.of(srcfile)  
			        .sourceRegion(region.x, region.y, region.width, region.height)  
			        .size(region.width, region.height).outputQuality(1.0) 
			        //.keepAspectRatio(false)  //不保持比例 
			        .toFile(destfile);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}  
	}
	public static void main(String[] args) {
		crop("D:\\data\\111.png","D:\\data\\1112.png",new ImageRegion(66, 264, 422, 426));
	}
}
```

```
/**
 * @author taojw
 *
 */
public class ImageRegion {
	public int x;
	public int y;
	public int width;
	public int height;
	public ImageRegion(int x,int y,int width,int height){
		this.x=x;
		this.y=y;
		this.width=width;
		this.height=height;
	}
}
```

TesseractOcrUtil，调用tesseract进程，返回识别结果。
```
import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.UUID;

import org.apache.commons.io.FileUtils;
import org.apache.commons.io.IOUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.fh.util.FileUtil;

/**
 * @author taojw
 *
 */
public class TesseractOcrUtil {
	private static final Logger log = LoggerFactory
			.getLogger(TesseractOcrUtil.class);
	private static final String tessPath;
	private static final String basePath;
	static {
		tessPath = FileUtil.getCommonProp("tesseract.path");
		basePath = new File(tessPath).getParentFile().getAbsolutePath();
	}

	public static String getByLangNum(String imagePath) {
		return get(imagePath, "num");
	}

	public static String getByLangChi(String imagePath) {
		return get(imagePath, "chi_sim");
	}

	public static String getByLangEng(String imagePath) {
		return get(imagePath, "eng");
	}

	public static String get(String imagePath, String lang) {
		String outName = UUID.randomUUID().toString();
		String outPath = basePath + File.separator
				+ outName + ".txt";
//		String cmd = tessPath + " " + imagePath + " " + outName + " -l " + lang;
		ProcessBuilder pb = new ProcessBuilder();
		pb.directory(new File(basePath));
		
		pb.command(tessPath,imagePath,outName,"-l",lang);
		
		pb.redirectErrorStream(true);
		
		Process process=null;
		String errormsg = "";
		String res = null;
		try {
			process = pb.start();
			// tesseract.exe 1.jpg 1 -l chi_sim
			int excode = process.waitFor();
			
			if (excode == 0) {
				BufferedReader in = new BufferedReader(new InputStreamReader(
						new FileInputStream(outPath), "UTF-8"));
				res = in.readLine();
				IOUtils.closeQuietly(in);
			} else {
				switch (excode) {
				case 1:
					errormsg = "Errors accessing files.There may be spaces in your image's filename.";
					break;
				case 29:
					errormsg = "Cannot recongnize the image or its selected region.";
					break;
				case 31:
					errormsg = "Unsupported image format.";
					break;
				default:
					errormsg = "Errors occurred.";
				}
				log.error("when ocr picture " + imagePath
						+ " an error occured. " + errormsg);
			}

		} catch (IOException e) {
			e.printStackTrace();
			log.warn("orc process occurs an io error",e);
		} catch (InterruptedException e) {
			e.printStackTrace();
			log.warn("orc process was interrupt unexpected!",e);
		}finally{
			FileUtils.deleteQuietly(new File(imagePath));
			FileUtils.deleteQuietly(new File(outPath));
		}
		if(res!=null){
			res=res.trim();
		}
		return res;
	}
}
```

```
/**
 * @author taojw
 *
 */
public class MaoyanTest implements PageProcessor{
	private static Site site=Site.me().setCharset("UTF-8").setUserAgent(
			"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_2) AppleWebKit/537.31 (KHTML, like Gecko) Chrome/26.0.1410.65 Safari/537.31");
	@Override
	public Site getSite() {
		return site;
	}

	@Override
	public void process(Page page) {

	}
	public  void start() {
		
    	Spider cnSpider = Spider.create(this).setDownloader(new SeleniumDownloader(5000,null,new TestAction()))
//    			.addUrl("https://shop34068488.taobao.com/?spm=a230r.7195193.1997079397.2.JLFlPa")
//    			.addUrl("http://piaofang.maoyan.com/company/cinema?date=2017-01-18&webCityId=288&cityTier=0&page=1&cityName=%E6%8F%AD%E9%98%B3");
    			.addUrl("http://piaofang.maoyan.com/company/cinema?date=2017-01-18&webCityId=84&cityTier=0&page=1&cityName=%E4%BF%9D%E5%AE%9A");
//    			.addPipeline(new JsonFilePipeline("D:\\data\\webmagicfile.json"))
    	
    	//SpiderMonitor.instance().register(cnSpider);
    	cnSpider.run();
	}
	public static void main(String[] args) {
		new MaoyanTest().start();
	}
	
	private class TestAction implements SeleniumAction{

		@Override
		public void execute(WebDriver driver) {
			WindowUtil.loadAll(driver);
			try {
				Thread.sleep(5000);
				//WebDriverWait wait = new WebDriverWait(driver, 10);
		        //wait.until(ExpectedConditions.presenceOfElementLocated(By.id("J_PromoPriceNum")));
		        
				File src=((TakesScreenshot)driver).getScreenshotAs(OutputType.FILE);
				String srcfile="D:\\data\\"+UUID.randomUUID().toString()+".png";
				FileUtils.copyFile(src, new File(srcfile));
				List<WebElement> movielist=driver.findElements(By.xpath("//*[@id='cinema-tbody']/tr"));
//				movielist.remove(0);
				for(int i=1;i<movielist.size();i++){
					int index=i+1;
					String movieName=driver.findElement(By.xpath("//*[@id='cinema-tbody']/tr["+index+"]/td[2]")).getText();
					
					String pattern = "//*[@id='cinema-tbody']/tr["+index+"]/td[3]";
					WebElement tel=driver.findElement(By.xpath(pattern));
					
					Point loc=tel.getLocation();
					Dimension d=tel.getSize();
					String cop_path="D:\\data\\crop\\current_piaofang_"+movieName+".png";
					ImageUtil.crop(srcfile, cop_path, new ImageRegion(loc.x, loc.y, d.width+10, d.height));
					System.out.println(TesseractOcrUtil.getByLangNum(cop_path));
					FileUtils.deleteQuietly(new File(srcfile));
				}

			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		
	}
}

```

可供参考链接：
运用phantomjs无头浏览器破解四种反爬虫技术：http://python.jobbole.com/86415/
selenium系列文章：http://www.cnblogs.com/TankXiao/p/5252754.html
selenium api：http://seleniumhq.github.io/selenium/docs/api/java/
tesseract-ocr样本训练: http://blog.csdn.net/firehood_/article/details/8433077
selenium多窗口切换：http://blog.csdn.net/meyoung01/article/details/13289177
http://stackoverflow.com/questions/21943379/switch-to-new-window-only-if-it-opens-in-new-window-and-verify-whether-page-is-l
Java OCR tesseract 图像智能字符识别技术 Java代码实现 http://blog.csdn.net/lmj623565791/article/details/23960391/
tesseract-ocr 提高验证码识别率手段 http://www.xuebuyuan.com/709203.html
http://stackoverflow.com/questions/32677313/selenium-phantomjs-custom-headers-in-java
http://stackoverflow.com/questions/24138398/how-to-implement-phantomjs-with-selenium-webdriver-using-java
selenium滚动条操作：
http://www.cnblogs.com/woniu123/p/6163392.html
在Selenium测试中获取JavaScript的执行结果：
http://libin0019.iteye.com/blog/1152860
使用Tesseract识别弱验证码：http://udn.yyuap.com/doc/ae/920457.html
[Python爬虫] 「暴力」破解猫眼电影票房数据的反爬虫机制：https://jizhi.im/blog/post/maoyan-anti-crawler
