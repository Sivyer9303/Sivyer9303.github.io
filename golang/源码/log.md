#### 简介

log包是golang中自带，用于格式化输出日志的包。



#### 基本操作

```go
// 设置日志前缀
log.SetPrefix("prefix__")
// 设置日志样式，使用log包中使用的常量设置比较好
// 如果存在多个格式需要设置，通过|符合连接
// 本质上是通过或操作，通过每位对应每个设置的方式来设置
log.SetFlags(log.Lmicroseconds | log.Llongfile)
// 设置log的输出writer
file, _ := os.Create("log.log")
log.SetOutput(file)
// 普通输出
log.Println("hey normal")
// 严重错误输出 -- 会终止程序
//log.Fatal("oh fatal")
// panic -- 会终止程序
//log.Panic("shit its panic")
```

