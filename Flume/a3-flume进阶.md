# **Flume** **进阶**

## **3.1 Flume** **事务**



![](picc/事务.png)

channel被动接收source的时间

channel被动被sinks拉去数据



doCommit()首先去检查Channel中的内存是否足够合并，足够则推送，否则回滚

doRollback（）做事务回滚此时数据依然在putList中存放



Take事务和put事务类似

事务回滚到Channel中





## **3.2 Flume Agent** **内部原理**





![](picc/内部原理.png)







**重要组件：**

**1）ChannelSelector** 

ChannelSelector 的作用就是选出 Event 将要被发往哪个 Channel。其共有两种类型， 

分别是 **Replicating**（复制）和 **Multiplexing**（多路复用）。

ReplicatingSelector 会将同一个 Event 发往所有的 Channel，Multiplexing 会根据相 

应的原则，将不同的 Event 发往不同的 Channel。



Replicating Channel Selector (default)

```
a1.sources = r1
a1.channels = c1 c2 c3
a1.sources.r1.selector.type = replicating
a1.sources.r1.channels = c1 c2 c3
a1.sources.r1.selector.optional = c3
```





Multiplexing Channel Selector

```
a1.sources = r1
a1.channels = c1 c2 c3 c4
a1.sources.r1.selector.type = multiplexing
多了复用需要跟拦截器一起用  添加一个头（头结构是map）
key是state
value是CZ、US

a1.sources.r1.selector.header = state  
a1.sources.r1.selector.mapping.CZ = c1
a1.sources.r1.selector.mapping.US = c2 c3
a1.sources.r1.selector.default = c4
```



**2）SinkProcessor** 

SinkProcessor 共 有 三 种 类 型 ， 分 别 是 **DefaultSinkProcessor** 、 

**LoadBalancingSinkProcessor** 和 **FailoverSinkProcessor** 

DefaultSinkProcessor 对 应 的 是 单 个 的 Sink ， LoadBalancingSinkProcessor 和 

FailoverSinkProcessor 对应的是 Sink Group，LoadBalancingSinkProcessor 可以实现负 

载均衡的功能，FailoverSinkProcessor 可以实现故障转移的功能。 



Default Sink Processor

Default sink processor accepts only a single sink. User is not forced to create processor (sink group) for single sinks. Instead user can follow the source - channel - sink pattern that was explained above in this user guide.



Failover Sink Processor

```
processor.type	default	The component type name, needs to be failover
```



Load balancing Sink Processor

```


processor.sinks	 –	Space-separated list of sinks that are participating in the group
processor.type	default	The component type name, needs to be load_balance
processor.backoff	false	Should failed sinks be backed off exponentially.
processor.selector	round_robin	Selection mechanism. Must be either round_robin, random or FQCN of custom class that inherits from AbstractSinkSelector
processor.selector.maxTimeOut	30000	Used by backoff selectors to limit exponential backoff (in milliseconds)
```

processor.selector.maxTimeOut	30000	Used by backoff selectors to limit exponential backoff (in milliseconds) 如果sinks挂掉此时在单位时间内就不会访问该挂掉的sinks（退避时间是 2 * n此访问（每次时间）

此时最大值是3000 ，在30s之后 依然会轮询进行访问



## **3.3 Flume** **拓扑结构**

### **3.3.1** **简单串联**



![](picc/串联.png)



Flume Agent 连接



这种模式是将多个 flume 顺序连接起来了，从最初的 source 开始到最终 sink 传送的 

目的存储系统。此模式不建议桥接过多的 flume 数量，flume 数量过多不仅会影响传输速率， 

而且一旦传输过程中某个节点 flume 宕机，会影响整个传输系统。 





### **3.3.2** **复制和多路复用**

![](picc/多路复用.png)



单 source，多 channel、sink



Flume 支持将事件流向一个或者多个目的地。这种模式可以将相同数据复制到多个 

channel 中，或者将不同数据分发到不同的 channel 中，sink 可以选择传送到不同的目的 

地。



### **3.3.3** **负载均衡和故障转移**



![](picc/负载均衡.png)



Flume 负载均衡或故障转移

Flume支持使用将多个sink逻辑上分到一个sink组，sink组配合不同的SinkProcessor 

可以实现负载均衡和错误恢复的功能。





### **3.3.4** **聚合**



![](picc/聚合.png)

这种模式是我们最常见的，也非常实用，日常 web 应用通常分布在上百个服务器，大者 

甚至上千个、上万个服务器。产生的日志，处理起来也非常麻烦。用 flume 的这种组合方式 

能很好的解决这一问题，每台服务器部署一个 flume 采集日志，传送到一个集中收集日志的 

flume，再由此 flume 上传到 hdfs、hive、hbase 等，进行日志分析。 