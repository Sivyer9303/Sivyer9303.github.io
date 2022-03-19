# Sivyer's Blog

### Introduction

> 私人博客，用于记录个人知识及总结，是使用自己的语言进行描述的，不保证不存在语法错误或者知识错误。

### Java

- JVM

  - [X] [jvm内存区域](./jvm/2021-04-11-jvm内存.md)
  - [X] [垃圾回收算法](./jvm/2021-04-19-垃圾回收算法.md)
  - [X] [垃圾收集器](./jvm/2021-04-21-垃圾收集器.md)
  - [ ] jvm错误排查手段及工具
  - [ ] 多线程及高并发
  - [ ] [容器类](./jvm/2021-04-23-容器类.md)
  - [ ] 类加载机制
  - [ ] 堆外内存
  - [ ] jvm内存模型
  - [ ] [aqs](./jvm/2021-05-27-aqs.md)
- jmh
- Redis

  - [X] [Redis数据结构](./Redis/2021-04-12-Redis数据结构.md)
  - [X] [Redis持久化](./Redis/2021-05-25-Redis持久化.md)
  - [X] [主从复制](./Redis/2021-05-29-Redis主从复制.md)
  - [X] [哨兵模式](./Redis/2021-06-01-Redis哨兵模式.md)
  - [X] [集群模式](./Redis/2021-06-03-Redis集群.md)
  - [X] [Redis中的事件](./Redis/2021-05-30-Redis事件.md)
  - [ ] [redis为什么高效](./Redis/2021-06-04-Redis高效原因.md)
  - [ ] 缓存雪崩、缓存穿透、缓存击穿
  - [X] [一致性hash](./Redis/2021-05-31-一致性hash.md)
  - [X] [Cluster slot](./Redis/2021-05-31-clusterslot.md)
  - [ ] 分布式锁
  - [ ] redis限流
- RocketMq

  - [ ] [基本概念](./RocketMq/2021-4-16-rocketmq基本概念.md)
  - [ ] 架构解析
  - [ ] 分布式事务
  - [ ] 心跳机制
  - [ ] 顺序消费
  - [ ] 消息去重
  - [ ] 消息重试
  - [ ] 主从复制
  - [ ] 高可用设计
- Zookeeper

  - [ ] 脑裂问题
  - [ ] 羊群效应
  - [ ] 分布式锁
- Kafka
- mongodb
- Es
- Netty
- benchmark
- Mysql

  - [ ] 游标cursor
  - [ ] 执行计划
  - [ ] 慢日志
- dubbo

  //TODO
- Spring

  - [ ] 事务监听器使用 - TransactionalEventListener
  - [X] [SpringBootApplication注解背后的秘密](./Spring/2021-05-25-SpringBootApplication背后的秘密.md)
  - [ ] [自动装配](./Spring/2021-05-25-自动装配.md)
- openresty
- mybatis-plus (baomidou)
- gRPC

  - [X] [gRPC简介](./gRpc/2021-05-12-gRpc.md)
- protocol buffers
- xxlJob
- prometheus

  - [X] [结合springboot、nacos学习prometheus](./prometheus/2022-01-20-结合springboot、nacos学习prometheus.md)
  - [ ] [go中的prometheus](./prometheus/2022-01-20-go中的prometheus.md)
- linux

  - [ ] [Linux命令](./linux/2021-05-12-Linux命令.md)
  - [ ] [namespace和cgroup](./k8s/2022-01-10-NameSpace和Cgroup.md)    -- TODO
- Consul

  - [ ] 安装
  - [ ] 使用Consul作为SpringCloud的注册发现服务-替换Euraka
  - [ ] 使用Consul作为SpringCloud的配置中心-替换SpringCloud Config
  - [ ] 使用Consul作为SpringCloud的事件分发中心-与SpringCloud Bus协作
  - [ ] Consul定制
  - [ ] Consul密码及权限控制
  - [ ] 尝试搭建Consul集群
  - [X] [与其他服务注册中心对比](./Consul/2021-05-18-各服务注册中心对比.md)
  - [ ] 与Fabio一起使用
- nacos

  - [ ] [搭建nacos](./Nacos/2021-06-07-搭建nacos.md)
  - [ ] [nacos的相关概念](./Nacos/2021-06-07-nacos基本概念.md)
  - [ ] nacos与Spring cloud集成
  - [ ] nacos权限控制
  - [ ] nacos-sdk-go
    - [ ] [整体架构分析](./Nacos/nacos-sdk-go/整体架构分析.md)
  - [ ] nacos是如何实现配置自动刷新的
    - [ ] NacosContextRefresher - nacos自动更新的入口
    - [ ] RefreshEvent - spring cloud 热更新入口
- K8S和Docker

  - [X] [k8s有哪些组件](./k8s/2021-05-24-k8s有哪些组件.md)
  - [X] [ETCD](./k8s/2022-01-05-ETCD.md)      // TODO 深化一下，基本上完成   [参考链接](http://jolestar.com/etcd-architecture/)
  - [ ] [K8S开放接口](./k8s/2022-01-07-K8S开放接口.md)    // TODO CSI未完成
  - [X] [Pod](./k8s/2022-01-09-Pod.md)
  - [X] [Node](./k8s/2022-01-14-Node.md)
  - [ ] [垃圾收集](./k8s/2022-02-24-垃圾收集.md)
  - [ ] [控制器](./k8s/2022-03-17-控制器.md)
  - [ ] HPA
- 一致性算法

  - [X] [一致性协议总览](./一致性协议/2021-05-19-一致性协议总览.md)
  - [X] [Paxos协议](./一致性协议/2021-05-19-Paxos协议.md)
  - [X] [Raft协议](./一致性协议/2021-05-19-Raft协议.md)
  - [X] [ZAB协议](./一致性协议/2021-05-19-ZAB协议.md)
  - [X] [Gossip协议](./一致性协议/2021-05-22-Gossip协议.md)
- 分布式事务

  - [X] [一致性](./分布式事务/2021-05-20-一致性.md)
  - [X] [2PC与3PC](./分布式事务/2021-05-20-2PC与3PC.md)
  - [ ] seata
    - [ ] [坑爹，tinyint被转换为布尔](./seata/记一次坑爹的线上bug - tinyint被转换为布尔.md)
- 其他

  - [ ] [selector-epoll](./others/2021-05-26-SelectPoll模型.md)
  - [ ] dataX
  - [ ] 刷题规划
    - [ ] [数组](./算法/数组.md)
    - [ ] 链表
    - [ ] 哈希表
    - [ ] 字符串
    - [ ] 栈与队列
    - [ ] 树
    - [ ] 回溯
    - [ ] 贪心
    - [ ] 动态规划
    - [ ] 图
- 书籍阅读

  - [ ] [凤凰架构](./book/2021-10-12-凤凰架构.md)
