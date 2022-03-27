#### Strings.Builder 与bytes.Buffer

两者都能用于字符拼接,两者都拥有WriteByte、WriteString、WriteRune等方法，但是在实现上是存在区别的。

##### struct对比

```
// A Builder is used to efficiently build a string using Write methods.
// It minimizes memory copying. The zero value is ready to use.
// Do not copy a non-zero Builder.
type Builder struct {
   addr *Builder // of receiver, to detect copies by value
   buf  []byte
}


// A Buffer is a variable-sized buffer of bytes with Read and Write methods.
// The zero value for Buffer is an empty buffer ready to use.
type Buffer struct {
	buf      []byte // contents are the bytes buf[off : len(buf)]
	off      int    // read at &buf[off], write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly.
}
```

从两者的结构上来看，存储的方式都是通过[]byte来存储的,Buffer会额外存储一个readOp，用于保存上一次操作的方式

Builder会额外存储一个自身的指针，在转换为string时，会通过buf指针转换，而Buffer会使用string([]byte)的方式强转。因此在性能上Builder更优秀（官方这么写的，具体为什么，我也不清楚...)

从写入方式来看，两者的写入方式存在差别

Builder采用append的方式，将传入的值拼接到[]byte数组中，而Buffer采用copy方式。



##### 使用方式的不同

Buffer比Builder多一部分方法，多一些Read方法，因此功能上也比Builder更丰富。



#### Strings.Compare

字符串比较方法，需要传入两个字符串，本质上这个方法调用的就是string1 < string2这种方法去直接比较，只是string1 < string2方法返回的是true、false这种，而Strings.Compare返回的是0，-1,1这种

```
func Compare(a, b string) int {
   if a == b {
      return 0
   }
   if a < b {
      return -1
   }
   return +1
}
```

!!!!!!!!!官方并不推荐使用这个工具方法，直接使用< > ,= 这些原生的比较即可!!!!!!!!!!



#### Strings.Index

获取某个string在另一个string中的位置。

```
// s为原string，substr是需要获取索引的string
// 大概的思路是，根据substr的第一个字符在s中的位置i，然后比较s从i到i+
func Index(s, substr string) int 
```

其他相关方法

```
// 查找rune在s中的位置，本质上使用的还是Index方法
func IndexRune(s string, r rune) int

// 查找s中出现的最后一个substr,使用的方式是把s转换为hash？不太懂这个
func LastIndex(s, substr string) int 

// 查找chars中任意一个字符在s中的位置，采用的方式是循环s，然后通过IndexRune方法，判断chars中是否包含s的每个rune
// 其中，会根据chars的长度来选择不同的方式，如果chars长度小于8，则转换为AscII码的集合，然后循环判断
// 如果长度大于8，则循环s，然后根据IndexRune判断
func IndexAny(s, chars string) int

// 返回根据f函数返回结果判定的索引
// 首个f函数返回为true的index即为目标index。
// example:
// str := "1a2b3c4d5e6f"
//	index := strings.IndexFunc(str, func(r rune) bool {
//		if r == 'e' {
//			return true
//		}
//		return false
//	})
//	fmt.Println(index)   // 9
func IndexFunc(s string, f func(rune) bool) int
```

