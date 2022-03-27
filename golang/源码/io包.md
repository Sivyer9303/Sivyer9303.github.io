### IO.go

#### Reader

```go
// 定义了Read方法，将读取结果放置到传入的参数p中
// 返回结果n为读取的长度
type Reader interface {
   Read(p []byte) (n int, err error)
}
```

#### Writer

```go
// 定义了write方法，将p中的数据写入到对应的流中
// 返回结果n为写入长度
type Writer interface {
   Write(p []byte) (n int, err error)
}
```

#### Closer

```go
// 定义了close方法
type Closer interface {
   Close() error
}
```

#### Seeker

```go
// 定义了Seek方法，用于偏移写入或者读取的所有
type Seeker interface {
   Seek(offset int64, whence int) (int64, error)
}
```



IO包中的主要基础接口就这几个，其他的都是对于这些接口的组合

例如ReadWriter、ReadClose等等

还有几个比较独特的接口

```go
// 一般是直接调用r.Read方法,可以理解为对Reader的二次封装
type ReaderFrom interface {
   ReadFrom(r Reader) (n int64, err error)
}

// 一般是直接调用w.Write方法,可以理解为对Writer的二次封装
type WriterTo interface {
	WriteTo(w Writer) (n int64, err error)
}

// 指定位置读
type ReaderAt interface {
	ReadAt(p []byte, off int64) (n int, err error)
}

// 指定位置写
type WriterAt interface {
	WriteAt(p []byte, off int64) (n int, err error)
}

```

#### 一些比较特殊的方法

```
// writer和reader之间的数据copy
// 底层采用的方法是首先判断reader是否实现了ReaderFrom接口或者writer是否实现了WriteTo接口，如果实现了，那么直接使用对应的readFrom或者writeTo。
// 如果都不是，那么创建一个buffer数组，使用buffer数据作为中间缓冲，read写入buffer，然后write写入write，直到结束。
func Copy(dst Writer, src Reader) (written int64, err error) {
   return copyBuffer(dst, src, nil)
}

// 创建一个限制读取个数的reader
func LimitReader(r Reader, n int64) Reader { 
	return &LimitedReader{r, n} 
}


// 创建一个读取指定区间的reader
func NewSectionReader(r ReaderAt, off int64, n int64) *SectionReader {
	return &SectionReader{r, off, off, off + n}
}

// 创建一个特殊的reader，这个reader一旦读取，就会直接写入writer（方法参数中的w）
func TeeReader(r Reader, w Writer) Reader {
	return &teeReader{r, w}
}

// 地区reader中的所有内容
func ReadAll(r Reader) ([]byte, error) {
	b := make([]byte, 0, 512)
	for {
		if len(b) == cap(b) {
			// Add more capacity (let append pick how much).
			b = append(b, 0)[:len(b)]
		}
		n, err := r.Read(b[len(b):cap(b)])
		b = b[:len(b)+n]
		if err != nil {
			if err == EOF {
				err = nil
			}
			return b, err
		}
	}
}


// discard是一个特殊的writer，write方法不会做任何操作
var Discard Writer = discard{}
```





### multi.go

​	io包下的multi主要作用是整合了一个multiReader和multiWrite，

#### multiReader的read方法

```
// 整体上的思路就是一个个reader读过去，只要碰到第一个没有EOF的reader即返回
// 其中会涉及到一个multiReader包含multiReader的处理
func (mr *multiReader) Read(p []byte) (n int, err error) {
   for len(mr.readers) > 0 {
      // Optimization to flatten nested multiReaders (Issue 13558).
      if len(mr.readers) == 1 {
      	// 如果遍历完了只剩一个了,且刚好又是multiReader,那么就循环其内部reader数组中的reader
         if r, ok := mr.readers[0].(*multiReader); ok {
            mr.readers = r.readers
            continue
         }
      }
      // 读取第一个reader中的数据,如果刚好reader也是一个multiReader,那么就会进入当前方法迭代，如果不是，那么就正常读取即可
      n, err = mr.readers[0].Read(p)
      if err == EOF {
         // Use eofReader instead of nil to avoid nil panic
         // after performing flatten (Issue 18232).
         mr.readers[0] = eofReader{} // permit earlier GC
         // 去掉第一个reader
         mr.readers = mr.readers[1:]
      }
      if n > 0 || err != EOF {
         if err == EOF && len(mr.readers) > 0 {
            // Don't return EOF yet. More readers remain.
            err = nil
         }
         return
      }
   }
   return 0, EOF
}
```

#### MultiWriter的write方法

```
// multiWriter写入，找到第一个能够写入的writer，然后写入即可
// 隐式的处理了multiWriter包含multiWriter的情况
func (t *multiWriter) Write(p []byte) (n int, err error) {
   for _, w := range t.writers {
      n, err = w.Write(p)
      if err != nil {
         return
      }
      if n != len(p) {
         err = ErrShortWrite
         return
      }
   }
   return len(p), nil
}
```



#### pipe.go

​	pipe中定义了一个通过管道读和写的实体。

```go
// A PipeReader is the read half of a pipe.
// 对应的read方法是直接调用了pipe的read方法
type PipeReader struct {
   p *pipe
}

// A PipeWriter is the write half of a pipe.
// 对应的write方法是直接调用了pipe的write方法
type PipeWriter struct {
	p *pipe
}
```

与普通的Reader和Writer存在的区别主要是，写入和读取都是通过chan实现的，因此为了保证chan不会阻塞，在创建时，应该直接调用现有方法去生成一对。

```go
// 创建维护了同一个pipe的pipeReader和pipeWriter
func Pipe() (*PipeReader, *PipeWriter) {
   p := &pipe{
      wrCh: make(chan []byte),
      rdCh: make(chan int),
      done: make(chan struct{}),
   }
   return &PipeReader{p}, &PipeWriter{p}
}
```

Read和write方法

```
func (p *pipe) Read(b []byte) (n int, err error) {
   select {
   case <-p.done:
      return 0, p.readCloseError()
   default:
   }
	
   // 会一直阻塞，直到调用了pipe的write方法	
   select {
   case bw := <-p.wrCh:
   	  // 将从chan中读取的数据写入到[]byte
      nr := copy(b, bw)
      // 告知write方法写入的长度
      p.rdCh <- nr
      return nr, nil
   case <-p.done:
      return 0, p.readCloseError()
   }
}

func (p *pipe) Write(b []byte) (n int, err error) {
	select {
	case <-p.done:
		return 0, p.writeCloseError()
	default:
		p.wrMu.Lock()
		defer p.wrMu.Unlock()
	}
	
	// 至少写一次，并直到写入完成
	for once := true; once || len(b) > 0; once = false {
		select {
		case p.wrCh <- b:
			nw := <-p.rdCh
			b = b[nw:]
			n += nw
		case <-p.done:
			return n, p.writeCloseError()
		}
	}
	return n, nil
}
```

// 目前来看的话，使用场景是创建完PipeReader和PipeWriter以后，交由两个不同的goroutine，然后可以进行数据的交互，通过查看pipe_test中的用法，可以看出，确实是这么玩的





#### ioutil.go

​	ioutil提供了很多实用的工具方法。其中比较常用的主要是以下几个方法。

```go
// 读取reader中的所有内容到byte数组中
func ReadAll(r io.Reader) ([]byte, error) {
   return io.ReadAll(r)
}

// 读取文件
// ReadFile reads the file named by filename and returns the contents.
// A successful call returns err == nil, not err == EOF. Because ReadFile
// reads the whole file, it does not treat an EOF from Read as an error
// to be reported.
//
// As of Go 1.16, this function simply calls os.ReadFile.
func ReadFile(filename string) ([]byte, error) {
	return os.ReadFile(filename)
}

// 写文件
// WriteFile writes data to a file named by filename.
// If the file does not exist, WriteFile creates it with permissions perm
// (before umask); otherwise WriteFile truncates it before writing, without changing permissions.
//
// As of Go 1.16, this function simply calls os.WriteFile.
func WriteFile(filename string, data []byte, perm fs.FileMode) error {
	return os.WriteFile(filename, data, perm)
}
```

