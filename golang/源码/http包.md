#### 简介

http包是用于创建、维护httpServer的包。

#### 创建一个http server

```
err = http.ListenAndServe("localhost:8011", nil)
if err != nil {
   fmt.Errorf("oops.... ")
}
```

创建一个http server非常简单，直接通过http.ListenAndServe方法即可。

