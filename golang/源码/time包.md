

#### 常用基本操作

- 创建时间

  时间的创建方式可以通过其构造器。

  ```
  func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time 
  ```

- 获取当前时间

  time.Now()

  创建的默认格式是2021-09-14 20:30:16.5467091 +0800 CST m=+0.006692401

- 获取年月日等基础信息

  ```
  now := time.Now()
  // 获取年、月、日、星期、天
  fmt.Println(now.Year())
  // 注意，这里获取到的月份数据类型为Month，它的基础类型是int，可以通过强转为int输出具体的月份值，一月为1
  fmt.Println(int(now.Month()))
  // 当为9月时，输出为September
  fmt.Println(now.Month())
  // 与month类似，获取到的数据类型是Weekday,基础类型也是int。
  // 这里输出为sunday、monday这种，sunday为1
  fmt.Println(now.Weekday())
  // 输出时间的对应天数，例如，9月14，输出为14
  fmt.Println(now.Day())
  // 获取对应的小时数
  fmt.Println(now.Hour())
  // 获取对应的分钟数
  fmt.Println(now.Minute())
  // 获取对应的秒数
  fmt.Println(now.Second())
  ```

- 时间间隔计算

  时间间隔计算有三种方式。三者的返回值都是Duration。Duration可以通过Minutes()、Hours()、Microseconds()、Milliseconds()、Nanoseconds()方法，转换为指定的时间单位。

  ```
  // 计算时间差
  // 计算过去的时间到现在的时间差
  fmt.Println(time.Since(now))
  // 计算当前时间到某个时间的时间差，可以是过去的
  fmt.Println(time.Until(now))
  // 计算任意两个时间的时间差
  fmt.Println(now.Sub(newTime))
  ```

- 比较两个时间

  ```
  now := time.Now()
  newTime := time.Now()
  // 计算是否比指定时间晚，当true时为比指定时间晚
  after := now.After(newTime)
  fmt.Println(after)
  // 计算是否比指定时间早，当true时为比指定时间早
  before := now.Before(newTime)
  fmt.Println(before)
  ```

- 时间的增减

  go中的时间增加可以通过Duration来实现的，想要进行增减，先要获取到对应的duration。

  ```
  now := time.Now()
  // 通过time.ParseDuration获取到指定的Duration.可以是负数
  oneHour, _ := time.ParseDuration("-1h")
  // 增加指定的Duration对应的间隔，间隔如果为负，那么效果是减小。
  oneHourAgo := now.Add(oneHour)
  fmt.Println(oneHourAgo.Format("2006/01/02 15:04:05"))
  ```

  也可以通过调用AddDate方法。

  ```
  // AddDate方法的三个参数分别是年月日，底层使用的方式是通过构造器重新创建一个Date
  date := now.AddDate(1, 1, 1)
  fmt.Println(date.Format("2006/01/02 15:04:05"))
  ```





#### Duration类的使用

​	Duration是一个底层为int64的数据类型。它存储的是以Nanosecond（纳秒）为最小单位的时间间隔。

​	

```
// time包已经预定义了一些常用的时间单位
const (
   Nanosecond  Duration = 1
   Microsecond          = 1000 * Nanosecond
   Millisecond          = 1000 * Microsecond
   Second               = 1000 * Millisecond
   Minute               = 60 * Second
   Hour                 = 60 * Minute
)
```

​	因为本身Duration是int64类型的数据，因此在进行转换时，可以直接将Duration转换为int64，然后进行计算即可，例如将Duration转换为小时

```
func (d Duration) Hours() float64 {
	hour := d / Hour
	nsec := d % Hour
	return float64(hour) + float64(nsec)/(60*60*1e9)
}
```

​	Duration的创建：

​	Duration可以通过time.ParseDuration方法将string转换为Duration。

​	例如：

​	time.ParseDuration("-1h")，其中h代表小时，其他单位如下，在time包中使用一个map来存储。

```
"ns": int64(Nanosecond),
"us": int64(Microsecond),
"µs": int64(Microsecond), // U+00B5 = micro symbol
"μs": int64(Microsecond), // U+03BC = Greek letter mu
"ms": int64(Millisecond),
"s":  int64(Second),
"m":  int64(Minute),
"h":  int64(Hour),
```





#### 时间与字符串格式转换

##### 字符串转Date

```go
dateStr := "2020/09/14 17:30:00"
parse, err2 := time.Parse("2006/01/02 15:04:05", dateStr)
// 如果要转换的字符串和指定的layout格式不一致，则会报错
if err2 != nil {
   panic(err2)
}
fmt.Println(parse)
```

##### Date转字符串

```go
now := time.Now()
fmt.Println(now)
// 按照time包自带的一些格式进行转换
format := now.Format(time.RFC1123)
fmt.Println(format)
// 自定义格式转换
format2 := now.Format("2006/01/02 15:04:05")
fmt.Println(format2)
```





#### 时区转换

获取时区

```go
// 根据时区名获取对应时区
// 例如Asia/Shanghai 、 Asia/Chongqi等
time.LoadLocation(locationName)

// 固定时区获取
time.FixedZone("CST", 8*3600) 
```

转换时间到对应时区

```go
// TimeLayout-转换格式  timeStr-时间字符串 location-时区
time.ParseInLocation(TimeLayout, timeStr, location)

now := time.Now()
var cstZone = time.FixedZone("CST", 8*3600) // 东八
// 通过time.In(location)转换时区
fmt.Println(now.In(cstZone).Format("2006-01-02 15:04:05"))
```



