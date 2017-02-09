title: java日期操作技巧 

#  java日期操作技巧 
##  JAVA日期加减运算 
参考：http://blog.csdn.net/liwenfeng1022/article/details/6534176
1.用java.util.Calender来实现
```

   Calendar calendar=Calendar.getInstance();   
   calendar.setTime(new Date()); 
   System.out.println(calendar.get(Calendar.DAY_OF_MONTH));//今天的日期 
   calendar.set(Calendar.DAY_OF_MONTH,calendar.get(Calendar.DAY_OF_MONTH)+1);//让日期加1  
   System.out.println(calendar.get(Calendar.DATE));//加1之后的日期Top

``` 
2.用java.text.SimpleDateFormat和java.util.Date来实现           
```

    Date d=new Date();   
   SimpleDateFormat df=new SimpleDateFormat("yyyy-MM-dd");   
   System.out.println("今天的日期："+df.format(d));   
   System.out.println("两天前的日期：" + df.format(new Date(d.getTime() - 2 * 24 * 60 * 60 * 1000)));  
   System.out.println("三天后的日期：" + df.format(new Date(d.getTime() + 3 * 24 * 60 * 60 * 1000)));

```

3、GregorianCalendar来实现
GregorianCalendar gc=new GregorianCalendar(); 
gc.setTime(new Date); 
gc.add(field,value); 
value为正则往后,为负则往前 
field取1加1年,取2加半年,取3加一季度,取4加一周 
取5加一天....

/*
*java中对日期的加减操作
*gc.add(1,-1)表示年份减一.
*gc.add(2,-1)表示月份减一.
*gc.add(3.-1)表示周减一.
*gc.add(5,-1)表示天减一.
*以此类推应该可以精确的毫秒吧.没有再试.大家可以试试.
*GregorianCalendar类的add(int field,int amount)方法表示年月日加减.
*field参数表示年,月.日等.
*amount参数表示要加减的数量.

##  毫秒转时间间隔 
我们知道Java8已经支持时间间隔了。但是我用的是java7.所以手动处理以下：
```

/**
	 * 处理媒体文件播放间隔
	 * @param millis 时间时间毫秒
	 * @return 如03:01:00 时长3小时1分0秒
	 */
	public  static String formatMediaDuration(long millis){
			long seconds=millis/1000;
			int second=(int) (seconds%60);
			int minute=(int)((seconds/60)%60);
			int hour=(int)((seconds/(60*60))%24);
			String str=formatTimeUnit(hour)+":"+formatTimeUnit(minute)+":"+formatTimeUnit(second);
			
			return str;
	}
	/**
	 * 格式化时间单元，支持秒，时，分。如3格式化为03
	 * @param unit
	 * @return
	 */
	private static String formatTimeUnit(int unit){
		String str="";
		if(unit<10){
			if(unit==0){
				str=unit+"0";
			}else{
				str="0"+unit;
			}
		}else{
			str=""+unit;
		}
		return str;
	}

```