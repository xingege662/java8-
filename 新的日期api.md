[TOC]
## 绪论
我们相信大多数有经验的程序员都会赞同java8之前的库对日期和时间的支持非常的不理想。然而，java8引入了全新的日期和时间API就是要解决这个问题。
## LocalDate,LocalTime,Instant,Duration以及Period
### 使用LocalDate和LocalTime
#### LocalDate
LocalDate的实例是一个不可变的对象，它只提供了简单的日期，并不含当天的时间信息。另外，它也不附带任何与市区相关的信息。
你可以通过静态工厂方法of创建一个LocalDate实例。LocalDate实例提供了多种方法来读取常用的值，比如年份，月份，星期几。
```
LocalDate date = LocalDate.of(2017,6,23);
int year = date.getYear();//2017
Month month = date.getMonth();//6
int day = date.getDayOfMonth; //23
DayOfWeek dow = date.getDayOfWeek();//Fridiay
int len = date.lenghtofMonth();//30
boolean leap = date.isLwapYear();//false
```
你还可以使用工厂方法从系统时钟中获取当前的日期

```
LocalDate today = LocalDate.now();
```
#### LocalTime

```
LocalTime time = LocalTime.of(13,45,20);
int hour = time.getHour(); //13
int minute = time.getHour(); // 45 
int second = time.getMinute();// 20
```
LocalDate和LocalTime都可以通过解析代表他们的字符串创建。使用静态方法parse，可以实现这一目的：

```
LocalDate date = LocalDate.parse("2017-03-18");
LocalTime time = LocalTime.parse("13:45:20");
```
你可以向parse方法传递一个DateTimeFormatter.该类的实例定义了如何格式化一个日期或者时间对象。正如我们之前所介绍的，他是替换老版java.util.DateFormat的推荐替代品。同时，也请注意，一旦传递的字符串参数无法被解析为合法的LocalDate和LocalTime对象，这两个parse方法都会跑出一个继承自RuntimeException的DateTimeParseException异常。

### 合并日期和时间
这个复合类叫做LocalDateTime,是LocalDate和LocalTime的合体。它同时表示了日期和时间，但不带有时区信息，你可以直接创建，也可以通过合并日期和时间对象构造。

```
LocalDateTime dt1 = LcoalDateTime.of(2017,3,18,13,45,20);
LocalDateTime dt2 = LocalDateTime.of(date,time);
LocalDateTime dt3 = date.atTime(13,45,20);
LocalDateTime dt4 = date.atTime(time);
LocalDateTime dt5 = time.atDate(date);
```
注意，通过他们各自的atTime或者atDate方法，向LocalDate传递了一个时间对象，或者向LoaclTime传递一个日期对象的方式，你可以创建一个LocalDateTime对象。你也可以使用toLocalDate或者toLocalTime方法，从LocalDateTime中提起LocalDate或者LocalTime组件：

```
LocalDate date1 = dt1.toLocalDate();
LocalDate time1 = dt1.toLocalTime();
```
### 定义Duration或者Period
我们已经了解了创建Temporal实例的几种方法。很自然地你会想到，我们需要创建两个Temporal对象之间的duration。Duration类的静态工厂方法between就是为这个目的而设计的。你可以创建两个LocalTime对象，两个LocalDateTimes对象。或者两个Instant对象之间的duration。

```
Duration d1 = Duration.between(time1,time2);
Duration d1 = Duration.between(dateTime1,dateTime2);
Duration d2 = Duration.between(instant1,instant2);
```
由于LocalDateTime和Instant是为了不同的目的而设计的，一个是为了便于人阅读使用，另一个是为了便于机器处理，所以你不能将二者混用。此外，由于Duration主要用于以秒和纳秒衡量时间的长短，你不能仅仅向between方法传递一个LcoalDate对象做参数。

如果你需要以年，月，或者日的方式对多个时间单位建模，可以使用Period类。使用该类的工厂方法between,你可以使用得到两个LocalDate之间的时长。

```
Period tenDays = Period.between((LocalDate.of(2017,3,8),LocalDate.of(2017.3.18));
```
## 操纵，解析和格式化日期
如果你已经有一个LocalDate对象，想要创建它的一个修改版，最直接也是最简单的方法是使用withAttribute方法。

```
LocalDate date1 = LcoalDate.of(2014.3.18);//2014-03-18
LocalDate date2 = date1.withYear(2011);//2011-03-18
LocalDate date3 = date2.withDayOfMonth(25);//2011-03-25
LocalDate date4 = date3.with(ChronoField.MONTH_OF_YEAR,9);//2011-09-25
```
以相对方式修改LocalDate对象的属性

```
LocalDate date1 = LocalDate.of(2014,3,18);
LocalDate date2 = date1.plusWeeks(1);//2014-03-25
LocalDate date3 = date2.minusYears(3);//2011-03-25
```
### 使用TemporalAdjuster
截止目前，你所看到的所有日期操作都是相对比较直接的。有的时候，你需要进行更加复杂的操作，比如，将日期调整到下个周日，下个工作日，或者是本月的最后一天。这时，你可以使用重载版本的with方法，向其传递一个提供了更多定制化选择的TemporalAdjuster对象，更加灵活地处理日期。

```
LocalDate date1 = LocalDate.of(2014,3,18);//2014-03-18
LocalDate date2 = date1.with(nextOrSame(DayOfWeek.SUNDAY));//2014-03-23
LocalDate date3 = date2.with(lastDayOfMonth());//2014-03-31
```
TemporalAdjuster中包含的工厂方法列表
![image](http://opy4iwqsf.bkt.clouddn.com/WX20170623-145830@2x.png)
### 打印输出及解析日期-时间对象

```
LocalDate date = LocalDate.of(2014,3,18);
String s1 = date.format(DateTimeFormatter.BASIC_ISO_DATE);//20140318
String s2 = date.format(DateTimeFormatter.ISO_LOCAL_DATE);//2014-03-18
```
你也可以通过解析代表日期或时间的字符串重新创建该日期对象。所有的日期和时间API都提供了表示时间点或者时间段的工厂方法，你可以使用工厂方法parse达到重新创建该日期对象的目的：

```
LocalDate date1 = LocalDate.parse("20140318",DateTimeFormatter.BASIC_ISO_DATE);
lcoalDate date2 = LocalDate.parse("2014-03-18",DateTimeFormatter.ISO_LOCAL_DATE);
```
和老的java.util.DateFormat相比较，所有的DateTimeFormatter实例都是线程安全的。所以，你能够以单例模式创建格式器实例，就像DateTimeFomatter所定义的那些常量，并能在多个线程间共享这些实例。DateTimeFormatter类还支持一个静态工厂方法，它可以按照某个特定的模式创建格式器。

```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date1 = LocalDate.of(2014,3,18);
String fomattedDate = date1.format(formatter);
LocalDate date 2 = LocalDate.parse(formattedDate,formatter);
```

