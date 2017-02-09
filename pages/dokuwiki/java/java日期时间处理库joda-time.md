title: java日期时间处理库joda-time 

#  java日期时间处理库Joda-Time 
JodaTime 提供了一组Java类包用于处理包括ISO8601标准在内的date和time。可以利用它把JDK Date和Calendar类完全替换掉，而且仍然能够提供很好的集成。
Joda-Time主要的特点包括：
1. 易于使用:Calendar让获取"正常的"的日期变得很困难，使它没办法提供简单的方法，而Joda-Time能够 直接进行访问域并且索引值1就是代表January。
2. 易于扩展：JDK支持多日历系统是通过Calendar的子类来实现，这样就显示的非常笨重而且事实 上要实现其它日历系统是很困难的。Joda-Time支持多日历系统是通过基于Chronology类的插件体系来实现。
3. 提供一组完整的功能：它打算提供 所有关系到date-time计算的功能．Joda-Time当前支持6种日历系统，而且在将来还会继续添加。有着比JDK Calendar更好的整体性能等等。

 目前Joda Time 已经纳入 JDK 8 的官方API了
Github：https://github.com/JodaOrg/joda-time
官网：http://www.joda.org/joda-time/
API：http://www.joda.org/joda-time/apidocs/index.html
快速入门:http://www.joda.org/joda-time/quickstart.html
完整：http://www.joda.org/joda-time/userguide.html

Maven:
```

<dependency>
  <groupId>joda-time</groupId>
  <artifactId>joda-time</artifactId>
  <version>2.9.1</version>
</dependency>

```
下面是一些代码示例：
```

public boolean isAfterPayDay(DateTime datetime) {
  if (datetime.getMonthOfYear() == 2) {   // February is month 2!!
    return datetime.getDayOfMonth() > 26;
  }
  return datetime.getDayOfMonth() > 28;
}

public Days daysToNewYear(LocalDate fromDate) {
  LocalDate newYear = fromDate.plusYears(1).withDayOfYear(1);
  return Days.daysBetween(fromDate, newYear);
}

public boolean isRentalOverdue(DateTime datetimeRented) {
  Period rentalPeriod = new Period().withDays(2).withHours(12);
  return datetimeRented.plus(rentalPeriod).isBeforeNow();
}

public String getBirthMonthText(LocalDate dateOfBirth) {
  return dateOfBirth.monthOfYear().getAsText(Locale.ENGLISH);
}

```

##  主要接口与类 
  * Instant - Immutable class 代表即时的时间点
  * DateTime - Immutable class 替代 JDK Calendar
  * LocalDate - Immutable class representing a local date **without a time** (no time-zone)
  * LocalTime - Immutable class representing a time **without a date** (no time-zone)
  * LocalDateTime - Immutable class representing a local date and time (no time-zone)
  * Interval、Period、Duration代表时间间隔
An Instant is a good class to use for the timestamp of an event,

##  Using the Date and Time classes 
```

DateTime dateTime = new DateTime(
  2000, //year
  1,    // month
  1,    // day
  0,    // hour (midnight is zero)
  0,    // minute
  0,    // second
  0     // milliseconds
);

```
提供了接受各种形式参数的构造器：
  * Date - a JDK instant
  * Calendar - a JDK calendar
  * String - in ISO8601 format
  * Long - in milliseconds
  * any Joda-Time date-time class

```

java.util.Date juDate = new Date();
  DateTime dt = new DateTime(juDate);

  DateTime dt = new DateTime();
  int month = dt.getMonthOfYear();  // where January is 1 and December is 12
  int year = dt.getYear();

```

` All the main date-time classes are immutable `, like String, and cannot be changed after creation.
However, simple methods have been provided to alter field values in a newly created object. **For example, to set the year, or add 2 hours you can use:**
```

  DateTime dt = new DateTime();
  DateTime year2000 = dt.withYear(2000);
  DateTime twoHoursLater = dt.plusHours(2);

```
In addition to the basic get methods, each date-time class provides property methods for each field. These provide access to the full wealth of Joda-Time functionality. For example, to access details about a month or year:
```

  DateTime dt = new DateTime();
  String monthName = dt.monthOfYear().getAsText();
  String frenchShortName = dt.monthOfYear().getAsShortText(Locale.FRENCH);
  boolean isLeapYear = dt.year().isLeap();
  DateTime rounded = dt.dayOfMonth().roundFloorCopy();

```
##  Calendar systems and time-zones 
Joda-Time提供支持多日历系统和各种时区。由`  Chronology and DateTimeZone ` classes 提供支持。
默认ISO日历系统
Joda-Time支持多日历系统是通过基于Chronology类的插件体系来实现。
 By contrast, the JDK uses subclasses such as GregorianCalendar. This code obtains a Joda-Time chronology by calling one of the factory methods on the Chronology implementation:
```

  Chronology coptic = CopticChronology.getInstance();

```
Time zones are implemented as part of the chronology. The code obtains a Joda-Time chronology in the Tokyo time-zone:
```

  DateTimeZone zone = DateTimeZone.forID("Asia/Tokyo");
  Chronology gregorianJuian = GJChronology.getInstance(zone);

```
##  Intervals and time periods 
An **interval** is represented by the ` Interval ` class. It** holds a start and end date-time**, and allows operations based around that range of time.
A time period is represented by the`  Period ` class. This **holds a period such as 6 months, 3 days and 7 hours**. You can create a Period directly, or derive it from an interval.
A time duration is represented by the ` Duration ` class. This **holds an exact duration in milliseconds.** You can create a Duration directly, or derive it from an interval.

For example, consider adding one day to a DateTime at the daylight savings cutover:
```

  DateTime dt = new DateTime(2005, 3, 26, 12, 0, 0, 0);
  DateTime plusPeriod = dt.plus(Period.days(1));
  DateTime plusDuration = dt.plus(new Duration(24L*60L*60L*1000L));

```
**Adding a period will add 23 hours** in this case, not 24 because of the daylight savings change, thus the time of the result will still be midday.
**Adding a duration will add 24 hours** no matter what, thus the time of the result **will change to 13:00.**


##  示列 
3、  获取现在距离今天结束还有多长时间
```

DateTimenowTime = new DateTime();
DateTime endOfDay = nowTime.millisOfDay().withMaximumValue();
endOfDay.getMillis()-nowTime.getMillis()

```
计算两个日期的相隔天数
```

DateTime nowTime = new DateTime();
DateTime futureTime = new DateTime(2015, 10, 1, 0, 0, 0);
Int days = Days.daysBetween(nowTime, futureTime).getDays();

```
传递 SimpleDateFormat 字符串
```

DateTime dateTime = SystemFactory.getClock().getDateTime();
dateTime.toString("MM/dd/yyyy hh:mm:ss.SSSa");
dateTime.toString("dd-MM-yyyy HH:mm:ss");
dateTime.toString("EEEE dd MMMM, yyyy HH:mm:ssa");
dateTime.toString("MM/dd/yyyy HH:mm ZZZZ");
dateTime.toString("MM/dd/yyyy HH:mm Z");

09/06/2009 02:30:00.000PM
06-Sep-2009 14:30:00
Sunday 06 September, 2009 14:30:00PM
09/06/2009 14:30 America/Chicago
09/06/2009 14:30 -0500

```
参考：http://www.ibm.com/developerworks/cn/java/j-jodatime.html