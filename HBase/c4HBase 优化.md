# **高可用**

在 HBase 中 HMaster 负责监控 HRegionServer 的生命周期，均衡 RegionServer 的负载

如果 HMaster 挂掉了，那么整个 HBase 集群将陷入不健康的状态，并且此时的工作状态并 

不会维持太久。所以 HBase 支持对 HMaster 的高可用配置。 

1．关闭 HBase 集群

```
 bin/stop-hbase.sh
```



2．在 conf 目录下创建 backup-masters 文件

```
touch conf/backup-masters
```



3．在 backup-masters 文件中配置高可用 HMaster 节点

```
 echo hadoop3 > conf/backup-masters
```



4．将整个 conf 目录 scp 到其他节点

```
[atguigu@hadoop3 hbase]$ scp -r conf/ hadoop2:/opt/module/hbase/
[atguigu@hadoop3 hbase]$ scp -r conf/ hadoop4:/opt/module/hbase/
```



5．打开页面测试查看 

http://hadooo3:16010



# **预分区**

每一个 region 维护着 StartRow 与 EndRow

如果加入的数据符合某个 Region 维护的 RowKey 范围，则该数据交给这个 Region 维护

将数据所要投放的分区提前大致的规划好，以提高 HBase 性能。



1．手动设定预分区

```
Hbase> create 'staff1','info','partition1',SPLITS => ['1000','2000','3000','4000']
```



2．生成 16 进制序列预分区

```
create 'staff2','info','partition2',{NUMREGIONS => 15, SPLITALGO => 'HexStringSplit'}
```



3．按照文件中设置的规则预分区 

创建 splits.txt 文件内容如下：

```
aaaa
bbbb
cccc
dddd
```

然后执行：

```
create 'staff3','partition3',SPLITS_FILE => 'splits.txt'
```



4．使用 JavaAPI 创建预分区

```
//自定义算法，产生一系列 hash 散列值存储在二维数组中
byte[][] splitKeys = 某个散列值函数
//创建 HbaseAdmin 实例
HBaseAdmin hAdmin = new HBaseAdmin(HbaseConfiguration.create());
//创建 HTableDescriptor 实例
HTableDescriptor tableDesc = new HTableDescriptor(tableName);
//通过 HTableDescriptor 实例和散列值二维数组创建带有预分区的 Hbase 表
hAdmin.createTable(tableDesc, splitKeys);
```



#  **RowKey** **设计**

一条数据的唯一标识就是 RowKey，那么这条数据存储于哪个分区，取决于 RowKey 处 

于哪个一个预分区的区间内，设计 RowKey 的主要目的 ，就是让数据均匀的分布于所有的 

region 中，在一定程度上防止数据倾斜。

1．生成随机数、hash、散列值

```
比如：
原 本 rowKey 为 1001 的 ， SHA1 后 变 成 ：
dd01903921ea24941c26a48f2cec24e0bb0e8cc7
原 本 rowKey 为 3001 的 ， SHA1 后 变 成 ：
49042c54de64a1e9bf0b33e00245660ef92dc7bd
原 本 rowKey 为 5001 的 ， SHA1 后 变 成 ：
7b61dec07e02c188790670af43e717f0f46e8913
在做此操作之前，一般我们会选择从数据集中抽取样本，来决定什么样的 rowKey 来 Hash
后作为每个分区的临界值。
```



2．字符串反转

```
20170524000001 转成 10000042507102
20170524000002 转成 20000042507102
```

这样也可以在一定程度上散列逐步 put 进来的数据。



3．字符串拼接

```
20170524000001_a12e
20170524000001_93i7
```



# **内存优化**

HBase 操作过程中需要大量的内存开销，毕竟 Table 是可以缓存在内存中的，一般会分 

配整个可用内存的 70%给 HBase 的 Java 堆。但是  ***不建议分配非常大的堆内存***  ，因为 GC 过 

程持续太久会导致 RegionServer 处于长期不可用状态，一般 16~48G 内存就可以了，如果因 

为框架占用内存过高导致系统内存不足，框架一样会被系统服务拖死











