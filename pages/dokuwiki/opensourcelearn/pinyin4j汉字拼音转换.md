title: pinyin4j汉字拼音转换 

#  pinyin4j汉字拼音转换 

Pinyin4j是一个流行的Java库，支持中文字符和拼音之间的转换。拼音输出格式可以定制。
项目地址:http://pinyin4j.sourceforge.net/ 
特性：
+ 支持同一汉字有多个发音
+ 还支持拼音的格式化输出，比如第几声之类的，
+ 同时支持简体中文、繁体中文转换为拼音…使用起来也非常简单。
#  汉字转拼音： 
String[] pinyin = PinyinHelper.toHanyuPinyinStringArray('重');
上面这行代码就是单个汉字转拼音了，例如“重”字，该方法返回一个String类型的数组：
"zhong4"
"chong2"
“重”是一个多音字，该方法的返回数组包含这个字的所有读音的拼音。每个读音最后有个数字就是音调（第一声 第二声 第三声 第四声，这个不用解释了）。
上面是最简单的一种获取单个汉字的方式，还可以使用HanyuPinyinOutputFormat来格式化返回拼音的格式。
```

HanyuPinyinOutputFormat format = new HanyuPinyinOutputFormat();
// UPPERCASE：大写  (ZHONG)
// LOWERCASE：小写  (zhong)
format.setCaseType(HanyuPinyinCaseType.LOWERCASE);
// WITHOUT_TONE：无音标  (zhong)
// WITH_TONE_NUMBER：1-4数字表示英标  (zhong4)
// WITH_TONE_MARK：直接用音标符（必须设置setVCharType(HanyuPinyinVCharType.WITH_U_UNICODE)否则异常）  (zhòng)
format.setToneType(HanyuPinyinToneType.WITH_TONE_MARK);

// WITH_V：用v表示ü  (nv)
// WITH_U_AND_COLON：用"u:"表示ü  (nu:)
// WITH_U_UNICODE：直接用ü (nü)
format.setVCharType(HanyuPinyinVCharType.WITH_U_UNICODE);
String[] pinyin = PinyinHelper.toHanyuPinyinStringArray('重', format);

```
**toHanyuPinyinStringArray如果传入的字符不是汉字不能转换成拼音，那么会直接返回null。**

#  很好的一个Demo 

```

import net.sourceforge.pinyin4j.PinyinHelper;
import net.sourceforge.pinyin4j.format.HanyuPinyinCaseType;
import net.sourceforge.pinyin4j.format.HanyuPinyinOutputFormat;
import net.sourceforge.pinyin4j.format.HanyuPinyinToneType;
import net.sourceforge.pinyin4j.format.exception.BadHanyuPinyinOutputFormatCombination;

import java.io.UnsupportedEncodingException;

/**
 * 拼音工具
 * 
 * @author leizhimin 2009-7-15 15:26:21
 */
public class PinyinToolkit {

	/**
	 * 获取汉字串拼音首字母，英文字符不变
	 * 
	 * @param chinese
	 *            汉字串
	 * @return 汉语拼音首字母
	 */
	public static String cn2FirstSpell(String chinese) {
		StringBuffer pybf = new StringBuffer();
		char[] arr = chinese.toCharArray();
		HanyuPinyinOutputFormat defaultFormat = new HanyuPinyinOutputFormat();
		defaultFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
		defaultFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
		for (int i = 0; i < arr.length; i++) {
			if (arr[i] > 128) {
				try {
					String[] _t = PinyinHelper.toHanyuPinyinStringArray(arr[i],
							defaultFormat);
					if (_t != null) {
						pybf.append(_t[0].charAt(0));
					}
				} catch (BadHanyuPinyinOutputFormatCombination e) {
					e.printStackTrace();
				}
			} else {
				pybf.append(arr[i]);
			}
		}
		// return pybf.toString().replaceAll("\\W", "").trim();
		return pybf.toString().trim();
	}

	/**
	 * 获取汉字串拼音，英文字符不变
	 * 
	 * @param chinese
	 *            汉字串
	 * @return 汉语拼音
	 */
	public static String cn2Spell(String chinese) {
		StringBuffer pybf = new StringBuffer();
		char[] arr = chinese.toCharArray();
          	// // 输出设置，大小写，音标方式等
		HanyuPinyinOutputFormat defaultFormat = new HanyuPinyinOutputFormat();
		defaultFormat.setCaseType(HanyuPinyinCaseType.LOWERCASE);
		defaultFormat.setToneType(HanyuPinyinToneType.WITHOUT_TONE);
		for (int i = 0; i < arr.length; i++) {
                 // 也可以采用如下形式
                  /**
                  *是中文转换拼音否则保留a-z或者A-Z。
			  * 	if(String.valueOf(c).matches("[//u4E00-//u9FA5]+")){ //中文字符
                  *或者换一种思考，先用String ar=PinyinHelper.toHanyuPinyinStringArray(arr[i],
			  *				defaultFormat)[0]
                  *     然后判断ar是否为null来确定是否为中文字符。                 
                  */
			if (arr[i] > 128) {
				try {
					pybf.append(PinyinHelper.toHanyuPinyinStringArray(arr[i],
							defaultFormat)[0]);
				} catch (BadHanyuPinyinOutputFormatCombination e) {
					e.printStackTrace();
				}
			} else {
				pybf.append(arr[i]);
			}
		}
		return pybf.toString();
	}

	public static void main(String[] args) throws UnsupportedEncodingException {
		String x = "我是一个小学生重启";
		System.out.println(cn2FirstSpell(x));
		System.out.println(cn2Spell(x));
	}
}

```
输出：
wsygxxszq
woshiyigexiaoxueshengzhongqi
#  总结： 

虽然pinyin4j很好用，但是还是有局限的。但是不能获取一个包含多音字的词的拼音。例如“重庆”，无法判断到底是“chongqing”还是“zhongqing”，pinyin4j不能通过上下文来判断多音字的读音。

参考：
http://blog.csdn.net/tanguang_honesty/article/details/8560255
http://lavasoft.blog.51cto.com/62575/178320/
http://my.oschina.net/hongdengyan/blog/152639
http://blog.csdn.net/foamflower/article/details/6209552