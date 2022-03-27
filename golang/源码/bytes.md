#### 简介

bytes包是golang中对bytes的操作的一个包。

主要包括bytes.Buffer、bytes.Reader、byte的一系列工具方法等。



#### bytes.Buffer

类似于strings.Builder，都是用作对string进行拼接的，两者的方法也类似，都有WriteRune、WriteByte、WriteString等方法。

```
buffer := bytes.NewBufferString("abc")
buffer.WriteString("def")
buffer.WriteByte('g')
buffer.WriteRune('H')
fmt.Println(buffer.String())   // abcdefgH
```



#### bytes.Reader

Reader用于读取byte数组

```
reader := bytes.NewReader([]byte("abcdefgHijklmnopq"))
// 获取长度
size := reader.Size()
fmt.Println(size)
// 弹出一个byte，会把byte从reader中去掉
byte1, _ := reader.ReadByte()
// 可以将读的byte返回去
//reader.UnreadByte()
// 如果不调用unreadbyte，这里第二次调用readbyte的结果是b
byte2, _ := reader.ReadByte()
fmt.Println(string(byte1))
fmt.Println(string(byte2))
```





bytes的其他方法其实和strings的差不多，都是类似的工具方法