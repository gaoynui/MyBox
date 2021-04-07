## HIVE

------

#### 一、介绍

​	**1、Hive 由 Facebook 实现并开源**

​	**2、是基于 Hadoop 的一个数据仓库工具**

​	**3、可以将结构化的数据映射为一张数据库表**

​	**4、并提供 HQL(Hive SQL)查询功能**

​	**5、底层数据是存储在 HDFS 上**

​	**6、Hive的本质是将 SQL 语句转换为 MapReduce 任务运行**

​	**7、使不熟悉 MapReduce 的用户很方便地利用 HQL 处理和计算 HDFS 上的结构化的数据，适用于离线的批量数据计算。**

```
优点：
1.可扩展性：横向扩展，可自由扩展集群规模，不需要重启服务。
2.延展性：UDF
3.容错性：即使有节点出现问题，SQL语句仍可完成执行
缺点：
1.不支持记录级别的增删改查，可按分区或插入新表
2.查询延迟严重，mapReduce Job的启动需要很长时间，不能用在交互系统中
3.不支持事务

数据库事务：
A给B转账1W元分两步：
从A账户里取出1W元；将1W元存入B账户
这两步是一个事务，如果某一步未完成则全部事务取消。
hive偏向分析处理(OLAP)，而非事务处理
```

#### 二、架构

​	<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\hive\hive架构.png" style="zoom:67%;" />

​	1.用户接口：

​		CLI/shell，jdbc/odbc(微软)，web

​	2.跨语言服务(thrift server)：

​		让用户使用多种不同的语言来操纵hive

​	3.驱动器Driver

​		(1)解释器(Interpreter):将HQL语句转换为抽象语法树(AST)

​		(2)编译器(Complier):将语法树编译为逻辑执行计划

​		(3)优化器(Optimizer):对逻辑执行计划进行优化

​		(4)执行器(Executor):调用底层运行框架执行逻辑执行计划

​	4.元数据存储系统

​		元数据：表的名字，表的列和分区及其属性，表的属性（内部表和 外部表），表的数据所在目录

​		元数据常常通过metastore存储到mysql中。use metastore

#### 三、SQL

​		1.window子句:

```
原始数据：
cookieid,createtime,pv
cookie1,2015-04-10,1
cookie1,2015-04-11,5
cookie1,2015-04-12,7
cookie1,2015-04-13,3
cookie1,2015-04-14,2
cookie1,2015-04-15,4
cookie1,2015-04-16,4

查询语句：
select 
   cookieid, 
   createtime, 
   pv, 
   sum(pv) over (partition by cookieid order by createtime rows between unbounded preceding and current row) as pv1, 
   sum(pv) over (partition by cookieid order by createtime) as pv2, 
   sum(pv) over (partition by cookieid) as pv3, 
   sum(pv) over (partition by cookieid order by createtime rows between 3 preceding and current row) as pv4, 
   sum(pv) over (partition by cookieid order by createtime rows between 3 preceding and 1 following) as pv5, 
   sum(pv) over (partition by cookieid order by createtime rows between current row and unbounded following) as pv6 
from cookie1;

查询结果：
cookieid,createtime,pv,pv1,pv2,pv3,pv4,pv5,pv6
cookie1,2015-04-16,4,26,26,26,13,13,4
cookie1,2015-04-15,4,22,22,26,16,20,8
cookie1,2015-04-14,2,18,18,26,17,21,10
cookie1,2015-04-13,3,16,16,26,16,18,13
cookie1,2015-04-12,7,13,13,26,13,16,20
cookie1,2015-04-11,5,6,6,26,6,13,25
cookie1,2015-04-10,1,1,1,26,1,6,26

解析：
pv1: 分组内从起点到当前行的pv累积，如，11号的pv1=10号的pv+11号的pv, 12号=10号+11号+12号
pv2: 同pv1
pv3: 分组内(cookie1)所有的pv累加
pv4: 分组内当前行+往前3行，如，11号=10号+11号， 12号=10号+11号+12号， 13号=10号+11号+12号+13号， 14号=11号+12号+13号+14号
pv5: 分组内当前行+往前3行+往后1行，如，14号=11号+12号+13号+14号+15号=5+7+3+2+4=21
pv6: 分组内当前行+往后所有行，如，13号=13号+14号+15号+16号=3+2+4+4=13，14号=14号+15号+16号=2+4+4=10

window子句：
PRECEDING：往前
FOLLOWING：往后
CURRENT ROW：当前行
UNBOUNDED：起点，
UNBOUNDED PRECEDING 表示从前面的起点，
UNBOUNDED FOLLOWING：表示到后面的终点
```

​	2.NTILE(n)：

​		用于将分组数据按照顺序切分成n片，返回当前切片值，如果切片不均匀，默认增加第一个切片的分布。结果相同表示分在一片

```
原始数据：
cookie1,2015-04-10,1
cookie1,2015-04-11,5
cookie1,2015-04-12,7
cookie1,2015-04-13,3
cookie1,2015-04-14,2
cookie1,2015-04-15,4
cookie1,2015-04-16,4
cookie2,2015-04-10,2
cookie2,2015-04-11,3
cookie2,2015-04-12,5
cookie2,2015-04-13,6
cookie2,2015-04-14,3
cookie2,2015-04-15,9
cookie2,2015-04-16,7

查询语句：
select
  cookieid,
  createtime,
  pv,
  ntile(2) over (partition by cookieid order by createtime) as rn1, --分组内将数据分成2片
  ntile(3) over (partition by cookieid order by createtime) as rn2, --分组内将数据分成3片
  ntile(4) over (order by createtime) as rn3 --将所有数据分成4片
from cookie.cookie2 
order by cookieid,createtime;

查询结果：
cookieid,createtime,pv,rn1,rn2,rn3
cookie1,2015-04-10,1,1,1,1
cookie1,2015-04-11,5,1,1,1
cookie1,2015-04-12,7,1,1,2
cookie1,2015-04-13,3,1,2,2
cookie1,2015-04-14,2,2,2,3
cookie1,2015-04-15,4,2,3,4
cookie1,2015-04-16,4,2,3,4
cookie2,2015-04-10,2,1,1,1
cookie2,2015-04-11,3,1,1,1
cookie2,2015-04-12,5,1,1,2
cookie2,2015-04-13,6,1,2,2
cookie2,2015-04-14,3,2,2,3
cookie2,2015-04-15,9,2,3,3
cookie2,2015-04-16,7,2,3,4
```

​	3.ROW_NUMBER()

​		从1开始，按照顺序，生成分组内记录的序列

```
查询语句：
select
  cookieid,
  createtime,
  pv,
  row_number() over (partition by cookieid order by pv desc) as rn
from cookie.cookie2;

查询结果：
rn
1
2
3
...
```

​	4.RANK()/DENSE_RANK()

​		生成数据项在分组中的排名，排名相等会在名次中留下空位/不会留下空位

```
查询语句：
select
  cookieid,
  createtime,
  pv,
  rank() over (partition by cookieid order by pv desc) as rn1,
  dense_rank() over (partition by cookieid order by pv desc) as rn2,
  row_number() over (partition by cookieid order by pv desc) as rn3
from cookie.cookie2 
where cookieid='cookie1';

查询结果：
cookieid,createtime,pv,rn1,rn2,rn3
cookie1,2015-04-12,7,1,1,1
cookie1,2015-04-11,5,2,2,2
cookie1,2015-04-16,4,3,3,3
cookie1,2015-04-15,4,3,3,4
cookie1,2015-04-13,3,5,4,5
cookie1,2015-04-14,2,6,5,6
cookie1,2015-04-10,1,7,6,7

区别：
row_number： 按顺序编号，不留空位
rank： 按顺序编号，相同的值编相同号，留空位
dense_rank： 按顺序编号，相同的值编相同的号，不留空位
```

​	5.CUME_DIST()

​		小于等于当前值的行数/分组内总行数

```
原始数据：
d1,user1,1000
d1,user2,2000
d1,user3,3000
d2,user4,4000
d2,user5,5000

查询语句：
select 
  dept,
  userid,
  sal,
  cume_dist() over (order by sal) as rn1,
  cume_dist() over (partition by dept order by sal) as rn2
from cookie.cookie3;

查询结果：
dept,userid,sal,rn1,rn2
d1,user1,1000,0.2,0.333
d1,user2,2000,0.4,0.666
d1,user3,3000,0.6,1.0
d2,user4,4000,0.8,0.5
d2,user5,5000,1.0,1.0
```

​	6.PERCENT_RANK()

​		分组内当前行的RANK值-1/分组内总行数-1

```
原始数据：
d1,user1,1000
d1,user2,2000
d1,user3,3000
d2,user4,4000
d2,user5,5000

查询语句：
select 
  dept,
  userid,
  sal,
  percent_rank() over (order by sal) as rn1, --分组内
  rank() over (order by sal) as rn11, --分组内的rank值
  sum(1) over (partition by null) as rn12, --分组内总行数
  percent_rank() over (partition by dept order by sal) as rn2,
  rank() over (partition by dept order by sal) as rn21,
  sum(1) over (partition by dept) as rn22 
from cookie.cookie3;

查询结果：
dept,userid,sal,rn1,rn11,rn12,rn2,rn21,rn22
d1,user3,3000,0.5,3,5,1.0,3,3
d1,user2,2000,0.25,2,5,0.5,2,3
d1,user1,1000,0.0,1,5,0.0,1,3
d2,user5,5000,1.0,5,5,1.0,2,2
d2,user4,4000,0.75,4,5,0.0,1,2
```

​	7.LAG(col,n,DEFAULT)/LEAD(col,n,DEFAULT)

​		统计窗口内往上/下第n行值。

​		col:列名

​		n:往上第n行(默认1)

​		DEFAULT:当往上第n行参数为NULL时取的值，若不指定为NULL

#### 四、hive SHELL

```
-i 从文件初始化HQL
-e 从命令行执行指定的HQL
-f 执行HQL脚本
-v 输出执行的HQL语句到控制台
-p connect to Hive Server on port number
-S 以不打印日志的形式执命名操作

例子：
hive -e "select * from user;"
hive -f hive.sql
```