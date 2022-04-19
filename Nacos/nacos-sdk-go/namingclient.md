### 简介

naming client是nacos中，负责处理和命名相关的客户端，主要用于注册、删除、修改实例，获取服务、获取实例等。

### 接口

```go
type INamingClient interface {

   // 注册实例
   RegisterInstance(param vo.RegisterInstanceParam) (bool, error)

   // 注销实例
   DeregisterInstance(param vo.DeregisterInstanceParam) (bool, error)

   // 更新实例
   UpdateInstance(param vo.UpdateInstanceParam) (bool, error)

   // 获取服务
   GetService(param vo.GetServiceParam) (model.Service, error)

   // 查询所有实例
   SelectAllInstances(param vo.SelectAllInstancesParam) ([]model.Instance, error)

   // 根据条件获取实例
   SelectInstances(param vo.SelectInstancesParam) ([]model.Instance, error)

   // 获取一个健康的实例
   SelectOneHealthyInstance(param vo.SelectOneHealthInstanceParam) (*model.Instance, error)

   // 订阅
   Subscribe(param *vo.SubscribeParam) error

   // 取消订阅
   Unsubscribe(param *vo.SubscribeParam) error

   // 获取所有服务信息
   GetAllServicesInfo(param vo.GetAllServiceInfoParam) (model.ServiceList, error)

   //关闭客户端
   CloseClient()
}
```

### 构成


```go
type NamingClient struct {
    // 用户获取客户端配置、获取服务端配置、获取http代理的接口
	nacos_client.INacosClient
    // 服务相关代理，用于注册、注销、发现实例等，使用的是NamingProxyDelegate，这个类是对httpProxy和grpcProxy的再一次封装
	serviceProxy      naming_proxy.INamingProxy
    // 服务信息持有者，用于存储service信息、解析service信息、注册或者解除service callback(用于在service变化时回调)
	serviceInfoHolder *naming_cache.ServiceInfoHolder
}
```

#### NamingProxyDelegate

naming client对于底层client的额外一层封装，在2.0中，大部分时候都是直接调用grpcClient，注册和注销instance的时候，会根据instance的类型，决定是使用http还是grpc(nacos 1.X和2.X的调用方式不同，1.X是Http，2.X是Grpc)

```
type NamingProxyDelegate struct {
   // http 代理
   httpClientProxy   *naming_http.NamingHttpProxy
   // grpc代理
   grpcClientProxy   *naming_grpc.NamingGrpcProxy
   // 服务持有者，同naming client
   serviceInfoHolder *naming_cache.ServiceInfoHolder
}
```

##### httpClient

是INamingProxy的实现类，主要用于使用http的方式去实现接口注册、接口注销等操作。

```
type NamingHttpProxy struct {
   // 客户端信息
   clientConfig      constant.ClientConfig
   // server端信息
   nacosServer       *nacos_server.NacosServer
   // 心跳反应器，用于监听心跳
   beatReactor       BeatReactor
   serviceInfoHolder *naming_cache.ServiceInfoHolder
}
```

###### beatReactor

是和server进行心跳的反应器，主要是在1.0中使用，2.0中使用grpc，没有心跳了。

```
type BeatReactor struct {
   beatMap             cache.ConcurrentMap
   nacosServer         *nacos_server.NacosServer
   beatThreadCount     int
   beatThreadSemaphore *semaphore.Weighted
   beatRecordMap       cache.ConcurrentMap
   clientCfg           constant.ClientConfig
   mux                 *sync.Mutex
}
```

核心方法：

1. AddBeatInfo 添加心跳

   接收serviceName和beatinfo两个参数，serviceName用于生成key(根据serviceName+IP+端口生成)，key为BeatReactor中beatMap的key值，用于存储对应的beatInfo,暂未知道这个beatMap有何作用...

   BeatInfo会包含instance中的基本信息，包括ip、端口、命名空间等信息。

   1.1  根据key移除原有的beatMap中的beatInfo，

   1.2  开启一个协程，不断发送心跳请求到server端，这个协程会根据beatInfo中的间隔请求，**可以通过设置beatInfo的state**，停止心跳协程。

2. RemoveBeatInfo移除心跳

   接收ServiceName+IP+端口生成key，**然后将对应的beatInfo的state改为停止**

###### serviceInfoHolder

  serviceInfo的持有者，会缓存serviceInfo和接受server发过来的service变更信息。

```
type ServiceInfoHolder struct {
   // serviceinfo的缓存
   ServiceInfoMap       cache.ConcurrentMap
   // 是否在service为空时不进行更新本地缓存
   updateCacheWhenEmpty bool
   // 缓存目录，通过缓存文件的形式缓存service信息
   cacheDir             string
   // 在启动时不加载缓存，为false时，表示加载缓存
   notLoadCacheAtStart  bool
   // 订阅回调，用于在service发生变化时，执行定制操作
   subCallback          *SubscribeCallback
   // 更新时间map，记录缓存更新时间
   UpdateTimeMap        cache.ConcurrentMap
}
```

核心方法：

1.ProcessService 解析service,解析完成会把service放入到serviceInfoMap中

2.loadCacheFromDisk 从缓存文件中加载缓存，并放入到serviceInfoMap中去

3.RegisterCallback 注册回调函数

4.DeregisterCallback取消回调函数注册

###### SubscribeCallback

注册回调器，用于注册一些在服务变动。

```
type SubscribeCallback struct {
   // map，用于存储service和对应callback之间的关系
   callbackFuncMap cache.ConcurrentMap
   // 锁
   mux             *sync.Mutex
}
```

核心方法：

1.AddCallbackFunc 添加回调方法到map中

2.RemoveCallbackFunc 移除回调方法到map中

3.ServiceChanged 服务发生变化时，触发回调



##### httpGrpcClient

```go
type NamingGrpcProxy struct {
   // 客户端配置
   clientConfig      constant.ClientConfig
   // 服务端配置
   nacosServer       *nacos_server.NacosServer
   // rpc 客户端，用于发送请求
   rpcClient         rpc.IRpcClient
   // 时间监听器
   eventListener     *ConnectionEventListener
   // 服务信息持有者
   serviceInfoHolder *naming_cache.ServiceInfoHolder
}
```

grpc的代理类，其中rpcClient用于真正的和服务端发送请求。

###### rpcClient

```go
// IRpcClient 接口，定义的rpc client应该实现的方法，是grpc client的父类
type IRpcClient interface {
   connectToServer(serverInfo ServerInfo) (IConnection, error)
   getConnectionType() ConnectionType
   putAllLabels(labels map[string]string)
   rpcPortOffset() uint64
   GetRpcClient() *RpcClient
}
```

//  grpc版本 rpc client

```
// grpc 版本的 rpc client实现
type GrpcClient struct {
   *RpcClient
}
```



底层的client为RpcClient  -- TODO

```
type RpcClient struct {
   // 名称
   Name                        string
   // 标签
   labels                      map[string]string
   // 当前连接
   currentConnection           IConnection
   // client状态
   rpcClientStatus             RpcClientStatus
   // 事件chan
   eventChan                   chan ConnectionEvent
   // 重连chan
   reconnectionChan            chan ReconnectContext
   // 连接事件监听器
   connectionEventListeners    atomic.Value
   // 上次活跃的时间
   lastActiveTimeStamp         time.Time
   // 执行的client,用于发送真正的请求
   executeClient               IRpcClient
   // nacos server信息
   nacosServer                 *nacos_server.NacosServer
   // request的处理器映射，不同的request对应不同的handler
   serverRequestHandlerMapping map[string]ServerRequestHandlerMapping
   mux                         *sync.Mutex
   clientAbilities             rpc_request.ClientAbilities
   Tenant                      string
}
```
