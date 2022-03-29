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

   

   

