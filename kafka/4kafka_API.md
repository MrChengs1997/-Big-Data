## Producer API

### 1.1、消息发送流程

Kafka的Producer发送消息采用的是**异步发送**方式

在消息发送过程中，涉及到两个线程---**mian线程**和**Sender线程**

以及**一个此案成共享变量--RecordAccumulator**

mian线程将消息发送RecordAccumulator

sender线程不断从RecordAccumulator中拉取消息发送到Kafka broker



![](picc/发送消息流程.png)

batch.size : 只有数据累计到batch.size之后，sender才会发送数据

linger.ms：如果数据迟迟未到batch.size，sender等到linger.time之后就会发送数据





### 1.2、环境准备

添加所需要的依赖

```
    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>0.11.0.0</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.kafka/kafka -->
    <dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.12</artifactId>
      <version>0.11.0.0</version>
    </dependency>
```



### 1.3、创建生产者(不带回调)

```
package com.mrcheng.kafka.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class MyProducer {

    public static void main(String[] args) {

        //1、创建生产者配置信息
        Properties properties = new Properties();
        // Kafka 服务端的主机名和端口号
        properties.put("bootstrap.servers", "hadoop2:9092,hadoop3:9092,hadoop4:9092");
        // 等待所有副本节点的应答级别
        properties.put("acks", "all");
        // 消息发送最大尝试次数
        properties.put("retries", 2);
        // 一批消息处理大小
        properties.put("batch.size", 16384);
        // 请求延时
        properties.put("linger.ms", 100);
        // 发送缓存区内存大小
        properties.put("buffer.memory", 33554432);//32M

        //properties.put("group.id", "test-consumer-group");//32M

        // key 序列化
        properties.put("key.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");
        // value 序列化
        properties.put("value.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");

//ERROR [ConsumerFetcherThread-console-consumer-72992_hadoop3-1575041762056-e8228ef3-0-0]:
// Error for partition [first,1]
// to broker 0:kafka.common.NotLeaderForPartitionException (kafka.consumer.ConsumerFetcherThread

        //2、创建生产者对象
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
        System.out.println("创建生产者对象...");
        System.out.println(producer.toString());


        for (int i =0 ;i<10;i++){
            System.out.println("msg" + i);
            //3、配置主题发送信息
            //Future<RecordMetadata> send(ProducerRecord<K, V> var1);
            //回调函数可以把当前数据在Kafka中的信息都返回
            //Future<RecordMetadata> send(ProducerRecord<K, V> var1, Callback var2);
            String msg = "msg" + i;
            producer.send(new ProducerRecord<>("first",msg));
        }

        System.out.println("结束1...");
        //4、关闭
        producer.close();
        System.out.println("结束2...");
    }
}

```



在命令行终端打开消费者进行消费消息

```
[root@hadoop2 kafka]# bin/kafka-console-consumer.sh  --topic first --zookeeper hadoop2:2181
```



执行代码之后

```
[root@hadoop2 kafka]# bin/kafka-console-consumer.sh  --topic first --zookeeper hadoop2:2181
Using the ConsoleConsumer with old consumer is deprecated and will be removed in a future major release. Consider using the new consumer by passing [bootstrap-server] instead of [zookeeper].
msg1
msg3
msg5
msg7
msg9
msg0
msg2
msg4
msg6
msg8
```



注意：可以使用进行替换



```
properties.put("bootstrap.servers", "hadoop2:9092,hadoop3:9092,hadoop4:9092");
ProducerConfig.BOOTSTRAP_SERVERS_CONFIG;
ProducerConfig.ACKS_CONFIG;
```





### 1.4、创建生产者（带回调）

```
public static void main(String[] args) {

        //1、创建生产者配置信息
        Properties properties = new Properties();
        // Kafka 服务端的主机名和端口号
        //ProducerConfig.ACKS_CONFIG;
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop2:9092,hadoop3:9092,hadoop4:9092");
        // 等待所有副本节点的应答级别
        properties.put("acks", "all");
        // 消息发送最大尝试次数
        properties.put("retries", 2);
        // 一批消息处理大小
        properties.put("batch.size", 16384);
        // 请求延时
        properties.put("linger.ms", 100);
        // 发送缓存区内存大小
        properties.put("buffer.memory", 33554432);//32M

        //properties.put("group.id", "test-consumer-group");//32M

        // key 序列化
        properties.put("key.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");
        // value 序列化
        properties.put("value.serializer",
                "org.apache.kafka.common.serialization.StringSerializer");


        //2、创建生产者对象
        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
        System.out.println("创建生产者对象...");
        System.out.println(producer.toString());


        for (int i =0 ;i<10;i++){
            System.out.println("msg" + i);
            //3、配置主题发送信息
            //Future<RecordMetadata> send(ProducerRecord<K, V> var1);
            //回调函数可以把当前数据在Kafka中的信息都返回
            //Future<RecordMetadata> send(ProducerRecord<K, V> var1, Callback var2);
            String msg = "msg" + i;
            producer.send(new ProducerRecord<>("first", msg), new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    //RecordMetadata:当前行数据的行信息
                    //Exception：异常

                    //由异常返回异常
                    //无异常但会数据信息
                    if (e == null){
                        System.out.println(recordMetadata.partition()+"--"+ recordMetadata.offset());
                    }else {
                        e.printStackTrace();
                    }

                }
            });
        }

        System.out.println("结束1...");
        //4、关闭
        producer.close();
        System.out.println("结束2...");


    }

```



控制台结果

```
创建生产者对象...
org.apache.kafka.clients.producer.KafkaProducer@30c7da1e
msg0
msg1
msg2
msg3
msg4
msg5
msg6
msg7
msg8
msg9
结束1...
0--22
0--23
0--24
0--25
0--26
1--30
1--31
1--32
1--33
1--34
结束2...
```



``````····ddwe
msg1
msg3
msg5
msg7
msg9
msg0
msg2
msg4
msg6
msg8

``````



### 1.5 ProducerRecord

new ProducerRecord<>("first", 0,"key",msg), new Callback() {});

first:topic

0:patition

key:key

msg:消息

```
 String msg = "msg" + i;
            producer.send(new ProducerRecord<>("first", 0,"key",msg), new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    //RecordMetadata:当前行数据的行信息
                    //Exception：异常

                    //由异常返回异常
                    //无异常但会数据信息
                    if (e == null){
                        System.out.println(recordMetadata.partition()+"--"+ recordMetadata.offset());
                    }else {
                        e.printStackTrace();
                    }

                }
            });
        }
```



此时使用控制台只能打印出value（设置的key不能打印出来）



