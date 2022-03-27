#### 简介

​	condition用于在多个goroutine之间进行通信，用于发起信号。

​	

#### 主要方法

##### wait

​	阻塞当前goroutine等待被通知，并将当前goroutine加入到等待队列中

##### signal

​	通知等待队列中一个的goroutine

##### broatcast

​	通知等待队列中的所有goroutine



#### 示例代码

```
var count int32
var lc sync.Mutex
cond := sync.NewCond(&lc)
var wg sync.WaitGroup
for i := 0; i < 5; i++ {
   wg.Add(1)
   go func(j int) {
      defer wg.Done()
      cond.L.Lock()
      defer cond.L.Unlock()
      if count < 5 {
         cond.Wait()
      }
      fmt.Println(" wait finish , this is ", j)
   }(i)
}

go func() {
   tick := time.Tick(time.Second)
   for {
      select {
      case <-tick:
         atomic.AddInt32(&count, 1)
         break
      case <-time.After(time.Second * 10):
         return
      }
   }
}()

wg.Add(1)
go func() {
   defer wg.Done()
   for {
      if count > 5 {
         cond.L.Lock()
         cond.Signal()
         cond.L.Unlock()
      }
   }
}()

wg.Wait()
fmt.Print("main goroutine finish")
```





#### 源码阅读

cond的源码比较简单，核心原子操作都已经被封装了。。

