---
title: golang标准库之time
---

本文主要介绍`golang`内置的`time`包的基本用法。

### <center>时间类型</center>

`time.Time`类型表示时间。我们可以通过`time.Now()`函数获取当前的时间对象，然后获取时间对象的年、月、日、时、分、秒等信息。eg:

```go
func timeDemo() {
	now := time.Now() //获取本地当前时间
	fmt.Printf("current time:%v\n", now)
 
	year := now.Year()     //年
	month := now.Month()   //月
	day := now.Day()       //日
	hour := now.Hour()     //小时
	minute := now.Minute() //分钟
	second := now.Second() //秒
	fmt.Printf("%d-%02d-%02d %02d:%02d:%02d\n", year, month, day, hour, minute, second)
}
```

### <center>时间戳</center>

时间戳是自`1970-01-01`至当前时间的总毫秒数。它也被称为`Unix`时间戳。

#### <font color=red>基于时间对象获取时间戳的示例代码</font>

```go
func timestampDemo() {
	now := time.Now()            //获取当前时间
	timestamp1 := now.Unix()     //时间戳
	timestamp2 := now.UnixNano() //纳秒时间戳
	fmt.Printf("current timestamp1:%v\n", timestamp1)
	fmt.Printf("current timestamp2:%v\n", timestamp2)
}
```

#### <font color=red>使用`time.Unix()`函数可以将时间戳转化为时间格式</font>

```go
func timestampDemo2(timestamp int64) {
	timeObj := time.Unix(timestamp, 0) //将时间戳转为时间格式
	fmt.Println(timeObj)
	year := timeObj.Year()     //年
	month := timeObj.Month()   //月
	day := timeObj.Day()       //日
	hour := timeObj.Hour()     //小时
	minute := timeObj.Minute() //分钟
	second := timeObj.Second() //秒
	fmt.Printf("%d-%02d-%02d %02d:%02d:%02d\n", year, month, day, hour, minute, second)
}
```

### <center>时间间隔</center>

`time.Duration`是`time`包定义的一个类型，它代表两个时间点之间经过的时间，以纳秒为单位。`time.Duration`表示一段时间的间隔，可表示的最长时间段大约是290年。

`time`包内定义的时间间隔类型的常量如下:

```go
const (
    Nanosecond  Duration = 1
    Microsecond          = 1000 * Nanosecond
    Millisecond          = 1000 * Microsecond
    Second               = 1000 * Millisecond
    Minute               = 60 * Second
    Hour                 = 60 * Minute
)
```

Eg:`time.Duration`表示1纳秒，`time.Second`表示1秒。

### <center>时间操作</center>

#### <font color=red>Add</font>

我们在日常编码过程中可能会遇到要求时间+时间间隔的需求，`golnag`的时间对象有提供`Add`方法。eg:

```go
func main() {
	now := time.Now()
	later := now.Add(24 * time.Hour) // 当前时间加1小时后的时间
	fmt.Println(later)
}
```

#### <font color=red>Sub</font>

求两个时间对象之间的差值:

```go
func (t Time) Sub(u Time) Duration
```

返回一个时间段 t-u。如果结果超出了 Duration 可以表示的最大值 / 最小值，将返回最大值 / 最小值。要获取时间点 t-d（d 为 Duration），可以使用 t.Add (-d)。

#### <font color=red>Equal</font>

```go
func (t Time) Equal(u Time) bool
```

判断两个时间是否相同，会考虑时区的影响，因此不同时区标准的时间也可以正确比较。本方法和用 t==u 不同，这种方法还会比较地点和时区信息。

#### <font color=red>Before</font>

```go
func (t Time) Before(u Time) bool
```

如果 t 代表的时间在 u 之前，返回真；否则返回假。

#### <font color=red>After</font>

```go
func (t Time) After(u Time) bool
```

如果 t 代表的时间点在 u 之后，返回真；否则返回假。

### <center>定时器</center>

使用 time.Tick（时间间隔）来设置定时器，定时器的本质是一个通道（channel）。

```go
func tickDemo() {
	ticker := time.Tick(2 * time.Second) //定义一个2秒间隔的定时器
	for i := range ticker {
		fmt.Println(i)//每2秒都会执行的任务
	}
}
```

### <center>时间格式化</center>

时间类型有一个自带的方法 Format 进行格式化，需要注意的是 Go 语言中格式化时间模板不是常见的 Y-m-d H:M:S 而是使用 Go 的诞生时间 2006 年 1 月 2 日 15 点 04 分（记忆口诀为 2006 1 2 3 4）。也许这就是技术人员的浪漫吧。

补充：如果想格式化为 12 小时方式，需指定 PM。

#### <font color=red>格式化时间(将时间对象转化为字符串)</font>

```go
func formatDemo() {
	now := time.Now()
	// 格式化的模板为Go的出生时间2006年1月2号15点04分 Mon Jan
	// 24小时制
	fmt.Println(now.Format("2006-01-02 15:04:05.000 Mon Jan"))
	// 12小时制, 上午显示AM，下午显示PM
	fmt.Println(now.Format("2006-01-02 03:04:05.000 PM Mon Jan"))
	fmt.Println(now.Format("2006/01/02 15:04"))
	fmt.Println(now.Format("15:04 2006/01/02"))
	fmt.Println(now.Format("2006/01/02"))
}
```

#### <font color=red>解析字符串格式的时间(将字符串转化为本地当前时间对象)</font>

```go
now := time.Now()
fmt.Println(now)
// 加载时区
loc, err := time.LoadLocation("Asia/Shanghai")
if err != nil {
	fmt.Println(err)
	return
}
// 按照指定时区和指定格式解析字符串时间
timeObj, err := time.ParseInLocation("2006/01/02 15:04:05", "2019/08/04 14:15:20", loc)
// 按照指定的格式解析字符串时间，解析为UTC时间
timeObj2, err := time.Parse("2006/01/02 15:04:05", "2019/08/04 14:15:20")
if err != nil {
	fmt.Println(err)
	return
}
fmt.Println(timeObj)
fmt.Println(timeObj.Sub(now))
```

### <center>time.Sleep</center>

```go
func Sleep(d Duration)
```

将当前 goroutine 暂停至少持续时间 d，持续时间为负数或零会导致睡眠立即返回。eg:

```go
n := 5
// time.Duration为强制类型转换，将int--->Duration
time.Sleep(time.Duration(n) * time.Second)
```

或者一个更常用的方式:

```go
time.Sleep(5 * time.Second)  // 数字5会自动转为Duration
```

