# 函数

## **7.1** **系统内置函数**

1）查看系统自带的函数

```
hive (default)> show functions;
OK
tab_name
!
!=
%
&
*
+
-
/
<
<=
<=>
<>
=
==
>
>=
^
abs
acos
add_months
and
array
array_contains
ascii
asin
assert_true
atan
avg
base64
between
bin
case
cbrt
ceil
ceiling
coalesce
collect_list
collect_set
compute_stats
concat
concat_ws
context_ngrams
conv
corr
cos
count
covar_pop
covar_samp
create_union
cume_dist
current_database
current_date
current_timestamp
current_user
date_add
date_format
date_sub
datediff
day
dayofmonth
decode
degrees
dense_rank
div
e
elt
encode
ewah_bitmap
ewah_bitmap_and
ewah_bitmap_empty
ewah_bitmap_or
exp
explode
factorial
field
find_in_set
first_value
floor
format_number
from_unixtime
from_utc_timestamp
get_json_object
greatest
hash
hex
histogram_numeric
hour
if
in
in_file
index
initcap
inline
instr
isnotnull
isnull
java_method
json_tuple
lag
last_day
last_value
lcase
lead
least
length
levenshtein
like
ln
locate
log
log10
log2
lower
lpad
ltrim
map
map_keys
map_values
matchpath
max
min
minute
month
months_between
named_struct
negative
next_day
ngrams
noop
noopstreaming
noopwithmap
noopwithmapstreaming
not
ntile
nvl
or
parse_url
parse_url_tuple
percent_rank
percentile
percentile_approx
pi
pmod
posexplode
positive
pow
power
printf
radians
rand
rank
reflect
reflect2
regexp
regexp_extract
regexp_replace
repeat
reverse
rlike
round
row_number
rpad
rtrim
second
sentences
shiftleft
shiftright
shiftrightunsigned
sign
sin
size
sort_array
soundex
space
split
sqrt
stack
std
stddev
stddev_pop
stddev_samp
str_to_map
struct
substr
substring
sum
tan
to_date
to_unix_timestamp
to_utc_timestamp
translate
trim
trunc
ucase
unbase64
unhex
unix_timestamp
upper
var_pop
var_samp
variance
weekofyear
when
windowingtablefunction
xpath
xpath_boolean
xpath_double
xpath_float
xpath_int
xpath_long
xpath_number
xpath_short
xpath_string
year
|
~
Time taken: 1.774 seconds, Fetched: 216 row(s)
hive (default)> 

```



2）显示自带的函数的用法

```
hive (default)> desc function trim;
OK
tab_name
trim(str) - Removes the leading and trailing space characters from str 
Time taken: 0.058 seconds, Fetched: 1 row(s)
```



3）详细显示自带的函数的用法

```
hive (default)> desc function extended trim;
OK
tab_name
trim(str) - Removes the leading and trailing space characters from str 
Example:
  > SELECT trim('   facebook  ') FROM src LIMIT 1;
  'facebook'
Time taken: 0.023 seconds, Fetched: 4 row(s)
```





## **7.2** **自定义函数**

1）Hive 自带了一些函数，比如：max/min 等，但是数量有限，自己可以通过自定义 UDF

来方便的扩展。



2）当 Hive 提供的内置函数无法满足你的业务处理需要时，此时就可以考虑使用用户自定义 

函数（UDF：user-defined function）。 



3）根据用户自定义函数类别分为以下三种：

```
（1）UDF（User-Defined-Function）
		一进一出
（2）UDAF（User-Defined Aggregation Function）
		聚集函数，多进一出
		类似于：count/max/min
（3）UDTF（User-Defined Table-Generating Functions）
		一进多出
```



4）官方文档地址 

https://cwiki.apache.org/confluence/display/Hive/HivePlugins



5）编程步骤



（1）继承 org.apache.hadoop.hive.ql.UDF 

（2）需要实现 evaluate 函数；evaluate 函数支持重载； 

（3）在 hive 的命令行窗口创建函数

​			a）添加 jar

```
add jar linux_jar_path
```

​			b）创建 function

```
create [temporary] function [dbname.]function_name AS 
class_name;
```

（4）在 hive 的命令行窗口删除函数



6）注意事项

**UDF 必须要有返回类型，可以返回 null，但是返回类型不能为 void；**



### **7.2.1** **自定义** **UDF** **函数**



1）创建一个 Maven 工程 Hive 



2）导入依赖 

```
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
        <version>1.2.1</version>
      </dependency>
```



3）创建一个类

evaluate方法名必须是这个

```
package hive;
import org.apache.hadoop.hive.ql.exec.UDF;

import java.io.UTFDataFormatException;


public class MyUDF extends UDF {
    public int evaluate(int data){

        return  data + 10;
    }
}

```



4）打成 jar 包上传到服务器/opt/module/hive/lib/hive-1.0-SNAPSHOT.jar

可以放置任意目录



5）将 jar 包添加到 hive 的 classpath

```
 add jar /opt/module/hive/lib/hive-1.0-SNAPSHOT.jar
```



6）创建临时函数与开发好的 java class 关联

as 全类名

```
hive (default)> create function addTen as 'hive.MyUDF';
```



7．即可在 hql 中使用自定义的函数

```
hive (default)> select * from aa;
OK
aa.id
1
2
3
4
5
6
7
8
Time taken: 27.059 seconds, Fetched: 8 row(s)
hive (default)> select addTen(id) from aa;
OK
_c0
11
12
13
14
15
16
17
18

```





注意：

1、此时自定义的函数只能再当前库中使用

2、可以使用  库名.函数名  

3、在文件中  使用-e  -f  等参数的时候 需要使用  库名.函数名，否则默认找default库

4、不适用temporary创建的是临时库，一旦窗口退出就会消失

5、jar的位置在lib下回进行重新加载，不需要重新进行add，存放在其他位置需要再次add

6、函数的方法名必须是evaluate

7、方法的重载可以进行进行不同参数的方法的使用





### **7.2.2** **自定义** **UDTF** **函数**

1）需求说明 

自定义一个 UDTF 实现将一个任意分割符的字符串切割成独立的单词，例如：

```
Line:"hello,world,hadoop,hive"

Myudtf(line, ",") 

hello
world
hadoop
hive
```



2）代码实现

```
package hive;

import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.ql.udf.generic.GenericUDTF;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.StructObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;

import java.util.ArrayList;
import java.util.List;

public class MyUdtf extends GenericUDTF {

    private List<String> dataList = new ArrayList<>();


    //原方法直接抛异常
    //做类型校验


    //初始化
    //定义输出数据的列名和数据类型
    //GenericUDTFJSONTuple类的initialize方法
    //return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames, fieldOIs);
    @Override
    public StructObjectInspector initialize(StructObjectInspector argOIs) throws UDFArgumentException {

        //fieldNames：定义输出数据的列名和类型
        //fieldOIs：添加输出数据的列名和类型
        //可以炸裂成多个列
        List<ObjectInspector> fieldOIs =  new ArrayList<>();

        List<String> fieldNames = new ArrayList<>();

        fieldNames.add("word");
        //类型是当前函数的返回值
        fieldOIs.add(PrimitiveObjectInspectorFactory.javaStringObjectInspector);

        return ObjectInspectorFactory.getStandardStructObjectInspector(fieldNames, fieldOIs);
    }

    @Override
    public void process(Object[] objects) throws HiveException {
        //获取数据
        String data = objects[0].toString();

        //获取分隔符
        String splitKey = objects[1].toString();

        //切分数据
        String[] words = data.split(splitKey);

        //写出
        //写出方法再父类中
        //protected final void forward(Object o) throws HiveException {
        //   this.collector.collect(o);
        //}
        for (String word : words) {

            //需要进行清空，即可实现一进多出
            dataList.clear();
            //将数据放到集合
            dataList.add(word);

            //写出数据的操作
            //写出的数据类型可能不匹配
            //定义一个全局的集合
            forward(dataList);
        }
    }

    @Override
    public void close() throws HiveException {
        //有io操作时候 需要使用close方法进行释放资源
    }
}

```



3）打成 jar 包上传到服务器/opt/module/hive/lib/udtf.jar



4）将 jar 包添加到 hive 的 classpath 下

```
hive (default)> add jar /opt/module/hive/lib/udtf.jar
              > ;
Added [/opt/module/hive/lib/udtf.jar] to class path
Added resources: [/opt/module/hive/lib/udtf.jar]
```



5）创建函数与开发好的 java class 关联

```
hive (default)> create function myudtf as 'hive.MyUdtf'
              > ;
OK
Time taken: 1.51 seconds
```



6）即可在 hql 中使用自定义的函数

```
hive (default)> select myudtf('hello,word,mechengs',',');
OK
word
hello
word
mechengs
Time taken: 32.827 seconds, Fetched: 3 row(s)
```





