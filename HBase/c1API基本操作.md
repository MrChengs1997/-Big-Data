## **环境**

新建maven工程

依赖

```xml
<dependency>
	 <groupId>org.apache.hbase</groupId>
	 <artifactId>hbase-server</artifactId>
	 <version>1.3.1</version>
</dependency>
<dependency>
 	<groupId>org.apache.hbase</groupId>
 	<artifactId>hbase-client</artifactId>
 	<version>1.3.1</version>
</dependency>
```



# 

## **DDL**



### **获取** **Configuration** 对象

已经过时的连接方法

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.HBaseAdmin;

	@Test
    public void contect() throws IOException {

        //获取集群配置信息
        HBaseConfiguration configuration = new HBaseConfiguration();
        configuration.set("hbase.zookeeper.quorum", "192.168.199.186,192.168.199.224");//连接zk的配置
        configuration.set("hbase.zookeeper.property.clientPort", "2181");//zk的端口
        //获取管理员对象
        HBaseAdmin admin = new HBaseAdmin(configuration);
        //判断是否存在
        admin.tableExists("stu5");
        //关闭连接
        admin.close();
    }
```

//hbase.zookeeper.quorum:来源于conf文件目录下的hbase-site.xml中的配置



新的连接方法

```java
   //获取集群配置信息
        Configuration configuration =  HBaseConfiguration.create();
        configuration.set("hbase.zookeeper.quorum", "192.168.199.186,192.168.199.224");//连接zk的配置
        configuration.set("hbase.zookeeper.property.clientPort", "2181");//zk的端口

        //获取管理员对象
        //HBaseAdmin admin = new HBaseAdmin(configuration);
        Connection connection = ConnectionFactory.createConnection(configuration);
        Admin admin = connection.getAdmin();
        //判断是否存在
        System.out.println(admin.tableExists(TableName.valueOf("stu5")));;//TableName对象
        //关闭连接
        admin.close();
```



```
true
```



### 判断表是否存在

```java
@Test
    public void contect() throws IOException {

        //获取集群配置信息
        HBaseConfiguration configuration = new HBaseConfiguration();
        configuration.set("hbase.zookeeper.quorum", "192.168.199.186,192.168.199.224");//连接zk的配置
        configuration.set("hbase.zookeeper.property.clientPort", "2181");//zk的端口

        //获取管理员对象
        HBaseAdmin admin = new HBaseAdmin(configuration);

        //判断是否存在
        System.out.println(admin.tableExists("stu5"));;

        //关闭连接
        admin.close();
    }
```



```reStructuredText
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
true
```



代码整合：

```java
    private static Connection connection;
    private static Admin admin;
    static{
      try {
          //1获取集群配置信息
          Configuration configuration =  HBaseConfiguration.create();
          configuration.set("hbase.zookeeper.quorum", "192.168.199.186,192.168.199.224");//连接zk的配置
          configuration.set("hbase.zookeeper.property.clientPort", "2181");//zk的端口
          //2获取管理员对象
           connection = ConnectionFactory.createConnection(configuration);
           admin = connection.getAdmin();
      }catch (Exception e){
      }
    }

 //关闭操作
    public  static  void close(){
        if (admin != null){
            try {
                admin.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (connection != null){
            try {
                connection.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }


    public static void main(String[] args) throws IOException {
        //判断是否存在
        System.out.println(admin.tableExists(TableName.valueOf("stu5")));;//TableName对象
        //关闭连接
        close();
    }
```





### 创建表

```java
    //创建表
    public  static void createTable(String tableName,String ... args) throws IOException {
        //1、判断是否存在列族信息
        if (args.length <=0){
            System.out.println("没有列族");
            //为设置列族信息
            return;
        }
        //2、判断表是否已经存在
        if (admin.tableExists(TableName.valueOf(tableName))){
            System.out.println("表已经存在");
            //表已经存在
            return;
        }

        //3、创建表
        //HTableDescriptor表描述
        //  byte[][] splitKeys：预分区

        //创建表描述器
        HTableDescriptor hTableDescriptor = new HTableDescriptor(TableName.valueOf(tableName));

        //循环添加列族信息
        for (String val :args){
            //创建列族描述器
            HColumnDescriptor hColumnDescriptor = new HColumnDescriptor(val);
            //添加列族信息
            hTableDescriptor.addFamily(hColumnDescriptor);
        }

        //创建表
        admin.createTable(hTableDescriptor);
    }
```



```java
        //创建表
        System.out.println(admin.tableExists(TableName.valueOf("stu6")));;//TableName对象
        createTable("stu6","info1","info2");
        System.out.println(admin.tableExists(TableName.valueOf("stu6")));;//TableName对象
```



```
false
true
```



### 删除表

```java
    //删除表
    public  static  void dropTable(String tableName) throws IOException {

        //判断表是否存在
        if (! admin.tableExists(TableName.valueOf(tableName))){
            System.out.println("表不存在");
            //表已经存在
            return;
        }

        //表下线
        admin.disableTable(TableName.valueOf(tableName));

        //删除表
        admin.deleteTable(TableName.valueOf(tableName));
        System.out.println("delete  table");
    }
```



```java
   //删除表
   dropTable("stu6");
```





### 创建命名空间

没有方法进行判断命名空间是否存在

命名空间如果存在会进行报错：NameSpaceExistException	

使用捕捉异常的方式进行判断是否存在命名空间



```java
   //创建命名空间
    public  static  void  createNameSpace(String name){
        //创建命名空间描述器
        NamespaceDescriptor build = NamespaceDescriptor.create(name).build();

        //创建命名空间
        try {
            admin.createNamespace(build);
        } catch (NamespaceExistException e1){
            e1.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
```



```
  //创建命名空间
        createNameSpace("mrchengs");
```



```shell
hbase(main):001:0> list_namespace
NAMESPACE                                                                  
default                                                                  
hbase                                                                  
2 row(s) in 1.4200 seconds

hbase(main):002:0> list_namespace
NAMESPACE                                                                                                                                                     
default                                                                  
hbase                                                                        
mrchengs                                                                                 
3 row(s) in 0.0310 seconds
```



### 在命名空间下创建表

```java
  //创建表
    public  static void createTable(String tableName,String ... args) throws IOException {
        //1、判断是否存在列族信息
        if (args.length <=0){
            System.out.println("没有列族");
            //为设置列族信息
            return;
        }
        //2、判断表是否已经存在
        if (admin.tableExists(TableName.valueOf(tableName))){
            System.out.println("表已经存在");
            //表已经存在
            return;
        }

        //3、创建表
        //HTableDescriptor表描述
        //  byte[][] splitKeys：预分区

        //创建表描述器
        HTableDescriptor hTableDescriptor = new HTableDescriptor(TableName.valueOf(tableName));

        //循环添加列族信息
        for (String val :args){
            //创建列族描述器
            HColumnDescriptor hColumnDescriptor = new HColumnDescriptor(val);
            //添加列族信息
            hTableDescriptor.addFamily(hColumnDescriptor);
        }

        //创建表
        admin.createTable(hTableDescriptor);
    }
```



```java
 createTable("mrchengs:stu6","info1","info2");
```



```shell
hbase(main):003:0> list
TABLE                                                                       
mrchengs:stu6                                                                     
stu                                                                   
stu4                                                                      
stu5                                                                       
student                                                                       
5 row(s) in 0.1350 seconds
=> ["mrchengs:stu6", "stu", "stu4", "stu5", "student"]

```



## dml

### 向表插入数据



```java
 public static  void putDate(String tableName,String rowKey,String cf,String cn,String val) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建put对象
        //rg.apache.hadoop.hbase.util：将普通类型转为字节数据，反之亦然
        Put put = new Put(Bytes.toBytes(rowKey));

        //给put对象赋值
        put.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn),Bytes.toBytes(val));

        //插入数据的方法
        table.put(put);

        //关闭连接
        table.close();
    }
```



```java
//添加数据
//put 'stu','1001','info1:name','zhnangsan'
putDate("stu","1010","info2","names","Mrchengs");
```



```
hbase(main):011:0>  scan 'stu'
ROW                                      COLUMN+CELL                            
 1001          column=info1:addr, timestamp=1577028345949, value=beijing   
 1001          column=info2:addr, timestamp=1577028364249, value=shanghai  
 1002          column=info1:phone, timestamp=1577028358740, value=12341134143  
 1003          column=info2:age, timestamp=1581845211688, value=13    
 1003          column=info2:name, timestamp=1581845174744, value=mingming  
 1003          column=info2:sex, timestamp=1581845226241, value=boy   
 1003          column=info2:sex1, timestamp=1581845282716, value=boy     
 1005          column=info1:name, timestamp=1577114556573, value=lisi     
4 row(s) in 0.0580 seconds

hbase(main):012:0>  scan 'stu'
ROW                                      COLUMN+CELL     
 1001            column=info1:addr, timestamp=1577028345949, value=beijing   
 1001           column=info2:addr, timestamp=1577028364249, value=shanghai   
 1002           column=info1:phone, timestamp=1577028358740, value=12341134143      
 1003           column=info2:age, timestamp=1581845211688, value=13   
 1003           column=info2:name, timestamp=1581845174744, value=mingming       
 1003          column=info2:sex, timestamp=1581845226241, value=boy         
 1003         column=info2:sex1, timestamp=1581845282716, value=boy   
 1005         column=info1:name, timestamp=1577114556573, value=lisi               
 1010         column=info2:names, timestamp=1582120919156, value=Mrchengs 
5 row(s) in 0.7680 seconds
```



为同一个rowkey添加多个列1:

```
        //给put对象赋值
        put.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn),Bytes.toBytes(val));
     	//
     	//put.addColumn(Bytes.toBytes(cf),Bytes.toBytes("xxx"),Bytes.toBytes("xxx"));
```

查询时就会出现同一个rowkey和两个列数据



添加多个列方式2:

```
void put(List<Put> puts) throws IOException;
```



命名空间中的表插入数据

```java
        //添加数据
        putDate("mrchengs:stu6","1010","info2","names","Mrchengs");
```



### 获取数据

#### get

创造查询数据

```
 1003      column=info1:city, timestamp=1582177663712, value=beijing      
 1003      column=info2:age, timestamp=1581845211688, value=13           
 1003      column=info2:name, timestamp=1581845174744, value=mingming 
 1003      column=info2:sex, timestamp=1581845226241, value=boy 
 1003      column=info2:sex1, timestamp=1581845282716, value=boy   
```



查询一：

```java
    //获取数据
    public  static  void  getDate(String tableName,String rowKey,String cn ,String cf) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建Get
        Get get = new Get(Bytes.toBytes(rowKey));

        //传递集合返回集合
        //传递单个对象，返回单个对象
        //Result[] get(List<Get> gets) throws IOException;
        //Result get(Get get) throws IOException;
        Result result = table.get(get);

        //解析result
       for (Cell cell : result.rawCells()){
           System.out.println("CF:"+Bytes.toString(CellUtil.cloneFamily(cell)));
           System.out.println("CN:"+Bytes.toString(CellUtil.cloneQualifier(cell)));
           System.out.println("VAL:"+Bytes.toString(CellUtil.cloneValue(cell)));
       }

       //关闭
        table.close();
    };
```



```java
//cn,cf值并未使用
getDate("stu","1003","info2","info");
```

未指定查询的列族，会将所有的rowKey的值都查询出

等同于：hbase(main):003:0> get 'stu','1003'

```
CF:info1
CN:city
VAL:beijing

CF:info2
CN:age
VAL:13

CF:info2
CN:name
VAL:mingming

CF:info2
CN:sex
VAL:boy

CF:info2
CN:sex1
VAL:boy
```



查询二：指定查询的列族

类似于hbase(main):003:0> get 'stu','1003'，'info1'

```java
    public static  void putDate(String tableName,String rowKey,String cf,String cn,String val) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建put对象
        //rg.apache.hadoop.hbase.util：将普通类型转为字节数据，反之亦然
        Put put = new Put(Bytes.toBytes(rowKey));

        //给put对象赋值
        put.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn),Bytes.toBytes(val));

        //插入数据的方法
        table.put(put);
        //table.put();

        //关闭连接
        table.close();
    }

    //获取数据
    public  static  void  getDate(String tableName,String rowKey,String cf ,String cn) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建Get
        Get get = new Get(Bytes.toBytes(rowKey));

        //执行获取的列族
        get.addFamily(Bytes.toBytes(cf));//执行查询的列族

        //传递集合返回集合
        //传递单个对象，返回单个对象
        //Result[] get(List<Get> gets) throws IOException;
        //Result get(Get get) throws IOException;
        Result result = table.get(get);

        //解析result
       for (Cell cell : result.rawCells()){
           System.out.println("CF:"+Bytes.toString(CellUtil.cloneFamily(cell)));
           System.out.println("CN:"+Bytes.toString(CellUtil.cloneQualifier(cell)));
           System.out.println("VAL:"+Bytes.toString(CellUtil.cloneValue(cell)));
           System.out.println();
       }

       //关闭
        table.close();
    }
```



```java
        getDate("stu","1003","info1","info");
```



```
CF:info1
CN:city
VAL:beijing
```



查询三：执行列族和列

hbase(main):003:0> get 'stu','1003'，'info2：name'



```java
    //获取数据
    public  static  void  getDate(String tableName,String rowKey,String cf ,String cn) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建Get
        Get get = new Get(Bytes.toBytes(rowKey));

        //执行获取的列族
        //get.addFamily(Bytes.toBytes(cf));//执行查询的列族
        get.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn));//指定列族&列

        //传递集合返回集合
        //传递单个对象，返回单个对象
        //Result[] get(List<Get> gets) throws IOException;
        //Result get(Get get) throws IOException;
        Result result = table.get(get);

        //解析result
       for (Cell cell : result.rawCells()){
           System.out.println("CF:"+Bytes.toString(CellUtil.cloneFamily(cell)));
           System.out.println("CN:"+Bytes.toString(CellUtil.cloneQualifier(cell)));
           System.out.println("VAL:"+Bytes.toString(CellUtil.cloneValue(cell)));
           System.out.println();
       }

       //关闭
        table.close();
    }
```



```java
 getDate("stu","1003","info2","sex");
```



```
CF:info2
CN:sex
VAL:boy
```





查询三：获取数据的版本数

此时1001的name是已经修改过的数据

```
hbase(main):014:0> scan 'stu5',{RAW=>TRUE,VERSIONS=>10}
ROW                                      COLUMN+CELL 
 1001          column=info:name, timestamp=1582179003853, value=Mrchegns
 1001          column=info:name, timestamp=1581848086058, value=qq
 1002          column=info:name, timestamp=1581848198318, value=ww 
 1003          column=info:name, timestamp=1581848211676, value=ss 
 1004          column=info:name, timestamp=1581848291880, value=xx
 1006          column=info:name, timestamp=1581850829665, value=mrchengs1
 1007          column=info:name, timestamp=1581850846275, value=mrchengs5 
```



```shell
hbase(main):015:0> describe 'stu5'
Table stu5 is ENABLED                                                       
stu5                                                                         
COLUMN FAMILIES DESCRIPTION     
{NAME => 'info', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', 
COMPRESSION => 'NONE', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}  
1 row(s) in 0.1360 seconds

```

VERSIONS => '1':在API查询中最大的查询版本数也是1



```java
 //获取数据
    public  static  void  getDate(String tableName,String rowKey,String cf ,String cn) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建Get
        Get get = new Get(Bytes.toBytes(rowKey));

        //执行获取的列族
        //get.addFamily(Bytes.toBytes(cf));//执行查询的列族
        get.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn));//指定列族&列
        //设置版本数
        get.setMaxVersions();

        //传递集合返回集合
        //传递单个对象，返回单个对象
        //Result[] get(List<Get> gets) throws IOException;
        //Result get(Get get) throws IOException;
        Result result = table.get(get);

        //解析result
       for (Cell cell : result.rawCells()){
           System.out.println("CF:"+Bytes.toString(CellUtil.cloneFamily(cell)));
           System.out.println("CN:"+Bytes.toString(CellUtil.cloneQualifier(cell)));
           System.out.println("VAL:"+Bytes.toString(CellUtil.cloneValue(cell)));
           System.out.println();
       }

       //关闭
        table.close();
    }
```



```java
 getDate("stu5","1001","info","name");
```



```
CF:info
CN:name
VAL:Mrchegns
```



#### scan

数据：

```
hbase(main):017:0> scan 'stu'
ROW                                      COLUMN+CELL                      
 1001          column=info1:addr, timestamp=1577028345949, value=beijing  
 1001          column=info2:addr, timestamp=1577028364249, value=shanghai  
 1002          column=info1:phone, timestamp=1577028358740, value=12341134143  
 1003          column=info1:city, timestamp=1582177663712, value=beijing 
 1003          column=info2:age, timestamp=1581845211688, value=13    
 1003          column=info2:name, timestamp=1581845174744, value=mingming 
 1003          column=info2:sex, timestamp=1581845226241, value=boy  
 1003          column=info2:sex1, timestamp=1581845282716, value=boy  
 1005          column=info1:name, timestamp=1577114556573, value=lisi 
 1010          column=info2:names, timestamp=1582120919156, value=Mrchengs                                                          
5 row(s) in 1.1360 seconds

```



查询一：扫描全表

```java
public static  void scanTable(String tableName) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建scan对象
        //空参数扫描全表
        Scan scan = new Scan();

        //扫描表
        ResultScanner scanner = table.getScanner(scan);

        //解析ResultScanner
        for (Result result:scanner){
            //解析result
            for (Cell cell:result.rawCells()){
                System.out.println("CF:"+Bytes.toString(CellUtil.cloneFamily(cell)));
                System.out.println("CN:"+Bytes.toString(CellUtil.cloneQualifier(cell)));
                System.out.println("VAL:"+Bytes.toString(CellUtil.cloneValue(cell)));
                System.out.println();
            }
        }

        //关闭
        table.close();
    }
```



```java
scanTable("stu");
```



```
CF:info1
CN:addr
VAL:beijing

CF:info2
CN:addr
VAL:shanghai

CF:info1
CN:phone
VAL:12341134143

CF:info1
CN:city
VAL:beijing

CF:info2
CN:age
VAL:13

CF:info2
CN:name
VAL:mingming

CF:info2
CN:sex
VAL:boy

CF:info2
CN:sex1
VAL:boy

CF:info1
CN:name
VAL:lisi

CF:info2
CN:names
VAL:Mrchengs
```





查询二：Scan(byte [] startRow, byte [] stopRow)

```java
    public static  void scanTable(String tableName) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建scan对象
        //空参数扫描全表
        Scan scan = new Scan(Bytes.toBytes("1001"),Bytes.toBytes("1003"));

        //扫描表
        ResultScanner scanner = table.getScanner(scan);

        //解析ResultScanner
        for (Result result:scanner){
            //解析result
            for (Cell cell:result.rawCells()){
                System.out.println("rowkey:" + Bytes.toString(CellUtil.cloneRow(cell)));
                System.out.println("CF:"+Bytes.toString(CellUtil.cloneFamily(cell)));
                System.out.println("CN:"+Bytes.toString(CellUtil.cloneQualifier(cell)));
                System.out.println("VAL:"+Bytes.toString(CellUtil.cloneValue(cell)));
                System.out.println();
            }
        }

        //关闭
        table.close();
    }
```



```
 scanTable("stu");
```



```
rowkey:1001
CF:info1
CN:addr
VAL:beijing

rowkey:1001
CF:info2
CN:addr
VAL:shanghai

rowkey:1002
CF:info1
CN:phone
VAL:12341134143
```





### 删除

delete命令：指定到cn，删除标记：DeleteColumn

deleteall命令：指定到rowkey，删除标记：DeleteFamily



删除一：deleteall：值传递rowkey

```java
    //删除
    public static  void deleteData(String tableName,String rowKey,String cf,String cn) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建delete对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));

        //删除
        table.delete(delete);

        //关闭
        table.close();
    }
```



```
 deleteData("stu","1010","","");
```



```
hbase(main):020:0> scan 'stu',{RAW=>TRUE,VERSIONS=>10}

1010     column=info1:, timestamp=1582188570081, type=DeleteFamily   
1010     column=info2:, timestamp=1582188570081, type=DeleteFamily           
1010     column=info2:names, timestamp=1582120919156, value=Mrchengs     
```





删除二：删除指定的列

```java
 //删除
    public static  void deleteData(String tableName,String rowKey,String cf,String cn) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建delete对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        //设置删除的列
        delete.addColumns(Bytes.toBytes(cf),Bytes.toBytes(cn));

        //删除
        table.delete(delete);

        //关闭
        table.close();
    }

```

结果：DeleteColumn



```java
  //删除
    public static  void deleteData(String tableName,String rowKey,String cf,String cn) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建delete对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        //设置删除的列
        delete.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn));

        //删除
        table.delete(delete);

        //关闭
        table.close();
    }
```

如果数据被修改过

在执行之后，会删除最新的数据，上一个数据会被查询查到（顶替原有最新的数据作为数据）



在文件合并合并之后，会进行删除指定的数据，不会被上一条数据顶替



删除三：时间戳

删除当前时间戳的标记

删除标记：delete

```java
 public static  void deleteData(String tableName,String rowKey,String cf,String cn) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建delete对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        //设置删除的列
        delete.addColumn(Bytes.toBytes(cf),Bytes.toBytes(cn),1577028345949L);

        //删除
        table.delete(delete);

        //关闭
        table.close();
    }

```



删除四：删除指定的列族

```java
  //删除
    public static  void deleteData(String tableName,String rowKey,String cf,String cn) throws IOException {
        //获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));

        //创建delete对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        //设置删除的列
        delete.addFamily(Bytes.toBytes(cf));
        
        //删除
        table.delete(delete);

        //关闭
        table.close();
    }

```

删除的标记：deletefamily