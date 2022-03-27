#### 简介

strconv包中，提供了很多字符串转换的方法可以从字符串转到对应的类型，也能从对应的类型转到字符串。



#### 整形转string

方法示例:

```go
// int转string
fmt.Println(strconv.Itoa(111)) // 111
// 无符号int，根据进制转string
// 例如这里的111，在16进制下等于 16 * 6 + 15 = 6f，因此结果等于6f
fmt.Println(strconv.FormatUint(111, 16)) //6f
// 有符号整数，根据进制转string
// 例如这里的-111，在16进制下等于 -6f，其实就是和上面那个FormatUint一样，只是说这里可以有符号
fmt.Println(strconv.FormatInt(-111, 16)) // -6f
// 将转换后的结果拼接到指定的byte数组中去，这里的结果是abc6f，因为111转换为16进制是-6f，然后拼接到abc后，就是abc-6f
fmt.Println(string(strconv.AppendInt([]byte("abc"), -111, 16))) //abc-6f
// 与上面类似，只是限定了必须是无符号整数
fmt.Println(string(strconv.AppendUint([]byte("abc"), 111, 16))) //abc6f
```



底层对于转换做的优化

small方法：

```
// small returns the string for an i with 0 <= i < nSmalls.
func small(i int) string {
   if i < 10 {
      return digits[i : i+1]
   }
   return smallsString[i*2 : i*2+2]
}

// 能够使用small方法的前提是，被转换的整形必须小于nSmalls.
const nSmalls = 100

// 小于100的整形以10进制拼接而成的字符串，每个数占据两个位置。
// 疑问，既然10以下的整型都用digits了，为什么还要在smallString中包含0-9    --- 解答，其他地方也在用这个常量，因此不能单以这个方法中的使用来判定
const smallsString = "00010203040506070809" +
   "10111213141516171819" +
   "20212223242526272829" +
   "30313233343536373839" +
   "40414243444546474849" +
   "50515253545556575859" +
   "60616263646566676869" +
   "70717273747576777879" +
   "80818283848586878889" +
   "90919293949596979899"

// 对于小于10的整形，直接使用digits
const digits = "0123456789abcdefghijklmnopqrstuvwxyz"
```



#### string转整形

```go
// string 转int ,两个返回值， int ,error
fmt.Println(strconv.Atoi("111"))
// Atoi调用的底层方式是ParseInt
fmt.Println(strconv.ParseInt("111", 10, 0))
// ParseUint是最底层的方法 传入负数会报错
fmt.Println(strconv.ParseUint("-111", 10, 0))
```





#### 其他转换

其他转换与整形类似或者很简单，这里不做赘述

