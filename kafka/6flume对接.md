## kafka&Flume

**flume**：cloudera 公司研发: 

- 适合多个生产者； 

- 适合下游数据消费者不多的情况； 

- 适合数据安全性要求不高的操作； 

- 适合与 Hadoop 生态圈对接的操作。 

**kafka**：linkedin 公司研发: 

- 适合数据下游消费众多的情况； 

- 适合数据安全性要求较高的操作，支持 replication。 





常用模型

线上数据 --> flume --> kafka --> flume(根据情景增删该流程) --> HDFS



## 对接配置

- flume配置文件的生成 /.../flume/job文件 目录下

  kafka-flume.conf

  ```
  #定义
  a1.sources = r1
  a1.channels = c1
  a1.sinks = k1 
  
  #Source
  a1.sources.r1.type = netcat
  a1.sources.r1.bind = localhost
  a1.sources.r1.port = 44444
  
  #channels
  a1.channels.c1.type = memory
  a1.channels.c1.capacity = 10000
  a1.channels.c1.transactionCapacity = 100
  
  # sinks
  a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
  a1.sinks.k1.kafka.topic = first
  a1.sinks.k1.kafka.bootstrap.servers = hadoop2:9092,hadoop3:9092,hadoop4:9092
  a1.sinks.k1.kafka.flumeBatchSize = 20
  a1.sinks.k1.kafka.producer.acks = 1
  a1.sinks.k1.kafka.producer.linger.ms = 1
  
  
  
  
  #bind
  a1.sources.r1.channels = c1
  a1.sinks.k1.channel = c1
  
  
  ```

  

- 启动服务flume

  ```
  [root@hadoop2 flume]# bin/flume-ng agent -c conf/ -f job/kafka-flume.conf -n a1 
  
  ```

  

- 启动zk集群&kafka集群



- 开启Kafka消费端

  ```
  [root@hadoop3 kafka]# bin/kafka-console-consumer.sh  --topic first --zookeeper hadoop2:2181
  
  ```

  

- 启动netcat并且发送数据

  ```
  [root@hadoop2 ~]# nc localhost 44444
  ss
  OK
  helle
  OK
  flume
  OK
  kakfa
  OK
  
  
  ```

  

- 观察kafka消费端

  ```
  [root@hadoop3 kafka]# bin/kafka-console-consumer.sh  --topic first --zookeeper hadoop2:2181
  Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].
  ss
  helle
  flume
  kakfa
  
  ```

  