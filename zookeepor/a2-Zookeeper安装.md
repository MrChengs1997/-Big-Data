## 2.1、本地模式安装部署

**1．安装前准备**
（1）安装Jdk

（2）拷贝Zookeeper安装包到Linux系统下

（3）解压到指定目录

```
[root@localhost software]# tar -zxvf zookeeper-3.4.10.tar.gz -C /opt/module/
```

**2．配置修改**
（1）将/opt/module/zookeeper-3.4.10/conf这个路径下的zoo_sample.cfg修改为zoo.cfg；

```
[root@localhost conf]# cp zoo_sample.cfg zoo.cfg
```

（2）打开zoo.cfg文件，修改dataDir路径：

```
[root@localhost conf]# vim zoo.cfg
```

修改如下内容：

dataDir=/opt/module/zookeeper-3.4.10/zkData



（3）在/opt/module/zookeeper-3.4.10/这个目录上创建zkData文件夹

```
[root@localhost zookeeper-3.4.10]# mkdir zkData
```



**3．操作Zookeeper**
（1）启动Zookeeper

```
[root@localhost zookeeper-3.4.10]# bin/zkServer.sh start
```

（2）查看进程是否启动

```
[root@localhost zookeeper-3.4.10]# jps 
2728 QuorumPeerMain 
2747 Jps
```

（3）查看状态：

```
root@localhost zookeeper-3.4.10]# bin/zkServer.sh status ZooKeeper JMX enabled by default Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg Mode: standalone
```

（4）启动客户端：

```
[root@localhost zookeeper-3.4.10]# bin/zkCli.sh Connecting to localhost:2181 2019-10-01 11:20:10,075 [myid:] - INFO [main:Environment@100] - Client environment:zookeeper.version=3.4.10-39d3a4f269333c922ed3db283be479f9deacaa0f, built on 03/23/2017 10:13 GMT 2019-10-01 11:20:10,081 [myid:] - INFO [main:Environment@100] - Client environment:host.name=localhost 2019-10-01 11:20:10,081 [myid:] - INFO [main:Environment@100] - Client environment:java.version=1.8.0_131 2019-10-01 11:20:10,084 [myid:] - INFO [main:Environment@100] - Client environment:java.vendor=Oracle Corporation 2019-10-01 11:20:10,085 [myid:] - INFO [main:Environment@100] - Client environment:java.home=/opt/jdk1.8.0_131/jre 2019-10-01 11:20:10,085 [myid:] - INFO [main:Environment@100] - Client environment:java.class.path=/opt/module/zookeeper-3.4.10/bin/../build/classes:/opt/module/zookeeper-3.4.10/bin/../build/lib/*.jar:/opt/module/zookeeper-3.4.10/bin/../lib/slf4j-log4j12-1.6.1.jar:/opt/module/zookeeper-3.4.10/bin/../lib/slf4j-api-1.6.1.jar:/opt/module/zookeeper-3.4.10/bin/../lib/netty-3.10.5.Final.jar:/opt/module/zookeeper-3.4.10/bin/../lib/log4j-1.2.16.jar:/opt/module/zookeeper-3.4.10/bin/../lib/jline-0.9.94.jar:/opt/module/zookeeper-3.4.10/bin/../zookeeper-3.4.10.jar:/opt/module/zookeeper-3.4.10/bin/../src/java/lib/*.jar:/opt/module/zookeeper-3.4.10/bin/../conf: 2019-10-01 11:20:10,085 [myid:] - INFO [main:Environment@100] - Client environment:java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib 2019-10-01 11:20:10,085 [myid:] - INFO [main:Environment@100] - Client environment:java.io.tmpdir=/tmp 2019-10-01 11:20:10,086 [myid:] - INFO [main:Environment@100] - Client environment:java.compiler=<NA> 2019-10-01 11:20:10,086 [myid:] - INFO [main:Environment@100] - Client environment:os.name=Linux 2019-10-01 11:20:10,086 [myid:] - INFO [main:Environment@100] - Client environment:os.arch=amd64 2019-10-01 11:20:10,086 [myid:] - INFO [main:Environment@100] - Client environment:os.version=3.10.0-957.el7.x86_64 2019-10-01 11:20:10,087 [myid:] - INFO [main:Environment@100] - Client environment:user.name=root 2019-10-01 11:20:10,087 [myid:] - INFO [main:Environment@100] - Client environment:user.home=/root 2019-10-01 11:20:10,087 [myid:] - INFO [main:Environment@100] - Client environment:user.dir=/opt/module/zookeeper-3.4.10 2019-10-01 11:20:10,090 [myid:] - INFO [main:ZooKeeper@438] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@506c589e Welcome to ZooKeeper! 2019-10-01 11:20:10,130 [myid:] - INFO [main-SendThread(localhost:2181):ClientCnxn$SendThread@1032] - Opening socket connection to server localhost/0:0:0:0:0:0:0:1:2181. Will not attempt to authenticate using SASL (unknown error) JLine support is enabled 2019-10-01 11:20:10,307 [myid:] - INFO [main-SendThread(localhost:2181):ClientCnxn$SendThread@876] - Socket connection established to localhost/0:0:0:0:0:0:0:1:2181, initiating session 2019-10-01 11:20:10,353 [myid:] - INFO [main-SendThread(localhost:2181):ClientCnxn$SendThread@1299] - Session establishment complete on server localhost/0:0:0:0:0:0:0:1:2181, sessionid = 0x16d87e6eb810000, negotiated timeout = 30000 

WATCHER:: 

WatchedEvent state:SyncConnected type:None path:null
```

（5）退出客户端：

```
v[zk: localhost:2181(CONNECTED) 0] quit
```

（6）停止Zookeeper

```
[root@localhost zookeeper-3.4.10]# bin/zkServer.sh stop ZooKeeper JMX enabled by default Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg Stopping zookeeper ... STOPPED 

[root@localhost zookeeper-3.4.10]# jps 
2847 Jps 
[root@localhost zookeeper-3.4.10]#
```



## 2.2 配置参数解读

Zookeeper中的配置文件zoo.cfg中参数含义解读如下：

**1．tickTime =2000：通信心跳数**，Zookeeper服务器与客户端心跳时间，单位毫秒
Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。

它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2* tickTime)

**2．initLimit =10：LF初始通信时限**
集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。

**3．syncLimit =5：LF同步通信时限**
集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit *
tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。

**4．dataDir：数据文件目录+数据持久化路径**
主要用于保存Zookeeper中的数据。

**5．clientPort =2181：客户端连接端口**
监听客户端连接的端口。







