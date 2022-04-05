#### 简介

在sync包中用的比较多，用来保证某个实体不被复制。因为当某个实体被复制之后，实体的值可能会被修改，导致并发安全问题。

示例：

```
type S struct {
   f1 int
   f2 *s
}

type s struct {
   name string
}

func TestNoCopy(t *testing.T) {
   mOld := S{
      f1: 0,
      f2: &s{name: "mike"},
   }
   mNew := mOld //拷贝
   mNew.f1 = 1
   mNew.f2.name = "jane"
   
   // f1因为是非指针，因此不会被修改
   fmt.Println(mOld.f1, mOld.f2) //输出：0 &{jane}
}
```

#### 基本原理

实体中存储一个额外的指针，表示源指针。例如strings.Builder.

```
// A Builder is used to efficiently build a string using Write methods.
// It minimizes memory copying. The zero value is ready to use.
// Do not copy a non-zero Builder.
type Builder struct {
   // 初始指针,如注释所说，用于防止被复制
   addr *Builder // of receiver, to detect copies by value
   buf  []byte
}
```

在Write类方法(WriteString、WriteRune等)时，会先进行copyCheck

```
func (b *Builder) copyCheck() {
   // 如果addr为空，则表示第一次对Builder进行操作，会将addr赋值为当前对象的指针
   if b.addr == nil {
      // This hack works around a failing of Go's escape analysis
      // that was causing b to escape and be heap allocated.
      // See issue 23382.
      // TODO: once issue 7921 is fixed, this should be reverted to
      // just "b.addr = b".
      b.addr = (*Builder)(noescape(unsafe.Pointer(b)))
   } else if b.addr != b {
      // 如果不为空，且两者不一致，则说明被复制了，会panic
      panic("strings: illegal use of non-zero Builder copied by value")
   }
}
```

再例如sync.Cond中的方式。

```
func (c *copyChecker) check() {
 if uintptr(*c) != uintptr(unsafe.Pointer(c)) &&
  !atomic.CompareAndSwapUintptr((*uintptr)(c), 0, uintptr(unsafe.Pointer(c))) &&
  uintptr(*c) != uintptr(unsafe.Pointer(c)) {
  panic("sync.Cond is copied")
 }
}
```

比较复杂，把表达式拆开

// 先直接检查指针是否一致，如果不相等，可能是因为为空，如果相等，则可以不用后续流程了。

uintptr(*c) != uintptr(unsafe.Pointer(c))

// 尝试将当前指针的值赋给c，如果赋值成功，CompareAndSwapUintptr会返回true，如果失败会返回false，这里预期指针原地址为0

!atomic.CompareAndSwapUintptr((*uintptr)(c), 0, uintptr(unsafe.Pointer(c)))

// 再次检查，经过前两轮的检查以后，可以确定，当前对象的指针不为空，再次判断是否相等

uintptr(*c) != uintptr(unsafe.Pointer(c))



#### go vet静态检查

因为go源码中的nocopy check都是运行时才能判断的，有时候写代码可能会不小心导致此类问题，但是问题会在运行时才会暴露，因此go官方提供了go vet静态代码检查。

goland中的安装方法：

setting -> tools->file watchers -> +  ,添加golint,go vet 是golint中的一种。



