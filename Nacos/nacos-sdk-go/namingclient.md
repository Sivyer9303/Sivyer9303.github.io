### 简介

naming client是nacos中，负责处理和命名相关的客户端，主要用于注册、删除、修改实例，获取服务、获取实例等。

### 构成


```
type NamingClient struct {
   nacos_client.INacosClient -- 定义的一个client接口，用于多态，实现不同接入不同的client
hostReactor  HostReactor   -- host响应器，用于在instance发生变化时，进行响应
serviceProxy NamingProxy   -- 服务代理，用于发送请求
subCallback  SubscribeCallback  -- 回调
beatReactor  BeatReactor  -- 心跳响应器
indexMap     cache.ConcurrentMap  -- 已启用
NamespaceId  string  -- 命名空间id
}
```

#### hostReactor
