### 起因

因为想参与到nacos-sdk-go的项目中去，选择了一个[issue](https://github.com/nacos-group/nacos-sdk-go/issues/231),目标是sdk集成prometheus。

issue中给出了提示，按照java中的MetricsMonitor类来做。

### 过程

1.学习prometheus基本原理

​	主要分为两个组件，client和server。client负责提供监控的数据，server负责记录数据并提供查询数据。

​	最开始陷入了一个思维僵局，以为是client主动发送数据到server。但是后面发现是server主动来拉取，通过在server的配置文件中添加对应client的请求地址(用于获取监控数据)，定时调用对应接口以获取数据。

2.了解nacos的监控数据方式

​	需要开启spring actuator的端点(management.endpoints.web.exposure.include=*)，并在项目中添加下列jar包。

```
<dependency>
    <groupId>io.prometheus</groupId>
    <artifactId>simpleclient</artifactId>
    <version>0.5.0</version>
</dependency>
```

​	开启以后PrometheusScrapeEndpoint端点就开启了，开启以后，通过CollectorRegistry获取对应的数据即可。

3.收集器与收集器注册器

​	收集器是真正去收集数据的实体，每个收集器都需要注册到注册器CollectorRegistry中，统一通过CollectorRegistry与收集器交互。

​	收集器类型，Counter(累计计数器)、Gauge(瞬时状态收集器)、Histogram（图形收集器，会进行统计分析）、Summary（类似histogram）

​	CollectorRegistry核心方法：

​		注册：将收集器注册到注册器中，不注册的话，不会被收集。

​		scrape:不知道怎么翻译，大概就是用来获取所有收集器数据的。

4.标签

​	类似于k8s的标签，用来给收集器进行标记，然后进行多维度的查询的，区别在于，收集器在定义注册之初，就设定好了标签数量，后面再进行统计收集时，标签的数量必须对得上才行。



​	