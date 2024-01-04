# java.time包介绍

## 简介

Java 1.0 有一个Date类，事后证明它过于简单了，当 Java 1.1 引入Calendar 类之后，Date 类中的大部分方法就被弃用了。但是 Calendar 的API还不够给力，它的实例是可修改的，并且没有处理诸如闰秒这样的问题。

java.time 包包含日期、时间、时刻和持续时间的主要API。这里定义的类代表主要的日期时间概念， 包括时刻、持续时间、日期、时间、时区和时段。 它们基于ISO日历系统，即事实上世界遵循公历规则的日历。 所有的类都是不可变的和线程安全的。

## 常用类总结

| 类                | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| Instant           | 时间线上的瞬时点。                                           |
| Duration          | 基于时间的时间量，例如 '34。5秒 '。                          |
| LocalDate         | ISO-8601日历系统中没有时区的日期， 例如2007-12-03。          |
| LocalTime         | ISO-8601日历系统中没有时区的时间， 例如10:15:30。            |
| LocalDateTime     | ISO-8601日历系统中没有时区的日期时间， 例如2007-12-03T10:15:30。 |
| Period            | ISO-8601日历系统中基于日期的时间量， 如 '2年3个月4天'。      |
| MonthDay          | ISO-8601日历系统中的一个月-日，例如 12-03。                  |
| YearMonth         | ISO-8601日历系统中的年-月，例如2007-12。                     |
| ZonedDateTime     | ISO-8601日历系统中带有时区的日期时间， 例如2007-12-03t10: 15:30 + 01:00欧洲/巴黎。 |
| ZonId             | 时区ID，例如欧洲/巴黎。                                      |
| ZoneOffset        | 格林威治/UTC的时区偏移，例如 +02:00。                        |
| DateTimeFormatter | 日期时间格式化器。                                           |



## 新API的使用示例

### 时间基本概念

时刻Instant是没有时区概念的，我们所说的时间戳也是。都表示的是0时区的时间。

而当我们使用`Date`、`LocalDateTime`等对象的时候，是包含有时区的含义的，一般根据电脑默认时区显示。所以假设有一个时刻`Instant`或者时间戳要转换为`Date`、`LocalDateTime`等对象，一般是会加上时区差的时间的。

Instant到Date

```java
Date.from(instant); // 一般会自动加上8小时
```

Instant到LocalDateTime

```java
// 设置UTC时区时间(ZonedDateTime)对象后转为Instant，instant时间和原来一样，因为都是0时区
LocalDateTime.now().atZone(ZoneOffset.UTC).toInstant();
// 这转为instant，时间比原来少8小时
LocalDateTime.now().atZone(ZoneOffset.systemDefault()).toInstant();
```

为了保证时间和物理意义一样，在使用LocalDateTime转化为Instant的时候，不要使用类似如下语句

```java
// 这里指定了LocalDateTime为UTC时区，虽然得到的Instant对象和原来的LocalDateTime对象时间一样
// 但是含义已经改变，再转成Date、LocalDateTime的时候，很难确定要不要加上时区偏移时间
LocalDateTime.now().toInstant(ZoneOffset.UTC);
```

应该使用如下语句

```java
LocalDateTime.now().atZone(ZoneOffset.systemDefault()).toInstant();
```

### 获取当前时刻

`Instant`类有一个静态工厂方法`now()`会返回当前的时刻，如下所示：

```java
Instant now = Instant.now();
System.out.println("当前时刻是：" + now);

// 当前时刻是：2016-04-18T15:41:06.876Z
```

时间戳信息里同时包含了日期和时间，这和`java.util.Date`很像。实际上`Instant`类确实等同于Java8之前的`Date`类，你可以使用`Date`类和`Instant`类各自的转换方法互相转换，例如：`Date.from(Instant)` 将`Instant`转换成`java.util.Date`，`Date.toInstant()`则是将`Date`类转换成`Instant`类。

### 获取两个时刻的间隔

```java
Instant before = Instant.now();
System.out.println("开始时刻" + before);
Thread.sleep(3000);
// do something
Instant after = Instant.now();
System.out.println("结束时刻" + before);
Duration duration = Duration.between(before, after);;
System.out.println("时间间隔是：" + duration.toMillis() + "ms");

// 开始时刻2022-09-14T09:08:10.600391Z
// 结束时刻2022-09-14T09:08:13.608180200Z
// 时间间隔是：3007ms
```

### 获取今天的日期

Java8中的`LocalDate`用于表示当天日期。和`java.util.Date`不同，它只有日期，不包含时间。当你仅需要表示日期时就用这个类。

```java
LocalDate today = LocalDate.now();
System.out.println("今天的日期是：" + today);
// 今天的日期是：2022-09-09
```

上面的代码创建了当天的日期，不含时间信息。打印出的日期格式非常友好，不像老的`Date`类打印出一堆没有格式化的信息。

```java
// 今天的日期是：Fri Sep 09 16:47:22 CST 2022
```

### 获取当前的年、月、日信息

`LocalDate`类提供了获取年、月、日的快捷方法，其实例还包含很多其它的日期属性。通过调用这些方法就可以很方便的得到需要的日期信息，不用像以前一样需要依赖`java.util.Calendar`类了。

```java
LocalDate today = LocalDate.now();
int year = today.getYear();
int month = today.getMonthValue();
int day = today.getDayOfMonth();
System.out.printf("当前日期：%d年 %d月 %d日%n", year, month, day);

// 当前日期：2022年9月9日
```

在Java8中得到年、月、日信息是这么简单直观，想用就用，没什么需要记的。对比看看以前Java是怎么处理年月日信息。

```java
Calendar calendar = Calendar.getInstance();
int year = calendar.get(Calendar.YEAR);
int month = calendar.get(Calendar.MONDAY);
int day = calendar.get(Calendar.DAY_OF_MONTH);
System.out.printf("当前日期：%d年 %d月 %d日%n", year, month + 1, day);

// 当前日期：2022年9月9日
```

### 检查闰年

`LocalDate`类有一个很实用的方法`isLeapYear()`判断该实例是否是一个闰年（能被4整除且不能被100整除 或者 能被400整除）。

```java
LocalDate today = LocalDate.now();
// LocalDate today = LocalDateTime.now().toLocalDate(); // LocalDateTime转换为LocalDate
if (today.isLeapYear()) {
    System.out.println("今年是闰年！");
} else {
    System.out.println("今年不是闰年！");
}

// 今年不是闰年！
```

### 计算两个日期之间的天数

有一个常见日期操作是计算两个日期之间的天数、周数或月数。在Java8中可以用`java.time.Period`类来做计算。下面这个例子中，我们计算了当天和将来某一天之间的月数。

```java
LocalDate now = LocalDate.of(2022, Month.SEPTEMBER, 10);;
LocalDate newYear = LocalDate.of(2023, 1, 1);
long days = now.until(newYear, ChronoUnit.DAYS);
// long weeks = now.until(newYear, ChronoUnit.WEEKS);
System.out.println("新年倒计时：" + days + "天");

// 新年倒计时：113天
```

从上面可以看到从2022年9月10日到2023年1月1日，中间共有113天。

### 获取特定日期/时间

在第一个例子里，我们通过静态工厂方法`now()`非常容易地创建了当天日期，我们还可以调用另一个有用的工厂方法`LocalDate.of()`创建任意日期，该方法需要传入年、月、日做参数，返回对应的`LocalDate`实例。这个方法没犯老`API`的设计错误，比如年度起始于1900，月份是从0开始等等。日期所见即所得，就像下面这个例子表示了9月10日。

```java
LocalDate date = LocalDate.of(2022, 9, 10);
System.out.println(date);
// 2022-09-10

LocalDateTime dateTime = LocalDateTime.of(2022, 9, 10, 10, 32, 15);
System.out.println(dateTime);
// 2022-09-10T10:32:15

```

使用Date方式

```java
Date date = new Date(2022 - 1900, 9 - 1, 10, 10, 32, 15);
System.out.println(date);

// Sat Sep 10 10:32:15 CST 2022
```

Calendar方式

```java
Calendar calendar = Calendar.getInstance();
calendar.set(Calendar.YEAR, 2022);
calendar.set(Calendar.MONTH, 9 - 1);
calendar.set(Calendar.DAY_OF_MONTH, 10);
System.out.println(calendar.getTime());

// Sat Sep 10 10:28:04 CST 2022
```

### 判断两个日期是否相等

现实生活中有一类时间处理就是判断两个日期是否相等。我们常常会检查今天是不是个特殊的日子，比如生日、纪念日或者某个节日。这时就需要把指定的日期与某个特定日期做比较，例如判断这一天是否是假期。下面这个例子会帮助你用`Java8`的方式去解决，`LocalDate`重写了`equal`方法，请看下面的例子：

```java
LocalDate today = LocalDate.now();
LocalDate date1 = LocalDate.of(2022, 9, 10);
if (today.equals(date1)) {
    System.out.printf("两个日期是同一天");
}
/**
if (today.isBefore(date1)) {
    System.out.printf("今天早于给定日期");
}
if (today.isAfter(date1)){
    System.out.printf("今天晚于给定日期");
}
**/
```

如果使用`Date`类来实现就会比较麻烦。

### 检查像生日这种周期性事件

`Java`中另一个日期时间的处理就是检查类似每月账单、生日、保险缴费日这些周期性事件。`Java`中如何检查这些节日或其它周期性事件呢？

答案是使用`MonthDay`类。这个类组合了月份和日，去掉了年，这意味着你可以用它判断每年都会发生事件。和这个类相似的还有一个`YearMonth`类。下面我们通过`MonthDay`来检查周期性事件：

```java
LocalDate today = LocalDate.now();
MonthDay birthday = MonthDay.of(6, 18);
MonthDay currentMonthDay = MonthDay.from(today);

if (currentMonthDay.equals(birthday)){
    System.out.println("今天是您的生日!!");
} else {
    System.out.println("对不起，今天不是您的生日!!");
}
```

只要当天的日期和生日匹配，无论是哪一年都会打印出祝贺信息。

### 获取当前时间

与`Java8`获取日期的例子很像，获取时间使用的是`LocalTime`类，一个只有时间没有日期的`LocalDate`的近亲。可以调用静态工厂方法`now()`来获取当前时间。

```java
LocalTime time = LocalTime.now();
System.out.println("当前时间是:" + time);

// 11:01:22.570946600
```

可以看到当前时间就只包含时间信息，没有日期。

### 如何在现有的时间上增加小时

通过增加小时、分、秒来计算将来的时间很常见。`Java8`除了不变类型和线程安全的好处之外，还提供了更好用的方法`plusHours()`。注意，这些方法返回一个全新的`LocalTime`实例，由于其不可变性，返回后一定要用变量赋值。

```java
LocalTime time = LocalTime.now();
LocalTime newTime = time.plusHours(2); // 添加两小时
// LocalTime newTime = time.plus(2, ChronoUnit.HOURS);
System.out.println("当前时间:" + time + ", 两小时后的时间: " +  newTime);

// 当前时间:10:52:38.707702900, 两小时后的时间: 12:52:38.707702900
```

可以看到，新的时间在当前时间 10:52:38 的基础上增加了2个小时。

### 计算未来或过去的日期时间

LocalDateTime 可以通过`plus()`方法增加时间，通过`minus()`减少时间。使用这两个方法可以计算未来和过去的时间。

```java
LocalDateTime localDateTime = LocalDateTime.of(2022, 10, 1, 12, 30);
LocalDateTime futureYear = localDateTime.plus(2, ChronoUnit.YEARS); // 增加两年
LocalDateTime lastYear = localDateTime.minusYears(1); // 减少一年
LocalDateTime nextWeek = localDateTime.plusWeeks(1); // 增加一周
System.out.println("两年后：" + futureYear);
System.out.println("一年前：" + lastYear);
System.out.println("下一周：" + nextWeek);

// 两年后：2024-10-01T12:30
// 一年前：2021-10-01T12:30
// 下一周：2022-10-08T12:30
```

例子结果中得到了三个日期。

### 日期时间的比较

另一个工作中常见的操作就是如何判断给定的一个日期是大于某天还是小于某天？在Java8中，`LocalDateTime`类有两类方法`isBefore()`和`isAfter()`用于比较日期。调用`isBefore()`方法时，如果给定日期小于当前日期则返回`true`。

```java
LocalDateTime now = LocalDateTime.now();

LocalDateTime tomorrow = now.plusDays(1);
if (tomorrow.isAfter(now)) {
    System.out.println("明天晚于今天！");
}
// 明天晚于今天！

LocalDateTime yesterday = now.minus(1, ChronoUnit.DAYS);
if (yesterday.isBefore(now)) {
    System.out.println("昨天先于今天！");
}
// 昨天先于今天！
```

在Java 8中比较日期非常方便，不需要使用额外的`Calendar`类来做这些基础工作了。

### 包含时差信息的日期和时间

在Java8中，`ZoneOffset`类用来表示时区，举例来说印度与UTC标准时区相差`+05:30`，可以通过`ZoneOffset.of()`静态方法来获取对应的时区。一旦得到了时差就可以通过传入`LocalDateTime`和`ZoneOffset`来创建一个`OffSetDateTime`对象。

```java
LocalDateTime datetime = LocalDateTime.of(2022, Month.MARCH, 15, 12, 30);
ZoneOffset offset = ZoneOffset.of("+08:00");
OffsetDateTime offsetDateTime = OffsetDateTime.of(datetime, offset);
System.out.println("包含时差信息的日期和时间 : " + offsetDateTime);
System.out.println("比零时区快了" + offsetDateTime.getOffset().getTotalSeconds() + "秒");

// 包含时差信息的日期和时间 : 2022-03-15T12:30+08:00
// 比零时区快了28800秒
```

现在的时间信息里已经包含了时区信息了。注意：`OffSetDateTime`是对计算机友好的，`ZoneDateTime`则对人更友好。

### xxxxxxxxxx Object currentProxy();java

`Java8`不仅分离了日期和时间，也把时区分离出来了。现在有一系列单独的类如`ZoneId`来处理特定时区，`ZoneDateTime`类来表示某时区下的时间。这在Java8以前都是`GregorianCalendar`类来做的。下面这个例子展示了如何把本时区的时间转换成另一个时区的时间。

```java
// 某时区下的日期和时间
ZoneId america = ZoneId.of("America/New_York"); // 获取时区
LocalDateTime localDateTime = LocalDateTime.now();
ZonedDateTime dateTimeInNewYork = ZonedDateTime.of(localDateTime, america);
System.out.println("特定时区下的当前日期时间: " + dateTimeInNewYork);

// 特定时区下的当前日期时间: 2022-09-14T15:51:14.766360700-04:00[America/New_York]
```

### 时区转换

```java
ZonedDateTime now = ZonedDateTime.now();
System.out.println(now);
// 相对本地（我们现在位置）而言现在UTC时间
ZonedDateTime utcNow = ZonedDateTime.ofInstant(now.toInstant(), ZoneId.of("UTC"));
System.out.println(utcNow);
// 相对本地而言现在美国时间
ZonedDateTime americaNow = ZonedDateTime.ofInstant(now.toInstant(), ZoneId.of("America/New_York"));
System.out.println(americaNow);

// 下面仅仅是转换时区，时间不变
// ZonedDateTime zeroDateTime = ZonedDateTime.of(now, ZoneId.of("UTC"));

// 2022-09-09T15:49:29.485342800+08:00[Asia/Shanghai]
// 2022-09-09T07:49:29.485342800Z[UTC]
// 2022-09-09T03:49:29.485342800-04:00[America/New_York]
```

```java
// 查看所有可用的时区
ZoneId.getAvailableZoneIds().forEach(System.out::println);
```

### 如何表示信用卡到期这类固定日期，答案就在YearMonth类

与`MonthDay`检查重复事件的例子相似，`YearMonth`是另一个组合类，可以用于表示信用卡到期日等。还可以用这个类得到当月共有多少天，`YearMonth`实例的`lengthOfMonth()`方法可以返回当月的天数，在判断2月有28天还是29天时很有用。

```java
YearMonth thisMonth = YearMonth.now();
System.out.println("这个月的天数是：" + thisMonth.lengthOfMonth());
// 这个月的天数是：30

YearMonth creditCardExpiry = YearMonth.of(2024, Month.FEBRUARY);
System.out.println("您的信用卡到期是：" + creditCardExpiry);
// 您的信用卡到期是：2024-02
```

### 日期调整器

对于日常安排来说，经常需要计算诸如 “每个月的第一个星期二” ，或者父亲节（每年6月份的第三个星期天）这样的日期。TemporaAdjusters类提供了大量用于常见调整的静态方法。只需要将调整方法的结果传递给with方法即可。如某个月的第一个星期二可以像下面这样这样计算：

```java
LocalDate firstTuesday = LocalDate.of(year, month, 1).with(
	TemporalAdjusters.nextOrSame(DayOfWeek.TUESDAY));
```

还可以通过实现TemporalAdjuster接口来创建自己的调整器。如下代码获取下一个星期天。

```java
LocalDate myNextdSunday = now.with(w -> { // 参数w为当前日期
    LocalDate result = (LocalDate) w;
    // 不断加1天，直到那天是星期天
    do {
        result = result.plusDays(1L);
    } while (result.getDayOfWeek() != DayOfWeek.SUNDAY);
    return result;
});
```

## 使用DateFormatter进行格式和解析

在`Java8`以前的世界里，日期和时间的格式化非常诡异，唯一的帮助类`SimpleDateFormat`也是非线程安全的，而且用作局部变量解析和格式化日期时显得很笨重，不过这都是过去式了。Java8 引入了全新的日期时间格式工具 `DateTimeFormatter` ，线程安全而且使用方便。

DateTimeFormatter 类提供了三种用于打印日期/时间值的格式化器。

1. 预定义的格式器
2. locale 相关的格式器
3. 带有定制模式的格式器

### 预定义的格式化器

要使用标准的格式器，可以直接调用其 format 方法和 parse 方法即可。

```java
LocalDateTime now = LocalDateTime.now();
String formatDate = DateTimeFormatter.ISO_WEEK_DATE.format(now);
System.out.println("格式化日期：" + formatDate);

String dateStr = "2022-W37-4";
LocalDate date = LocalDate.parse(dateStr, DateTimeFormatter.ISO_WEEK_DATE);
// 使用以下解析代码代码会抛出异常，格式化日期缺少时间信息
// LocalDateTime dateTime = LocalDateTime.parse(dateStr, DateTimeFormatter.ISO_WEEK_DATE);
System.out.println("解析后日期：" + date);

// 格式化日期：2022-W37-4
// 解析后日期：2022-09-15
```

完整的预定义格式器可参见下表。

| 格式器                                                 | 描述                                                         | 示例                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| BASIC_ISO_DATE                                         | 年、月、日、时区偏移量                                       | 20111203+0800                                                |
| ISO_LOCAL_DATE ，ISO_LOCAL_TIME，ISO_LOCAL_DATE_TIME   | 分隔符为-、：、T                                             | 2011-12-03，10:15:30，2011-12-03T10:15:30                    |
| ISO_OFFSET_DATE，ISO_OFFSET_TIME，ISO_OFFSET_DATE_TIME | 类似ISO_LOCAL_XXX，但是有时区偏移量                          | 2011-12-03+08:00，10:15:30+08:00，2011-12-03T10:15:30+08:00  |
| ISO_ZONED_DATE_TIME                                    | 带有时区偏移量和时区ID                                       | 2011-12-03T10:15:30+01:00[Europe/Paris]                      |
| ISO_INSTANT                                            | 在UTC中，用Z时区ID来表示                                     | 2011-12-03T10:15:30Z                                         |
| ISO_DATE，ISO_TIME，ISO_DATE_TIME                      | 类似ISO_OFFSET_DATE，ISO_OFFSET_TIME和ISO_ZONED_DATE_TIME，但是时区日期是可选的 | 2011-12-03，10:15:30，2011-12-03T10:15:30+01:00[Europe/Paris] |
| ISO_ORDINAL_DATE                                       | 年和当前年的第几日                                           | 2012-337                                                     |
| ISO_WEEK_DATE                                          | 年和当前年的第几周、周几                                     | 2012-W48-6                                                   |
| RFC_1123_DATE_TIME                                     | 用于邮件时间戳的标准                                         | Tue, 3 Jun 2008 11:05:30 GMT                                 |

### 使用带 locale 的格式化器

使用 locale 相关的格式化器，可以实现日期时间国际化，同时还可以通过`withZone()`方法设置时区。

对于时间和日期而言，有4种与locale相关的格式化风格，即 SHORT、MEDIUM、LONG、FULL，格式如下表所示：

| 风格   | 日期                | 时间         |
| ------ | ------------------- | ------------ |
| SHORT  | 2022/9/15           | 上午11:05    |
| MEDIUM | 2022年9月15日       | 上午11:05:35 |
| LONG   | 2022年9月15日       | 上午11:05:35 |
| FULL   | 2022年9月15日星期四 | 上午11:05:35 |

格式化日期时间代码示例：

```java
// 获取不同的 Locale
Locale america = Locale.US;
Locale korea = Locale.KOREA;
ZonedDateTime now = ZonedDateTime.now();
// 获取可以带 Locale 的格式化器
DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);

// 使用不同的 locale 格式化当前时间，没有指定则使用默认 locale
String chinaTime = formatter.format(now);
String americaTime = formatter.withLocale(america).withZone(ZoneId.of("America/New_York")).format(now);
String koreaTime = formatter.withLocale(korea).format(now);

System.out.println("中国时间：" + chinaTime);
System.out.println("美国时间：" + americaTime);
System.out.println("韩国时间：" + koreaTime);

// 中国时间：2022年9月15日 CST 上午10:56:28
// 美国时间：September 14, 2022 at 10:56:28 PM EDT
// 韩国时间：2022년 9월 15일 오전 10시 56분 28초 CST
```

解析日期时间字符串：

```java
String dateTimeStr = "2022年9月15日 CST 上午10:56:28";
DateTimeFormatter formatter = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
LocalDateTime dateTime = LocalDateTime.parse(dateTimeStr, formatter);
System.out.println("解析的日期时间是：" + dateTime);

// 解析的日期时间是：2022-09-15T10:56:28
```

注意：如果字符串日期时间风格与格式化器风格不一致，将会抛出异常。

### 使用带有定制模式的格式化器

尽管内置格式化工具很好用，但有时还是需要定义特定的日期格式，下面这个例子展示了如何创建自定义日期格式化工具。可以调用`DateTimeFormatter`的`ofPattern()`静态方法并传入任意格式返回其实例，格式中的字符和以前代表的一样，M代表月，m代表分。如果格式不规范会抛出`DateTimeParseException`异常。

```java
LocalDateTime now = LocalDateTime.now();
// 根据自定义模式获取格式化器
DateTimeFormatter commonFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
DateTimeFormatter specialFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd a KK时 mm分 ss秒");

// 格式化
String specialDateTime = specialFormatter.format(now);
String commonDateTime = commonFormatter.format(now);

System.out.println("普通格式：" + commonDateTime);
System.out.println("特殊格式：" + specialDateTime);

// 解析
String dateTimeStr = "2022-09-15 下午 03时 27分 06秒";
LocalDateTime dateTime = LocalDateTime.parse(dateTimeStr, specialFormatter);
System.out.println("解析日期：" + dateTime);

// 普通格式：2022-09-15 15:29:10
// 特殊格式：2022-09-15 下午 03时 29分 10秒

// 解析日期：2022-09-15T15:27:06
```

定制模式中字母的含义

| 符号 | 含义                       | 展示        | 示例                                           |
| ---- | -------------------------- | ----------- | ---------------------------------------------- |
| G    | era                        | text        | AD; Anno Domini; A                             |
| u    | year                       | year        | 2004; 04                                       |
| y    | year-of-era                | year        | 2004; 04                                       |
| D    | day-of-year                | number      | 189                                            |
| M/L  | month-of-year              | number/text | 7; 07; Jul; July; J                            |
| d    | day-of-month               | number      | 10                                             |
|      |                            |             |                                                |
| Q/q  | quarter-of-year            | number/text | 3; 03; Q3; 3rd quarter                         |
| Y    | week-based-year            | year        | 1996; 96                                       |
| w    | week-of-week-based-year    | number      | 27                                             |
| W    | week-of-month              | number      | 4                                              |
| E    | day-of-week                | text        | Tue; Tuesday; T                                |
| e/c  | localized day-of-week      | number/text | 2; 02; Tue; Tuesday; T                         |
| F    | week-of-month              | number      | 3                                              |
|      |                            |             |                                                |
| a    | am-pm-of-day               | text        | PM                                             |
| h    | clock-hour-of-am-pm (1-12) | number      | 12                                             |
| K    | hour-of-am-pm (0-11)       | number      | 0                                              |
| k    | clock-hour-of-am-pm (1-24) | number      | 0                                              |
|      |                            |             |                                                |
| H    | hour-of-day (0-23)         | number      | 0                                              |
| m    | minute-of-hour             | number      | 30                                             |
| s    | second-of-minute           | number      | 55                                             |
| S    | fraction-of-second         | fraction    | 978                                            |
| A    | milli-of-day               | number      | 1234                                           |
| n    | nano-of-second             | number      | 987654321                                      |
| N    | nano-of-day                | number      | 1234000000                                     |
|      |                            |             |                                                |
| V    | time-zone ID               | zone-id     | America/Los_Angeles; Z; -08:30                 |
| z    | time-zone name             | zone-name   | Pacific Standard Time; PST                     |
| O    | localized zone-offset      | offset-O    | GMT+8; GMT+08:00; UTC-08:00;                   |
| X    | zone-offset 'Z' for zero   | offset-X    | Z; -08; -0830; -08:30; -083015; -08:30:15;     |
| x    | zone-offset                | offset-x    | +0000; -08; -0830; -08:30; -083015; -08:30:15; |
| Z    | zone-offset                | offset-Z    | +0000; -0800; -08:00;                          |



### 与遗留代码的互操作

作为全新的创造，Java Date 和 Time API必须能够与已有类之间进行互操作，特别是无处不在的 java.util.Date、GregorianCalendar和java.sql.Date/Time/Timestamp。

| 类                                           | 转换到遗留类                         | 转换自遗留类                |
| -------------------------------------------- | ------------------------------------ | --------------------------- |
| Instant - java.util.Date                     | Date.from(instant)                   | date.toInstant()            |
| ZonedDateTime - java.util.GregorianClalendar | GregorianCalenar.from(zonedDateTime) | cal.toZoneDateTime()        |
| Instant - java.sql.Timestamp                 | TimeStamp.from(instant)              | timestamp.toInstant()       |
| LocalDateTime - java.sql.Timestamp           | TimeStamp.valueOf(localDateTime)     | timeStamp.toLocalDateTime() |
| LocalDate - java.sql.Date                    | Date.valueOf(localDate)              | date.toLocalDate()          |
| LocalTime - java.sql.Time                    | Date.valueOf(localTime)              | time.toLocalTime()          |
| DateTimeFormatter - java.text.DateFormat     | formatter.toFormat()                 | --                          |
| ZonedId - java.util.TimeZone                 | TimeZone.getTimeZone(id)             | timeZone.toZoneId()         |
| Instant - java.nio.file.attribute.FileTime   | FileTime.from(instance)              | fileTime.toInstance()       |



## 常用API

> 方法后面出现的数字，表示的是支持该方法的最低 JDK 版本

### java.time.Instant

- static Instatant now()

  从最佳的可用系统时钟中获取当前的时刻。

- Instant plus(TemporalAmount amountToAdd)

- Instant minus(TemporalAmout amountToSubstract)

  产生一个时刻，该时刻与当前时刻距离给定的实时间量。Duration 和 Period 实现了 TemporalAmount 接口。

- Instant (plus|minus)(Nanos|Millis|Seconds)(long Number)

  产生一个时刻，该时刻与当前时刻距离给定数量的纳秒、微妙或秒。

- long toEpochMilli()

  返回从新纪元到当前所包含的毫秒数。

### java.time.Duration

- static Duration of(Nanos | Millis | seconds | Minutes l Hours | Days)(long number)
  产生一个给定数量的指定时间单位的时间间隔。

- static Duration between(Temporal startInclusive, Temporal endExclusive)
  应生一个在给定时间点之问的 ouration对象。 Instant 类实现了 Temporal 按口， LocalDate/
  LocalTine/LocalDateTime 和 ZonedDateTime也实现了该接口。

- long toNanos()

- long toMillis()

- long toSeconds () 9

- long toMinutes()

- long toHours ()

- long toSeconds ()

- long toSeconds()

- long toDays ()
  获取当前时长按照方法名中的时间单位度量的数量

- int to (Nanos Millis Seconds Minutes Hours)Part() 9

- long to(Days Hours |Minutes | Seconds | Millis | Nanos)Part() 9
  当前时长中给定时间单位的部分。例如，在100 秒的时间间隔中，分钟的部分是 1，
  秒的部分是 40。

- Instant plus (TemporalAmount amountToAdd)

- Instant minus (TemporalAmount amountToSubtract) luo
  产生一个时刻，该时刻与当前时刻距离给定的时间量。Duration 和 Period 类实现了 TemporalAmount 接口。

- Duration multipliedBy(long multiplicand)
  Duration dividedBy(long divisor)

- Duration negated ()
  产生一个时长，该时长是通过当前时刻乘以或除以给定的量或 -1 得到的。

- boolean isZero()

- boolean isNegative()

  如果当前Duration对象是0或负数，则返回true。

- Duration (plus | minus)(Nanos | Millis | seconds | Minutes l Hours | Days)(long number)

  产生一个时长，该时长是通过当前时刻加上或减去给定的数量的指定时间单位而得到的。

### java.time.LocalDate

- static LocalDate now ()
  获取当前的 LocalDate。

- static LocalDate of (int year, int month, int dayOfMonth)

- static LocalDate of (int year, Month month, int dayOfMonth)
  用给定的年、月（1到12之间的整数或者Month枚举的值）和日（1到31之间）产生一个本地日期。

- LocalDate (plus l minus)(Days l weeks Months Years )(long number)
  产生一个 LocalDate，该对象是通过在当前对象上加上或减去给定数量的时间单位获得的。

- LocalDate plus (TemporalAmount amountToAdd)

- LocalDate minus (TemporalAmount amountToSubtract)
  产生一个时刻，该时刻与当前时刻距离给定的时间量。Duration 和 Period 类实现了 TemporalAmount 接口。

- LocalDate withDayofMonth(int dayofMonth)

- LocalDate withDay0fYear(int dayOfYear)

- LocalDate withMonth(int month)

- LocalDate withYear (int year)
  返回一个新的 LocalDate，将月份日期、年日期、月或年修改为给定值。

- int getDayOfMonth()
  获取月份日期（1到 31之间）。

- int getDayofYear()
  获取年日期（1到366之间）。

- DayofWeek getDay OfWeek()
  荻取星期日期，返回某个 Dayofweek 枚举值。

- Month getMonth()

- int getMonthValue ( )
  获取用Month 枚举值表示的月份，或者用 1到12之间的数字表示的月份。

- int getYear()
  获取年份，在-999,999,999到 999,999,999之间

- Period until (ChronoLocalDate endDateExclusive)
  获取直到给定终止日期的period。 LocalDate 和 date 类针对非公历实现了 ChronolocalDate
  接口。

- boolean isBefore (ChronoLocalDate other)

- boolean isAfter (ChronoLocalDate other)
  如果该日期在给定日期之前或之后，则返回 true。

- boolean isLeapYear()
  如果当前是国年，则返回 true。即，该年份能够被4整除，但是不能被 100 签除，或者能够被 400整除。该算法应该可以应用于所有已经过去的年份，尽管在历史上它并不淮确（国年是在公元前 46 年发明出來的，而油及整除 100 和 400 的规则是在 1582年的公历改革中引的。这场改革经历丁 300年才被广泛接受）。

- StreansLocalDates datesUntil(LocalDate endExclusive) 9

- StreaneLocalDates datesUntil(LocalDate endExclusive, Period step) 9
  产生一个日期流，以当的的 LocalDate 对象直至参数 enuEsxelusive 指定的日期，其中步长尺寸为1，或者是给定的 period。

- LocalDate with(TemporalAdjuster adjuster)

  返回该日期通过给定的调整器调整后的结果。

- static LocalDate parse (CharSequence text)

- static LocalDate parse (CharSequence text, DateTimeFormatter formatter)

  用默认的格式器或给定的格式器产生一个 LocalDate。

### java.time.LocalTime

- static LocalTime now( )
  获取当前的 LocalTime。
- static LocalTime of(int hour, int minute)
- static LocalTime of(int hour, int minute, int second)
- static LocalTime offint hour, int minute, int second, int nanolfSecond)
  产生一个LralTime，它具有给定的小时 （0到23之问）、分钟、秒（0到 s9之问)和
  纳秒（0到 999,999,999之间）。
  LocalTime (plus | minus) (Hours |Minutes | Seconds | Nanos) (long number)
  产生一个LocalTine，该对象是通过在当前对象上加上或减去给定数量的时间单位获得的
- LocalTime plus (TemporalAmount amountToAdd)
- LocalTime minus (TemporalAmount amountToSubtract)
  产生一个时刻，该时刻与当前时刻距离给定的时间量。
- LocalTime with(Hour l Minute l Second Nano) (int value)
  返回一个新的 LocalTime， 将小时、分钟、秒或纳秒修改为给定值。
- int getHour()
  获取小时（0到23之间）。
- int getMinute()
- int getSecond ()
  获取分钟或秒（0到59之间）。
- int getNano ()
  获取纳秒（0到999,999,999之间）。
- int toSecondOfDay ()
- long toNanoOfDay()
  产生自午夜到当前 LocalTime 的秒或纳秒数。 00
- boolean isBefore (LocalTime other)
- boolean isAfter (LocalTime other)
  如果当期日期在给定日期之前或之后，则返回 true。

### java.time.Period

用给定数量的时间单位产生一个Period 对象。

- int get (Days | Months | Years) ()
  获取当前 Period 对象的日、月或年。
  Period (plus|minus) (Days | Months | Years) (long number)
  产生一个Loelbate，该对象是通过在当前对系上加上或减去给定数量的时回单位我稱的。
- Period plus (TemporalAmount amountToAdd)
- Period minus (TemporalAmount amountToSubtract)
  产生一个时刻，该时刻与当前时刻距旁给定的时间量。Duration 和 Period炎实现了 TemporalAmount 接口。
- Period with(Days | Months | Years) (int number)
  返回一个新的 Period，将日、月、年修改为给定值。

### java.time.temporal.TemporalAdjusters

-  static TemporalAdjuster next (DayOfweek dayOfWeek)
- static TemporalAdjuster nextorSame (DayOfweek dayOfWeek)
- static TemporalAdjuster previous (DayOfWeek  dayOfWeek)
- static TemporalAdjuster previousOrSame (DayOfweek dayOfWeek)
  返回一个调整器，用于将日期调整为给定的星期日期
- static TemporalAdjuster dayOfWeekInMonth(int n, DayOfWeek dayOfWeek)
- static TemporalAdjuster lastInMonth (DayOfWeek dayOfWeek)
  返回_个调整器，用于将日期调整为月份中第n个或最后上个给定的星期目期。
- static TemporalAdjuster firstDayOfMonth()
- static TemporalAdjuster firstDayOfNextMonth()
- static TemporalAdjuster firstDayOfYear()
- static TemporalAdjuster firstDayOfNextYear()
- static TemporalAdjuster lastDayOfMonth()
- static TemporalAdjuster lastDayOfYear()
  返回一个调整器，用于将日期调整为月份或年份中给定的日期。

### java.time.ZonedDateTime

- int getDayOfMonth()
  获取月份日期（1到31之间）。

- int getDayOfYear()
  获取年份日期（1到 366之间）。

- DayofWeek getDayOfWeek()
  获取星期日期，返回 DayofWeek枚举的值。

- Month getMonth()

- int getMonthValue()
  获取用 Month 枚举值表示的月份，或者用1 到12 之间的数宇表示的月份。

- int getYear()
  获取年份，在 -999 999 999到 999 999 999 之间.

- int getHour()
  获取小时（0到23之间）。

- int getMinute()

- int getSecond()
  获取分钟到秒（0到59之间）。

- int getNano ()
  获取纳秒（0到999 999 999之间）。

- public ZoneOffset getoffset()
  获取与UTC 的时间差距。差距可在-12:00~+14:00 变化。有些时区还有小数时间差。时间差会随着夏令时变化。

- LocalDate toLocalDate()

- LocalTime toLocalTime()

- LocalDateTime toLocalDateTime()

- Instant toInstant()
  生成当地日期、时间，或日期/时间，或相应的瞬间。

- boolean isBefore(ChronoZonedDateTime other)

- boolean isAfter(ChronoZonedDateTime other)
  如果这个时区 日期/时问在给定的时区日期，时间之前或之后，则返回 true。

- static ZonedDateTime parse(CharSequence text)

- static ZonedDateTime parse(CharSequence text, DateTimeFormatter formatter)

  用默认的格式器或给定的格式器产生一个 LocalDate。

### java.time.format.DateTimeFormatter

- String format (TemporalAccessor temporal)
  格式化给定值。Instant、LocalDate、LocaLTime 、LocalDateTime 和 ZonedDateTine，以及许多其他类，都实现了 TemporalAccessor 接口。

- static DateTineFormatter ofLocalizedDate(FormatStyle datestyle)

- static DateTimeFormatter ofLocalizedTime(Formatstyle timestyle)

- static DateTimeFormatter ofLocalizedDateTime(FormatStyle dateTimeStyle)

- static DateTimeFormatter ofLocalizedDateTime(FormatStyle dateStyle, Formatstyle timeStyle)
  产生一个用于给定风格的格式器。 Fommatstyle枚举的值包括 SHORT、 MEDIUM、 LONG 和 FULL.

- DateTimeFormatter withLocale(Locale locale)
  用给定的地点产生一个等价于当前格式器的格式器。

- static DateTimeFormatter ofPattern(String pattern)

- static DateTimeFormatter ofPattern(String pattern, Locale locale)

  用给定的模式和地点产生一个格式器。













