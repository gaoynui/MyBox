Spark

------

#### 一、介绍

​	[官网](http://spark.apache.org/)

​	Spark是一个实现快速通用的**集群计算平台**。

​	它是由 加州大学伯克利分校AMP实验室 开发的通用内存并行计算框架，用来构建大型的、低延迟的数据分析应用程序。它扩展了广泛使用的MapReduce计算模型。高效的支撑更多计算模式，包括交互式查询和流处理。

​	Spark的一个主要特点是能够在内存中进行计算，及时依赖磁盘进行复杂的运算。

​	Spark是MapReduce的替代方案，而且兼容HDFS、Hive，可融入Hadoop的生态系统，以弥补MapReduce的不足。

#### 二、四大特性

1. 高效性

2. 易用性：支持Java、Python、Scala的API，Python和Scala的shell

3. 通用性

   <img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\spark\通用性.png" style="zoom:80%;" />

4. 兼容性

   Spark可以非常方便地与其他的开源产品进行融合。比如，Spark可以使用Hadoop的YARN和Apache Mesos作为它的资源管理和调度器，并且可以处理所有Hadoop支持的数据，包括HDFS、HBase和Cassandra等。

   

   ```
   yarn & mesos
   
   	Mesos于2007年诞生于UC Berkeley并在Twitter和Airbnb等公司的商用下不断被巩固，它的设计初衷是作为整个数据中心的一个可拓展的全局资源管理器。YARN出于管理Hadoop规模的需求。在YARN出现之前，资源管理（功能）集成在Hadoop MapReduce V1架构中，为了有助于MapReduce的扩展而将其移除（转移到YARN中实现）。
   
   	资源调度上的区别:Mesos让framework决定mesos提供的资源是否合适，从而接受或者拒绝这个资源，当framework长期拒绝资源，mesos将跳过该framework，将资源提供给其他framework(mesos本身并不知道各个应用程序资源需求)。对于yarn来说，决定权在于yarn本身:应用程序的App Mst会把各个任务的资源要求汇报给Yarn，Yarn根据需求为应用程序分配资源。
   
   	支持框架的区别:在Mesos中，各种计算框架是完全融入Mesos中的，也就是说，如果你想在Mesos中添加一个新的计算框架，首先需要在Mesos中部署一套该框架；而在YARN中，各种框架作为client端的library使用，仅仅是你编写的程序的一个库，不需要事先部署一套该框架。
   ```

#### 三、组成

**SparkCore**：将分布式数据抽象为弹性分布式数据集（RDD），实现了应用任务调度、RPC、序列化和压缩，并为运行在其上的上层组件提供API。

**SparkSQL**：Spark Sql 是Spark来操作结构化数据的程序包，可以让我使用SQL语句的方式来查询数据，Spark支持多种数据源，包含Hive表，parquest以及JSON等内容。

**SparkStreaming**： 是Spark提供的实时数据进行流式计算的组件。

**MLlib**：提供常用机器学习算法的实现库。

**GraphX**：提供一个分布式图计算框架，能高效进行图计算。

**BlinkDB**：用于在海量数据上进行交互式SQL的近似查询引擎。

**Tachyon**：以内存为中心高容错的的分布式文件系统。

#### 四、RDD

​	RDD（Resilient Distributed Dataset）叫做**弹性分布式数据集**，是Spark中最基本的数据抽象，它代表一个不可变、可分区、里面的元素可并行计算的集合。RDD具有数据流模型的特点：自动容错、位置感知性调度和可伸缩性。RDD允许用户在执行多个查询时显式地将工作集缓存在内存中，后续的查询能够重用工作集，这极大地提升了查询速度。

​	1.属性：

​	（1）一组分片(partition):数据集的基本组成单位。对于RDD来说，每个分片都会被一个计算任务处理，并决定并行计算的粒度。用户可以在创建RDD时指定RDD的分片个数，如果没有指定，那么就会采用默认值。默认值就是程序所分配到的CPU Core的数目。

​	(2)一个计算每个分区的函数。Spark中RDD的计算是以分片为单位的，每个RDD都会实现compute函数以达到这个目的。compute函数会对迭代器进行复合，不需要保存每次计算的结果。

​	(3)RDD之间的依赖关系。RDD的每次转换都会生成一个新的RDD，所以RDD之间就会形成类似于流水线一样的前后依赖关系。在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据，而不是对RDD的所有分区进行重新计算。

​	(4)一个Partitioner，即RDD的分片函数。当前Spark中实现了两种类型的分片函数，一个是基于哈希的HashPartitioner，另外一个是基于范围的RangePartitioner。只有对于key-value的RDD，才会有Partitioner，非key-value的RDD的Parititioner的值是None。Partitioner函数不但决定了RDD本身的分片数量，也决定了parent RDD Shuffle输出时的分片数量。

​	(5)一个列表，存储存取每个Partition的优先位置（preferred location）。对于一个HDFS文件来说，这个列表保存的就是每个Partition所在的块的位置。按照“移动数据不如移动计算”的理念，Spark在进行任务调度的时候，会尽可能地将计算任务分配到其所要处理数据块的存储位置。

![](C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\spark\RDD解析过程.png)

2.两种操作：Transformation & Action

​	(1)Transformation

​	file = sc.textFile(path)

| 接口      | 例子 | 解释 |
| :---- | ---- | :--- |
| map(func) | rdd = file.map(lambda w: w.split("-")) | 返回一个新的RDD，该RDD由每一个输入元素经过func函数转换后组成 |
| flatMap(func) | rdd = file.flatMap(lambda line: line.split("-")) | 类似于map，但是每一个输入元素可以被映射为0或多个输出元素（所以func应该返回一个序列，而不是单一元素） |
| filter() | rdd = file.filter(lambda w: w != "-") | 返回一个新的RDD，该RDD由经过func函数计算后返回值为true的输入元素组成 |
| distinct(n) | rdd = file.distinct(2) | 对数据进行去重，返回一个新的RDD，n参数用于设置任务并行数。 |
| sample(withReplacement,fraction,seed=None) | rdd1 = file.sample(False, 0.2, 2019) | 对数据进行采样，withReplacement参数表示是否放回抽样；fraction参数表示抽样比例；seed表示随机种子。 |
| union(otherRDD) | rdd1 = sc.parallelize(["union","other","rdd"]) rdd = file.union(rdd1) | 与另一个RDD数据集合并，返回一个新的RDD。 |
| intersection(otherRDD) | rdd1 = sc.parallelize([1,2,3,4,5]) rdd2 = sc.parallelize([3,4,5,6,7]) rdd3 = rdd1.intersection(rdd2) | 与另一个RDD数据集进行求交集计算，返回新的RDD。 |
| subtract() | rdd1 = sc.parallelize([1,2,3,4,5]) rdd2 = sc.parallelize([2,4])          rdd3 = rdd1.subtract(rdd2) | 对otherRDD进行减法操作，将原始RDD的元素减去新输入RDD的元素，将差值返回新RDD。 |
| cartesian() | rdd1 = sc.parallelize([1,2])          rdd2 = sc.parallelize([3,4])         rdd3 = rdd1.cartesian(rdd2) | 对两个RDD数据集U，V求笛卡尔积，返回一个新的RDD数据集，其中每个元素为(u,v)。 |
| groupByKey(n) | rdd = file.groupByKey() | 对具有相同键的值进行分组，返回一个元素为(K,[Iterable])的键值对RDD，n用于指定任务并行数，默认为8。 |
| reduceByKey(func,n) | rdd = rdd.reduceByKey(lambda w1,w2: w1+w2) | 对具有相同键的值进行合并，返回一个新的键值对RDD，n用于设置任务并行数。 |
| sortByKey(ascending, n) | 对词频统计结果按降序排序（先把K-V值互换）                                rdd = rdd.map(lambda v: (v[1],v[0])) rdd = rdd.sortByKey(False)          rdd = rdd.map(lambda v: (v[1],v[0])) | 可以对键值对RDD按照键进行排序操作，其中K需要实现Ordered方法。ascending 决定RDD中的元素按升序还是降序排序，默认是True升序,n用于设置任务并行数。 |
| sortBy(func,[ascending], n) | x = sc.parallelize(['wills', 'kris', 'april', 'chang'])                                                        def sortByFirstLetter(s): return s[0]             def sortBySecondLetter(s): return s[1]                y = x.sortBy(sortByFirstLetter).collect()        yy = x.sortBy(sortBySecondLetter).collect()  print '按第一个字母排序结果： {}'.format(y) print '按第二个字母排序结果：{}'.format(yy) | 与sortByKey类似，但是更灵活 第一个参数是根据什么排序 第二个是怎么排序 false倒序  第三个排序后分区数 默认与原RDD一样 |
| aggregateByKey(zeroValue,seqFunc,combFunc,n) | 对新闻出现过的单词进行词频统计                                                     rdd = rdd.aggregateByKey(0,lambda v1,v2: v1+v2, lambda a1,a2: a1+a2) | 可以对具有相同键的值进行聚合，把(K,V)键值对RDD转换为新的(K,U)键值对RDD，其中U由给定的combFunc和中立零值zeroValue聚合而成，U可以有与V不一致的形式；zeroValue 可以是0如果聚合的目的是求和，可以是List如果目的是对值进行统合，可以是Set如果目的是聚合唯一值；seqFunc: (U,V) => U 对分区内的元素进行聚合（操作发生在每个分区内部）；combFunc: (U,U) => U 对不同分区的聚合结果做进一步的聚合（操作发生在全部分区的聚合结果间）；n用于设置任务并行数 |

​	(2)Action

| 接口                                             | 例子                                                         | 解释                                                         |
| ------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| reduce(func)                                     | sum = file.reduce(lambda v1,v2: v1 + v2)                     | 通过指定函数（如求和、统计）对RDD数据集元素进行聚合。        |
| collect()                                        |                                                              | 以数组的形式返回RDD数据集的所有元素。                        |
| count()                                          |                                                              | 返回RDD数据集中元素的个数。                                  |
| first()                                          |                                                              | 返回RDD数据集的第一个元素。                                  |
| take(n)                                          |                                                              | 以数组的形式返回RDD数据集的前num个元素。                     |
| top(num,key=None)                                | rdd = sc.parallelize([5,24,3,12,46])                                                       print(rdd.top(3))   # 输出 [46,24,12]                                                                   rdd = sc.parallelize([5,24,3,12,46])                                                        print(rdd.top(3, key=str))    # 输出 [5,46,3] | 以数组的形式返回RDD数据集的前num个元素，默认按**降序**，或者通过key函数指定。 |
| takeOrdered(num,key=None)                        | rdd = sc.parallelize([5,2,3,1,4])                    print(rdd.takeOrdered(3)) # 输出 [1,2,3]                                      rdd = sc.parallelize([5,2,3,1,4])                             print(rdd.takeOrdered(3, key=lambda x: -x)) # 输出 [5,4,3] | 以数组的形式返回RDD数据集的前num个元素，默认按**升序**排序，或者通过key函数指定。 |
| takeSample(withReplacement,num,seed=n)           | rdd = sc.parallelize([5,2,3,1,4]) print(rdd.takeSample(False,3,seed=2019)) | 对RDD数据集进行采样，并以数组的形式返回，withReplacement参数表示是否放回抽样；num参数表示抽样个数；seed表示随机种子。 |
| lookup()                                         |                                                              | 以数组的形式返回键值对RDD中键为key的所有值，如果RDD数据集经过特定转换操作按照key进行了分区，那么此行动操作效率会很高。 |
| foreach()                                        | file.foreach(print)                                          | 将RDD数据集中的每个元素加载到指定函数进行操作，无返回值。    |
| aggregate(zeroValue,seqOp,combOp)                | #求一个数组元素的均值                         rdd = sc.parallelize([5,2,3,1,4])               res = rdd.aggregate((0,0), lambda u,v: (u[0]+v,u[1]+1), lambda u1,u2: (u1[0]+u2[0],u1[1]+u2[1])) print(res[0]/res[1]) | 对RDD数据集的元素进行聚合，不要求返回值类型与RDD类型一致;zeroValue: U 给定初始值，形式与最终返回值U一致；seqOp: (U,V) => U 对分区内的元素进行聚合（操作发生在每个分区内部）；combOp: (U,U) => U 对不同分区的聚合结果做进一步的聚合（操作发生在全部分区的聚合结果间) |
| countByKey()                                     | rdd = sc.parallelize([("rdd",1),("rdd",2),("spark",2)])                                            res = rdd.countByKey()                             #输出结果 {'rdd': 2, 'spark': 1} | 以字典的形式返回键值对RDD数据集中每个键的元素的统计数，即(K,count) |
| countByValue()                                   | rdd = sc.parallelize([2,2,3,1,1])                    res = rdd.countByValue()                               #输出结果 {2: 2, 3: 1, 1: 2} | 以字典的形式返回RDD数据集中每个元素的统计数，即(V,count)     |
| saveAsTextFile(path, compressionCodecClass=None) | f = NamedTemporaryFile(delete=True) f.close()                                                         codec = "org.apache.hadoop.io.compress.GzipCodec" sc.parallelize(['spark', 'rdd']).saveAsTextFile(f.name, codec) | 把RDD数据集保存为文本文件，并可以指定是否压缩。              |

3.宽依赖和窄依赖

​	![](C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\spark\宽窄依赖.png)

#### 五、广播和累加器

​	1.概述

​		在spark程序中，当一个传递给Spark操作(例如map和reduce)的函数在远程节点上面运行时，Spark操作实际上操作的是这个函数所用变量的一个独立副本。这些变量会被复制到每台机器上，并且这些变量在远程机器上的所有更新都不会传递回驱动程序。通常跨任务的读写变量是低效的，但是，Spark还是为两种常见的使用模式提供了两种有限的共享变量：广播变量（broadcast variable）和累加器（accumulator）

​	2.广播变量

​		<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\spark\广播变量.png" style="zoom:67%;" />

```scala
// 定义广播变量,变量一旦被定义为一个广播变量，那么这个变量只能读，不能修改
val a = 3
val broadcast = sc.broadcast(a)
// 还原广播变量
val c = broadcast.value
```

​		注意事项：

​			1、能不能将一个RDD使用广播变量广播出去？不能，因为RDD是不存储数据的。**可以将RDD的结果广播出去。**

​			2、 广播变量只能在Driver端定义，**不能在Executor端定义。**

​			3、 在Driver端可以修改广播变量的值，**在Executor端无法修改广播变量的值。**

​			4、如果Executor端用到了Driver的变量，如果**不使用广播变量在Executor有多少task就有多少Driver端的变量副本。**

​			5、如果Executor端用到了Driver的变量，如果**使用广播变量在每个Executor中只有一份Driver端的变量副本。**

​	3.累加器

​		在spark应用程序中，我们经常会有这样的需求，如异常监控，调试，记录符合某特性的数据的数目，这种需求都需要用到计数器，如果一个变量不被声明为一个累加器，那么它将在被改变时不会再driver端进行全局汇总，即在分布式运行时每个task运行的只是原始变量的一个副本，并不能改变原始变量的值，但是当这个变量被声明为累加器后，该变量就会有分布式计数的功能。

​		<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\spark\累加器变量.png" style="zoom:67%;" />

```scala
// 定义一个累加器
val a = sc.accumulator(0)
// 还原一个累加器
val b = a.value
```

​		注意事项：

​			1、 **累加器在Driver端定义赋初始值，累加器只能在Driver端读取最后的值，在Excutor端更新。**

​			2、累加器不是一个调优的操作，因为如果不这样做，结果是错的

#### 六、Spark运行流程（job = n * stage）(stage = m * task)

​	1.基本概念

​		（1）Application：表示你的应用程序

​		（2）Driver：表示main()函数，创建SparkContext。由SparkContext负责与ClusterManager通信，进行资源的申请，任务的分配和监控等。程序执行完毕后关闭SparkContext

​		（3）Executor：某个Application运行在Worker节点上的一个进程，该进程负责运行某些task，并且负责将数据存在内存或者磁盘上。在Spark on Yarn模式下，其进程名称为 CoarseGrainedExecutor Backend，一个CoarseGrainedExecutor Backend进程有且仅有一个executor对象，它负责将Task包装成taskRunner，并从线程池中抽取出一个空闲线程运行Task，这样，每个CoarseGrainedExecutorBackend能并行运行Task的数据就取决于分配给它的CPU的个数。

​		（4）Worker：集群中可以运行Application代码的节点。在Standalone模式中指的是通过slave文件配置的worker节点，在Spark on Yarn模式中指的就是NodeManager节点。

​		（5）Task：在Executor进程中执行任务的**工作单元**，多个Task组成一个Stage

​		（6）Job：包含多个Task组成的并行计算，是由Action行为触发的

​		（7）Stage：每个Job会被拆分很多组Task，作为一个TaskSet，其名称为Stage

​		（8）DAGScheduler：根据Job构建基于Stage的DAG(有向无环图)，并**提交Stage给TaskScheduler**，其划分Stage的依据是RDD之间的依赖关系

​		（9）TaskScheduler：将TaskSet提交给Worker（集群）运行，每个Executor运行什么Task就是在此处分配的。

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\spark\job允许流程.png" style="zoom:67%;" />

​	2.两种运行模式

​		[Spark On Yarn的两种模式yarn-cluster和yarn-client深度剖析](https://www.cnblogs.com/ittangtang/p/7967386.html)

```
1.YarnCluster的Driver是在集群的某一台NM上，但是Yarn-Client就是在RM的机器上； 
2.而Driver会和Executors进行通信，所以Yarn_cluster在提交App之后可以关闭Client，而Yarn-Client不可以； 
```



#### 七、SparkCore调优

​	1.开发调优

​		(1)原则一:尽可能复用同一个RDD

​		(2)原则二:对多次使用的RDD持久化

​			cache()方法:使用非序列化的方式将RDD中的数据全部尝试持久化到内存中。

​				val rdd1 = sc.textFile("hdfs://192.168.0.1:9000/hello.txt").cache()

​			persist()方法:手动选择持久化级别，并使用指定的方式进行持久化。

​				val rdd1 = sc.textFile("hdfs://192.168.0.1:9000/hello.txt").persist(StorageLevel.MEMORY_AND_DISK_SER)

​		(3)原则三:尽量避免使用shuffle类算子，因为Spark作业运行过程中，最消耗性能的地方就是shuffle过程。shuffle过程，简单来说，就是将分布在集群中多个节点上的同一个key，拉取到同一个节点上，进行聚合或join等操作

​		(4)原则四:使用map-side预聚合的shuffle操作，map-side预聚合，说的是在每个节点本地对相同的key进行一次聚合操作

​		(5)原则五:使用高性能的算子

​			a.reduceByKey()/aggregateByKey()替代groupByKey()

​			b.mapPartitions类的算子，一次函数调用会处理一个partition所有的数据，而不是一次函数调用处理一条，性能相对来说会高一些。但是有的时候，使用mapPartitions会出现OOM（内存溢出）的问题。因为单次函数调用就要处理掉一个partition所有的数据，如果内存不够，垃圾回收时是无法回收掉太多对象的，很可能出现OOM异常。

​			c.foreachPartitions()替代foreach()

​			d.通常对一个RDD执行filter算子过滤掉RDD中较多数据后（比如30%以上的数据），建议使用coalesce算子，手动减少RDD的partition数量，将RDD中的数据压缩到更少的partition中去。因为filter之后，RDD的每个partition中都会有很多数据被过滤掉，此时如果照常进行后续的计算，其实每个task处理的partition中的数据量并不是很多，有一点资源浪费，而且此时处理的task越多，可能速度反而越慢。因此用coalesce减少partition数量，将RDD中的数据压缩到更少的partition之后，只要使用更少的task即可处理完所有的partition。

​		(6)原则六:广播大变量

​		*(7)原则七:使用Kryo优化序列化性能

​		*(8)原则八:Data Locality本地化级别

​			**PROCESS_LOCAL**：进程本地化，代码和数据在同一个进程中，也就是在同一个executor中；计算数据的task由executor执行，数据在executor的BlockManager中；性能最好

​			**NODE_LOCAL**：节点本地化，代码和数据在同一个节点中；比如说，数据作为一个HDFS block块，就在节点上，而task在节点上某个executor中运行；或者是，数据和task在一个节点上的不同executor中；数据需要在进程间进行传输
​			**NO_PREF**：对于task来说，数据从哪里获取都一样，没有好坏之分
​			**RACK_LOCAL**：机架本地化，数据和task在一个机架的两个节点上；数据需要通过网络在节点之间进行传输
​			**ANY**：数据和task可能在集群中的任何地方，而且不在一个机架中，性能最差

​	2.倾斜调优

​		数据倾斜现象：

​			绝大多数task执行得都非常快，但个别task执行极慢。比如，总共有1000个task，997个task都在1分钟之内执行完了，但是剩余两三个task却要一两个小时。这种情况很常见。

​			原本能够正常执行的Spark作业，某天突然报出OOM（内存溢出）异常，观察异常栈，是我们写的业务代码造成的。这种情况比较少见。

​		<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\spark\数据倾斜.png" style="zoom: 50%;" />

​	3.shuffle调优

​		shuffle定义：下一个stage向上一个stage要数据

​	4.JVM的GC(garbage collector)机制

​		（1）概述：jvm 中，程序计数器、虚拟机栈、本地方法栈都是随线程而生随线程而灭，栈帧随着方法的进入和退出做入栈和出栈操作，实现了自动的内存清理，因此，我们的**内存垃圾回收主要集中于 java 堆和方法区中**，在程序运行期间，这部分内存的分配和使用都是动态的。

​				GC其实是一种自动的内存管理工具，其行为主要包括2步:在Java堆中，为新创建的对象分配空间;在Java堆中，回收没用的对象占用的空间。

​				判断对象是否存活一般有两种方式：

​				**引用计数**：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，缺点是无法解决对象相互循环引用的问题。

​				**可达性分析（Reachability Analysis）**：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。不可达对象。

```
在Java语言中，GC Roots包括：

  虚拟机栈中引用的对象。

  方法区中类静态属性实体引用的对象。

  方法区中常量引用的对象。

  本地方法栈中JNI引用的对象。
```

#### 八、分区

​	概念：分区是RDD内部并行计算的一个计算单元，RDD的数据集在逻辑上被划分为多个分片，每一个分片称为分区，分区的格式决定了并行计算的粒度，而每个分区的数值计算都是在一个任务中进行的，因此任务的个数，也是由RDD(准确来说是作业最后一个RDD)的分区数决定。

​	原则：尽可能是得分区的个数等于集群核心数目

​	分区设置：

```
# 生成RDD时指定n个分区
sc.parallelize(array, n)
# local[n],分区个数为cpu核数
conf = new SparkConf().setMater("local[*]")
# 配置参数
con = new SparkConf().set("spark.defalut.parallelism", "5")
# RDD重分区
sqcTemp = spark.createDataFrame(List)
sqc = sqcTemp.repartition(n)
# config配置
.config("spark.sql.files.openCostInBytes", "4194304") # 4MB
.config("spark.sql.shuffle.partitions", "10") # 分片下小文件数据量
# coalesce
spark.sql("select * from sour_table").coalesce(10).write.format("hive").mode("Append").partitionBy("imp_date").saveAsTable("des_table")
```

