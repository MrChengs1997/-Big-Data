## **基本操作**



**1．进入 HBase 客户端命令行** 

```
[root@hadoop2 hbase]#  bin/hbase shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/module/hbase/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/opt/module/hadoop-2.7.2/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.3.1, r930b9a55528fe45d8edce7af42fef2d35e77677a, Thu Apr  6 19:36:54 PDT 2017

hbase(main):001:0> 

```



**2．查看帮助命令** 

```
hbase(main):001:0>  help

HBase Shell, version 1.3.1, r930b9a55528fe45d8edce7af42fef2d35e77677a, Thu Apr  6 19:36:54 PDT 2017
Type 'help "COMMAND"', (e.g. 'help "get"' -- the quotes are necessary) for help on a specific command.
Commands are grouped. Type 'help "COMMAND_GROUP"', (e.g. 'help "general"') for help on a command group.

COMMAND GROUPS:
  Group name: general
  Commands: status, table_help, version, whoami

  Group name: ddl
  Commands: alter, alter_async, alter_status, create, describe, disable, disable_all, drop, drop_all, enable, enable_all, exists, get_table, is_disabled, is_enabled, list, locate_region, show_filters

  Group name: namespace
  Commands: alter_namespace, create_namespace, describe_namespace, drop_namespace, list_namespace, list_namespace_tables

  Group name: dml
  Commands: append, count, delete, deleteall, get, get_counter, get_splits, incr, put, scan, truncate, truncate_preserve

  Group name: tools
  Commands: assign, balance_switch, balancer, balancer_enabled, catalogjanitor_enabled, catalogjanitor_run, catalogjanitor_switch, close_region, compact, compact_rs, flush, major_compact, merge_region, move, normalize, normalizer_enabled, normalizer_switch, split, splitormerge_enabled, splitormerge_switch, trace, unassign, wal_roll, zk_dump

  Group name: replication
  Commands: add_peer, append_peer_tableCFs, disable_peer, disable_table_replication, enable_peer, enable_table_replication, get_peer_config, list_peer_configs, list_peers, list_replicated_tables, remove_peer, remove_peer_tableCFs, set_peer_tableCFs, show_peer_tableCFs

  Group name: snapshots
  Commands: clone_snapshot, delete_all_snapshot, delete_snapshot, delete_table_snapshots, list_snapshots, list_table_snapshots, restore_snapshot, snapshot

  Group name: configuration
  Commands: update_all_config, update_config

  Group name: quotas
  Commands: list_quotas, set_quota

  Group name: security
  Commands: grant, list_security_capabilities, revoke, user_permission

  Group name: procedures
  Commands: abort_procedure, list_procedures

  Group name: visibility labels
  Commands: add_labels, clear_auths, get_auths, list_labels, set_auths, set_visibility

SHELL USAGE:
Quote all names in HBase Shell such as table and column names.  Commas delimit
command parameters.  Type <RETURN> after entering a command to run it.
Dictionaries of configuration used in the creation and alteration of tables are
Ruby Hashes. They look like this:

  {'key1' => 'value1', 'key2' => 'value2', ...}

and are opened and closed with curley-braces.  Key/values are delimited by the
'=>' character combination.  Usually keys are predefined constants such as
NAME, VERSIONS, COMPRESSION, etc.  Constants do not need to be quoted.  Type
'Object.constants' to see a (messy) list of all constants in the environment.

If you are using binary keys or values and need to enter them in the shell, use
double-quote'd hexadecimal representation. For example:

  hbase> get 't1', "key\x03\x3f\xcd"
  hbase> get 't1', "key\003\023\011"
  hbase> put 't1', "test\xef\xff", 'f1:', "\x01\x33\x40"

The HBase shell is the (J)Ruby IRB with the above HBase-specific commands added.
For more on the HBase Shell, see http://hbase.apache.org/book.html
```

**Group name: ddl**
Commands: alter, alter_async, alter_status, create, describe, disable, disable_all, drop, drop_all, enable, enable_all, exists, get_table, is_disabled, is_enabled, list, locate_region, show_filters



**Group name: dml**
Commands: append, count, **delete**, **deleteall**, **get**, get_counter, get_splits, incr, **put**, **scan**, truncate, truncate_preserve



**3．查看当前数据库中有哪些表** 

```
hbase(main):002:0>  list
TABLE                                                                                                             
0 row(s) in 1.8160 seconds

=> []

```





## **表的操作**



### **1．创建表** 

‘student’：表名

‘info’：列族

列族可以多个

```
hbase(main):003:0> create 'student','info'


0 row(s) in 12.4090 seconds

=> Hbase::Table - student
hbase(main):004:0> list
TABLE                                                                                                             
student                                                                                                           
1 row(s) in 0.1390 seconds

=> ["student"]

```



```
hbase(main):005:0> create 'stu','info1','info2'
0 row(s) in 4.3490 seconds

=> Hbase::Table - stu

```

```
hbase(main):008:0> describe 'stu'
Table stu is ENABLED                                                                                              
stu                                                                                                               
COLUMN FAMILIES DESCRIPTION                                                                                       
{NAME => 'info1', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA
_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLO
CKSIZE => '65536', REPLICATION_SCOPE => '0'}                                                                      
{NAME => 'info2', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA
_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLO
CKSIZE => '65536', REPLICATION_SCOPE => '0'}                                                                      
2 row(s) in 1.4010 seconds

```





### 2、命名空间

查看命名空间

```
hbase(main):010:0> list_namespace
NAMESPACE                                                                                                         
default                                                                                                           
hbase                                                                                                             
2 row(s) in 0.1380 seconds

```



创建命名空间

```
hbase(main):011:0> create_namespace 'bigdata'
0 row(s) in 1.0100 seconds

```



查看

```
hbase(main):012:0> list_namespace
NAMESPACE                                                                                                         
bigdata                                                                                                           
default                                                                                                           
hbase                                                                                                             
3 row(s) in 0.3210 seconds
```



把表创建在命名空间中

```
hbase(main):013:0> create 'bigdata:stu','info'
0 row(s) in 2.2810 seconds

=> Hbase::Table - bigdata:stu

```



查看

```
hbase(main):014:0> list
TABLE                                                                                                             
bigdata:stu                     
stu              
student                   
3 row(s) in 0.0150 seconds
=> ["bigdata:stu", "stu", "student"]
```

[**"bigdata:stu"**, "stu", "student"]

不带命名空间就是default目录下的



删除命名空间

注意：命名空间必须时控的才能删除

```
//disable表
hbase(main):015:0> disable 'bigdata:stu'
0 row(s) in 2.5240 seconds

//删除表
hbase(main):016:0> drop 'bigdata:stu'
0 row(s) in 2.5510 seconds

//删除命名空间
hbase(main):018:0> drop_namespace 'bigdata'
0 row(s) in 1.0320 seconds

```





### 3、DML

#### **新增**

```
hbase(main):001:0> put 'stu','1001','info1:name','zhnangsan'
0 row(s) in 2.4140 seconds

```



#### **get**

```
hbase(main):003:0> get 'stu','1001'
COLUMN                        CELL                                                                                
 info1:name                   timestamp=1577025881287, value=zhnangsan                                            
1 row(s) in 0.6680 seconds

```



插入多条数据

```
hbase(main):001:0> put 'stu','1001','info1:name','zhnangsan'
0 row(s) in 2.4140 seconds
hbase(main):002:0> put 'stu','1001','info1:sex','male'
0 row(s) in 0.2010 seconds
hbase(main):003:0> put 'stu','1001','info1:addr','beijing'
0 row(s) in 0.0670 seconds
hbase(main):004:0> put 'stu','1001','info1:name','lisi'
0 row(s) in 0.0160 seconds
hbase(main):005:0> put 'stu','1002','info1:phone','12341134143'
0 row(s) in 0.0400 seconds
hbase(main):006:0> put 'stu','1001','info2:addr','shanghai'
0 row(s) in 0.0270 seconds
```

```
hbase(main):007:0> scan 'stu'
ROW                           COLUMN+CELL                                                                         
 1001         column=info1:addr, timestamp=1577028345949, value=beijing                           
 1001         column=info1:name, timestamp=1577028352834, value=lisi                              
 1001         column=info1:sex, timestamp=1577028337497, value=male                               
 1001         column=info2:addr, timestamp=1577028364249, value=shanghai                          
 1002         column=info1:phone, timestamp=1577028358740, value=12341134143 
 2 row(s) in 0.0320 seconds
```

有几个数据看有几个rowkey（1001，1002）....

此时由于两条数据的在一行中回将value值继续宁替换

put 'stu','1001','info1:name','zhnangsan'

put 'stu','1001','info1:name','lisi'



使用get进行查询

```
hbase(main):012:0> get 'stu' ,'1001'
COLUMN                        CELL                                                       
 info1:addr                   timestamp=1577028345949, value=beijing                     
 info1:name                   timestamp=1577028678482, value=lisi                         
 info1:sex                    timestamp=1577028337497, value=male                         
 info2:addr                   timestamp=1577028364249, value=shanghai                     
1 row(s) in 0.1040 seconds

```

一行数据三个列



获取info下面name(指定到列)

```
hbase(main):013:0> get 'stu' ,'1001','info1:name'
COLUMN                        CELL                                                       
 info1:name                   timestamp=1577028678482, value=lisi                         
1 row(s) in 0.0840 seconds
```





获取info列族(指定到列)

```
hbase(main):014:0> get 'stu' ,'1001','info1'
COLUMN                        CELL                                                       
 info1:addr                   timestamp=1577028345949, value=beijing                     
 info1:name                   timestamp=1577028678482, value=lisi                         
 info1:sex                    timestamp=1577028337497, value=male                         
1 row(s) in 0.0340 seconds

```



#### **扫描查看表数据** 

```
hbase(main):002:0> scan 'stu'
ROW                           COLUMN+CELL                                                 
 1001                         column=info1:name, timestamp=1577025881287, value=zhnangsan 
1 row(s) in 5.8180 seconds
```



指定**ROW**的范围

区间：**左闭右开**

```
hbase(main):019:0> scan 'stu',{STARTROW=>'1001',STOPROW=>'1002'}
ROW                           COLUMN+CELL                                                 
 1001                         column=info1:addr, timestamp=1577028345949, value=beijing   
 1001                         column=info1:name, timestamp=1577028678482, value=lisi     
 1001                         column=info1:sex, timestamp=1577028337497, value=male       
 1001                         column=info2:addr, timestamp=1577028364249, value=shanghai 
1 row(s) in 0.0230 seconds
```

STARTROW=>'1001'：正无穷

STOPROW=>'1002'：负无穷





#### **改**

```
hbase(main):001:0> scan 'stu'
ROW                           COLUMN+CELL                                                 
 1001             column=info1:addr, timestamp=1577028345949, value=beijing   
 1001             column=info1:name, timestamp=1577028678482, value=lisi     
 1001             column=info1:sex, timestamp=1577028337497, value=male       
 1001             column=info2:addr, timestamp=1577028364249, value=shanghai 
 1002             column=info1:phone, timestamp=1577028358740, value=12341134143         
2 row(s) in 0.7320 seconds
//进行修改数据
hbase(main):002:0> put 'stu','1001','info1:name','wangwu'
0 row(s) in 0.1350 seconds

hbase(main):003:0> scan 'stu'
ROW                           COLUMN+CELL                                                 
 1001                         column=info1:addr, timestamp=1577028345949, value=beijing   
 1001                         column=info1:name, timestamp=1577029638957, value=wangwu   
 1001                         column=info1:sex, timestamp=1577028337497, value=male       
 1001                         column=info2:addr, timestamp=1577028364249, value=shanghai 
 1002                         column=info1:phone, timestamp=1577028358740, value=12341134143                      
2 row(s) in 0.0350 seconds

```

此时可以发现数据已经改变

可以使用scan进行查看10个版本以内的被修改的数据

此时的值依然时 保存在内存中的

```
hbase(main):005:0> scan 'stu',{RAW=>true,VERSIONS=>10}
ROW                           COLUMN+CELL                                                 
 1001           column=info1:addr, timestamp=1577028345949, value=beijing   
 1001           column=info1:name, timestamp=1577029638957, value=wangwu   
 1001           column=info1:name, timestamp=1577028678482, value=lisi     
 1001           column=info1:name, timestamp=1577028604442, value=zhnangsan 
 1001           column=info1:name, timestamp=1577028352834, value=lisi     
 1001           column=info1:name, timestamp=1577025881287, value=zhnangsan 
 1001           column=info1:sex, timestamp=1577028337497, value=male       
 1001           column=info2:addr, timestamp=1577028364249, value=shanghai 
 1002           column=info1:phone, timestamp=1577028358740, value=12341134143                      
2 row(s) in 0.0630 seconds

```

**put 'stu','1001','info1:name','wangwu',1577029638960**

手动传入时间戳，时间戳必须在其之后





#### **删除**

```
hbase(main):006:0> scan 'stu'
ROW                           COLUMN+CELL                                                 
 1001       column=info1:addr, timestamp=1577028345949, value=beijing   
 1001       column=info1:name, timestamp=1577029638957, value=wangwu   
 1001       column=info1:sex, timestamp=1577028337497, value=male       
 1001       column=info2:addr, timestamp=1577028364249, value=shanghai 
 1002       column=info1:phone, timestamp=1577028358740, value=12341134143               
2 row(s) in 0.0350 seconds

hbase(main):007:0> delete 'stu','1001','info1:sex'
0 row(s) in 0.0540 seconds

hbase(main):008:0> scan 'stu'
ROW                           COLUMN+CELL                                                 
 1001           column=info1:addr, timestamp=1577028345949, value=beijing   
 1001           column=info1:name, timestamp=1577029638957, value=wangwu   
 1001           column=info2:addr, timestamp=1577028364249, value=shanghai 
 1002           column=info1:phone, timestamp=1577028358740, value=12341134143           
2 row(s) in 0.0210 seconds
```



**同时也会将已经覆盖的数据进行删除**

```
hbase(main):009:0> delete 'stu','1001','info1:name'
0 row(s) in 0.0210 seconds

hbase(main):010:0> scan 'stu'
ROW                           COLUMN+CELL                                                 
 1001          column=info1:addr, timestamp=1577028345949, value=beijing   
 1001          column=info2:addr, timestamp=1577028364249, value=shanghai 
 1002          column=info1:phone, timestamp=1577028358740, value=12341134143             
2 row(s) in 0.0210 seconds

```



```
hbase(main):011:0>  scan 'stu',{RAW=>true,VERSIONS=>10}
ROW                           COLUMN+CELL                                                 
 1001             column=info1:addr, timestamp=1577028345949, value=beijing               
 1001             column=info1:name, timestamp=1577030242400, type=DeleteColumn           
 1001             column=info1:name, timestamp=1577029638957, value=wangwu   
 1001             column=info1:name, timestamp=1577028678482, value=lisi     
 1001             column=info1:name, timestamp=1577028604442, value=zhnangsan 
 1001             column=info1:name, timestamp=1577028352834, value=lisi     
 1001             column=info1:name, timestamp=1577025881287, value=zhnangsan 
 1001             column=info1:sex, timestamp=1577030073221, type=DeleteColumn           
 1001             column=info1:sex, timestamp=1577028337497, value=male       
 1001             column=info2:addr, timestamp=1577028364249, value=shanghai 
 1002             column=info1:phone, timestamp=1577028358740, value=12341134143 
```



此时添加新值为之间的时间戳

put 'stu','1001','info1:name','Mr_'，1577030242400

使用scan进行扫描时没有数据

此时被表记进行限制**DeleteColumn**





**1001       column=info1:sex, timestamp=1577028337497, value=male**   

delete 'stu','1001','info1:sex'，1577028337411

此时穿的时间戳比元数据的时间戳小，不会进行对数据的删除

delete 'stu','1001','info1:sex'，1577028337999

此时穿的时间戳比元数据的时间戳大，会进行对数据的删除



注意：指定到删除列族无法进行删除

delete 'stu','1001','info1'

命令行无法进行删除但是API可以进行删除



**删除ROWKEY**

delete：需要山三个参数

deleteall：删除一个rowkey

deleteall  'stu','1002'





**删表**

truncate  '表名'

提示：清空表的操作顺序为先 disable，然后再 truncate。



### **4、变更表信息** 

```
hbase(main):002:0> put 'stu','1005','info1:name','zhangsan'
0 row(s) in 0.2360 seconds

hbase(main):003:0> put 'stu','1005','info1:name','lisi'
0 row(s) in 0.0110 seconds

hbase(main):004:0> scan 'stu'
ROW                                      COLUMN+CELL                                                                                                          
 1001              column=info1:addr, timestamp=1577028345949, value=beijing             
 1001              column=info2:addr, timestamp=1577028364249, value=shanghai             
 1002              column=info1:phone, timestamp=1577028358740, value=12341134143         
 1005              column=info1:name, timestamp=1577114556573, value=lisi                                                               
3 row(s) in 0.0350 seconds
```



```
hbase(main):005:0> get 'stu','1005',{COLUMN=>'info1:name',VERSIONS=>3}
COLUMN                                   CELL                                                                                                                 
 info1:name                              timestamp=1577114556573, value=lisi                                                                                  
1 row(s) in 0.0760 seconds

```



将 info 列族中的数据存放 3 个版本：

```
hbase(main):022:0> alter 'student',{NAME=>'info',VERSIONS=>3}
hbase(main):022:0> get 
'student','1001',{COLUMN=>'info:name',VERSIONS=>3}
```

版本是HBase保留几个版本的



### 刷新

刷新命令：flush  '表名'