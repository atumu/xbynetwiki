title: android文字与语音转换 

#  Android TTS文字转语音开发 
使用讯飞平台：http://www.xfyun.cn/
使用百度平台：http://yuyin.baidu.com

#  百度语音平台开发 
##  语音合成TTS 
###  安装 

将开发包中的 libs 目录整体拷贝到工程目录，libs 目录包括了各平台的 SO 库，开发者视应用需要可以进行删减。galaxy_lite.jar 是百度 Android 公共基础库，如果项目中还集成了其它百度 SDK，如 Push SDK，在打包过程中出现类似如下的错误信息：
Unable to execute dex: Multiple dex files defineLcom/baidu/android/common/logging/Log
请将此 Jar 包移除。
如果 Eclipse ADT 版本插件低于 17，需要手工添加依赖库，添加方法为：Project => Properties =>Java Build Path => Libraries => Add JAR…
###  权限声明 

BDTTSClient 需要权限如下：
名称	用途
android.permission.INTERNET	允许应用联网，获取合成的音频数据
android.permission.ACCESS_NETWORK_STATE	获取当前网络状态，优化网络故障提示
android.permission.READ_PHONE_STATE	获取设备的 ID 信息，用于生成统计数据
需要在 AndroidManifest.xml 文件，增加以上三个权限：
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
##  语音合成 
简单示例：
// 注：第二个参数当前请传入任意非空字符串即可
SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(getApplicationContext(), "holder", this);
// 注：your-apiKey 和 your-secretKey 需要换成在百度开发者中心注册应用得到的对应值
speechSynthesizer.setApiKey("your-apiKey", "your-secretKey");
speechSynthesizer.speak("百度一下");
具体请查看官方文档

###  日志： 
为了方便调试，BDTTSClient 提供了对日志级别进行设置的接口，通过如下代码可以设置日志级别。
SpeechLogger.setLogLevel(SpeechLogger.SPEECH_LOG_LEVEL_DEBUG);
提供 6 个级别的日志，从低到高依次为VERBOSE、DEBUG、INFO、WARN、ERROR、OFF，其中VERBOSE将输出最多日志，OFF将关闭所有日志，默认日志级别为DEBUG。

