# **2.1 Flume** **安装部署**

## **2.1.1** **安装地址**

1） Flume 官网地址 

http://flume.apache.org/ 

2）文档查看地址 

http://flume.apache.org/FlumeUserGuide.html 

3）下载地址 

http://archive.apache.org/dist/flume/





## **2.1.2** **安装部署**

1）将 apache-flume-1.7.0-bin.tar.gz 上传到 linux 的/opt/software 目录下 

2）解压 apache-flume-1.7.0-bin.tar.gz 到/opt/module/目录下

```
tar -zxf apache-flume-1.7.0-bin.tar.gz -C /opt/module/
```



3）修改 apache-flume-1.7.0-bin 的名称为 flume

```
 mv apache-flume-1.7.0-bin flume
```



4）将 flume/conf 下的 flume-env.sh.template 文件修改为 flume-env.sh，并配置 flume

env.sh 文件

```
[root@hadoop102 conf]$ mv flume-env.sh.template flume-env.sh
[root@hadoop102 conf]$ vi flume-env.sh
export JAVA_HOME=/opt/module/jdk1.8.0_144
```



# **2.2 Flume** **入门案例**

## **2.2.1** **监控端口数据官方案例**

**1）案例需求：** 

使用 Flume 监听一个端口，收集该端口数据，并打印到控制台。 

**2）需求分析：**

![](picc/案例.png)



**3）实现步骤：** 

**1.安装 netcat 工具** 

```
 sudo yum install -y nc
```

**2.判断 44444 端口是否被占用**

```
sudo netstat -tunlp | grep 44444
```



客户端必须依赖服务端存在

不可以单独存在



**3.创建 Flume Agent 配置文件 flume-netcat-logger.conf**

在 flume 目录下创建 job 文件夹并进入 job 文件夹。

```
mkdir job
```

在 job 文件夹下创建 Flume Agent 配置文件 flume-netcat-logger.conf。

```
vim flume-netcat-logger.conf
```

在 flume-netcat-logger.conf 文件中添加如下内容

```
添加内容如下：
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1
# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444
# Describe the sink
a1.sinks.k1.type = logger
# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

注：配置文件来源于官方手册 http://flume.apache.org/FlumeUserGuide.html



![](picc/simaple1.png)



**4. 先开启 flume 监听端口**

**第一种写法：**

```
[root@hadoop2 flume]# bin/flume-ng agent --conf conf/ --conf-file job/flume-netcat-logger.conf --name a1 -Dflume.root.logger=INFO,console

...
ame: c1 started
2019-10-25 12:01:01,005 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:171)] Starting Sink k1
2019-10-25 12:01:01,009 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:182)] Starting Source r1
2019-10-25 12:01:01,011 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:155)] Source starting
2019-10-25 12:01:01,060 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.source.NetcatSource.start(NetcatSource.java:169)] Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]

```



本地开启一个窗口进行对本地的44444端口的服务

```
[root@hadoop2 ~]# nc localhost 44444

```

进行发送消息

```
[root@hadoop2 ~]# nc localhost 44444
hello
OK

```



本地服务端会接收到消息

```
mpl[/127.0.0.1:44444]
2019-10-25 12:03:01,471 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 68 65 6C 6C 6F                                  hello }

```

关闭服务端

```
^C2019-10-25 12:04:00,014 (agent-shutdown-hook) [INFO - org.apache.flume.lifecycle.LifecycleSupervisor.stop(LifecycleSupervisor.java:78)] Stopping lifecycle supervisor 11
2019-10-25 12:04:00,021 (agent-shutdown-hook) [INFO - org.apache.flume.node.PollingPropertiesFileConfigurationProvider.stop(PollingPropertiesFileConfigurationProvider.java:84)] Configuration provider stopping
2019-10-25 12:04:00,021 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:149)] Component type: CHANNEL, name: c1 stopped
2019-10-25 12:04:00,021 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:155)] Shutdown Metric for type: CHANNEL, name: c1. channel.start.time == 1572019261004
2019-10-25 12:04:00,022 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:161)] Shutdown Metric for type: CHANNEL, name: c1. channel.stop.time == 1572019440021
2019-10-25 12:04:00,022 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:177)] Shutdown Metric for type: CHANNEL, name: c1. channel.capacity == 1000
2019-10-25 12:04:00,023 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:177)] Shutdown Metric for type: CHANNEL, name: c1. channel.current.size == 0
2019-10-25 12:04:00,023 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:177)] Shutdown Metric for type: CHANNEL, name: c1. channel.event.put.attempt == 1
2019-10-25 12:04:00,023 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:177)] Shutdown Metric for type: CHANNEL, name: c1. channel.event.put.success == 1
2019-10-25 12:04:00,023 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:177)] Shutdown Metric for type: CHANNEL, name: c1. channel.event.take.attempt == 26
2019-10-25 12:04:00,023 (agent-shutdown-hook) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.stop(MonitoredCounterGroup.java:177)] Shutdown Metric for type: CHANNEL, name: c1. channel.event.take.success == 1
2019-10-25 12:04:00,025 (agent-shutdown-hook) [INFO - org.apache.flume.source.NetcatSource.stop(NetcatSource.java:196)] Source stopping

```

agent-shutdown-hook钩子程序进行一些收尾工作

先听source然后将channel中的数据处理完成



**第二种写法**：

```
 bin/flume-ng agent -c conf/ -n $agent_name -f 
job/flume-netcat-logger.conf  -Dflume.root.logger=INFO,console
```

参数说明： 

--conf/-c：表示配置文件存储在 conf/目录 

--name/-n：表示给 agent 起名为 a1 

--conf-file/-f：flume 本次启动读取的配置文件是在 job 文件夹下的 flume-telnet.conf 

文件。

-Dflume.root.logger=INFO,console ：-D 表示 flume 运行时动态修改 flume.root.logger 

参数属性值，并将控制台日志打印级别设置为 INFO 级别。日志级别包括:log、info、warn、 

error。 



## **2.2.2** **实时监控单个追加文件**

**1）案例需求：实时监控 Hive 日志，并上传到 HDFS 中** 



**2）需求分析**： 

![](picc/simaple2.png)

**3）实现步骤：** 

**1.Flume 要想将数据输出到 HDFS，须持有 Hadoop 相关 jar 包**

需要些数据到hdfs

```
commons-configuration-1.6.jar
hadoop-auth-2.7.2.jar
hadoop-common-2.7.2.jar
hadoop-hdfs-2.7.2.jar
commons-io-2.4.jar
htrace-core-3.1.0-incubating.jar
```

拷贝到/opt/module/flume/lib 文件夹下。





**2.创建 flume-file-hdfs.conf 文件**

job目录下

```
 vim flume-file-hdfs.conf
```

注：要想读取 Linux 系统中的文件，就得按照 Linux 命令的规则执行命令。由于 Hive 日志 

在 Linux 系统中所以读取文件的类型选择：exec 即 execute 执行的意思。表示执行 Linux 

命令来读取文件。

```
添加内容如下：
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F  /opt/module/hive/logs/hive.log


# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```





**3.运行 Flume**

```
bin/flume-ng agent -c conf/ -f job/flume-file-hdfs.conf -n a1 -Dflume.root.logger=INFO,console
```



默认会打印10行

```
39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,364 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,364 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,364 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,365 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,365 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,365 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,366 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,366 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }
2019-10-27 09:30:11,366 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 30 20 31 30 3A 35 33 2019-10-20 10:53 }

```





**4.开启 Hadoop 和 Hive 并操作 Hive 产生日志**

```
bin/hive
```



开启hive之后进行打印日志10行





使用hive进行查询

```
hive (default)> select * from stu;
```



此时进行打印的日志

```
....
2019-10-27 09:36:37,251 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 37 20 30 39 3A 33 36 2019-10-27 09:36 }
2019-10-27 09:36:37,251 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 32 30 31 39 2D 31 30 2D 32 37 20 30 39 3A 33 36 2019-10-27 09:36 }
...

```





**5.使用到HDFS**

```
# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/module/hive/logs/hive.log
a2.sources.r2.shell = /bin/bash -c

# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop2:9000/flume/%Y%m%d/%H

#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs- #是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true

#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1

#重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour

#是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true

#积攒多少个 Event 才 flush 到 HDFS 一次
a2.sinks.k2.hdfs.batchSize = 1000

#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream

#多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 30

#设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700

#文件的滚动与 Event 数量无关
a2.sinks.k2.hdfs.rollCount = 0

# Use a channel which buffers events in memory
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

**注意**： 

对于所有与时间相关的转义序列，Event Header 中必须存在以 “timestamp”的 key（除非 

hdfs.useLocalTimeStamp 设置为 true，此方法会使用 TimestampInterceptor 自动添加 

timestamp）。 

a3.sinks.k3.hdfs.useLocalTimeStamp = true



配置文件

![](picc/HDFS.png)



启动flume

```
[root@hadoop2 flume]# bin/flume-ng agent -c conf/ -f job/flume-file-hdfs.conf  -n a2

```



启动之后就会创建相应的文件夹

在点进去之后会创建相应的时间类型的文件夹

![](picc/hdfs_flume.png)



执行一个hive命令

```
hive (default)> select * from stu;
```

![](picc/hive_select.png)





## 2.2.3 实时监控目录下多个新文件

**1）案例需求：使用 Flume 监听整个目录的文件，并上传至 HDFS** 

**2）需求分析：**

![](picc/spooldir.png)

**1．创建配置文件 flume-dir-hdfs.conf** 

创建一个文件 

```
 vim flume-dir-hdfs.conf
```

```
a3.sources = r3
a3.sinks = k3
a3.channels = c3
# Describe/configure the source
a3.sources.r3.type = spooldir
a3.sources.r3.spoolDir = /opt/module/flume/upload
a3.sources.r3.fileSuffix = .COMPLETED
a3.sources.r3.fileHeader = true
#忽略所有以.tmp 结尾的文件，不上传
a3.sources.r3.ignorePattern = ([^ ]*\.tmp)
# Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path = 
hdfs://hadoop102:9000/flume/upload/%Y%m%d/%H
#上传文件的前缀
a3.sinks.k3.hdfs.filePrefix = upload- #是否按照时间滚动文件夹
a3.sinks.k3.hdfs.round = true
#多少时间单位创建一个新的文件夹
a3.sinks.k3.hdfs.roundValue = 1
#重新定义时间单位
a3.sinks.k3.hdfs.roundUnit = hour
#是否使用本地时间戳
a3.sinks.k3.hdfs.useLocalTimeStamp = true
#积攒多少个 Event 才 flush 到 HDFS 一次
a3.sinks.k3.hdfs.batchSize = 100
实时读取目录文件到HDFS案例
a3.sources = r3 a3.sinks = k3 a3.channels = c3 # Describe/configure the source
a3.sources.r3.type = spooldir
a3.sources.r3.spoolDir = /opt/module/flume/upload
a3.sources.r3.fileSuffix = .COMPLETED
a3.sources.r3.fileHeader = true
a3.sources.r3.ignorePattern = ([^ ]*\.tmp) # Describe the sink
a3.sinks.k3.type = hdfs
a3.sinks.k3.hdfs.path =
hdfs://hadoop102:9000/flume/upload/%Y%m%d/% Ha3.sinks.k3.hdfs.filePrefix = upload- a3.sinks.k3.hdfs.round = true
a3.sinks.k3.hdfs.roundValue = 1 a3.sinks.k3.hdfs.roundUnit = hour
a3.sinks.k3.hdfs.useLocalTimeStamp = true
a3.sinks.k3.hdfs.batchSize = 100
a3.sinks.k3.hdfs.fileType = DataStream
a3.sinks.k3.hdfs.rollInterval = 60
a3.sinks.k3.hdfs.rollSize = 134217700
a3.sinks.k3.hdfs.rollCount = 0 # Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100
# Bind the source and sink to the channel
a3.sources.r3.channels = c3 a3.sinks.k3.channel = c3 #定义source
#定义sink
#定义channel
#定义source类型为目录
#定义监控目录
#定义文件上传完，后缀
#是否有文件头
#忽略所有以.tmp结尾的文件，不上传
#sink类型为hdfs
#文件上传到hdfs的路径
#上传文件到hdfs的前缀
#是否按时间滚动文件
#多少时间单位创建一个新的文件夹
#重新定义时间单位
#是否使用本地时间戳
#积攒多少个Event才flush到HDFS一次
#设置文件类型，可支持压缩
#多久生成新文件
#多大生成新文件
#多少event生成新文件

#设置文件类型，可支持压缩
a3.sinks.k3.hdfs.fileType = DataStream
#多久生成一个新的文件
a3.sinks.k3.hdfs.rollInterval = 60
#设置每个文件的滚动大小大概是 128M
a3.sinks.k3.hdfs.rollSize = 134217700
#文件的滚动与 Event 数量无关
a3.sinks.k3.hdfs.rollCount = 0
# Use a channel which buffers events in memory
a3.channels.c3.type = memory
a3.channels.c3.capacity = 1000
a3.channels.c3.transactionCapacity = 100
# Bind the source and sink to the channel
a3.sources.r3.channels = c3
a3.sinks.k3.channel = c3
```



![](picc/spooldir_test.png)



**2.启动监控文件夹命令**

```
[root@hadoop2 flume]# bin/flume-ng agent  -c conf/ -f job/flume-dir-hdfs.conf  -n a3

```

说明：在使用 Spooling Directory Source 时 

不要在监控目录中创建并持续修改文件 

上传完成的文件会以.COMPLETED 结尾 

被监控文件夹每 500 毫秒扫描一次文件变动



**3. 向 upload 文件夹中添加文件**

在/opt/module/flume 目录下创建 upload 文件夹

```
 mkdir upload
```

向 upload 文件夹中添加文件

```
cp a.txt  upload/

```



**4. 查看 HDFS 上的数据**

![](picc/dir.png)

查看hdfs上的文件

```
[root@hadoop2 hadoop-2.7.2]# bin/hadoop fs -cat /flume/upload/20191028/12/upload-.1572278671681
123123
```

查看源文件

```
[root@hadoop2 flume]# cat a.txt 
123123
```



```
[root@hadoop2 flume]# ll upload/
total 8
-rw-r--r--. 1 root root    7 Oct 28 12:04 a.txt.COMPLETED
-rw-r--r--. 1 root root 2520 Oct 28 11:59 README.md.COMPLETED
```



对于后缀名只是简单的进行校验

否则可能会被拦截

已经放进去的文件此时不会被二次上传（即新文件）



此时会报错

在报错的情况下不能再次进行上传

此时需要进行删除之后上传的文件



可以使用正则  匹配后置结尾的文件

如a.tmp

可以被过滤就不会进行上传新文件





## **2.2.4 实时监控目录下的多个追加文件**

