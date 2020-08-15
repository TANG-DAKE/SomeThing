## Spark为什么比Hadoop快

##### 1.前言

Spark是基于内存的计算，而Hadoop是基于磁盘的计算

Spark和Hadoop的根本差异是多个任务之间的数据通信问题：Spark多个任务之间数据通信（NettyRPC）是基于内存，而Hadoop是基于磁盘。

##### 2.Spark比Hadoop快的主要原因有：

​	1.消除了冗余的HDFS读写
​		Hadoop每次shuffle操作后，必须写到磁盘，而Spark在shuffle后不一定落盘，可以cache到内存中，以便迭代时使用。如果操作复杂，很多的shufle操作，那么Hadoop的读写IO时间会大大增加。

​	2.消除了冗余的MapReduce阶段
​		Hadoop的shuffle操作一定连着完整的MapReduce操作，冗余繁琐。而Spark支持DAG，基于RDD提供了丰富的算子操作，且reduce操作产生shuffle数据，可以缓存在内存中。

​	3.JVM的优化
​		Spark Task的启动时间快。Spark采用fork线程的方式，Spark每次MapReduce操作是基于线程的，只在启动。而Hadoop采用创建新的进程的方式，启动一个Task便会启动一次JVM。

​		Spark的Executor是启动一次JVM，内存的Task操作是在线程池内线程复用的。

​		每次启动JVM的时间可能就需要几秒甚至十几秒，那么当Task多了，这个时间Hadoop不知道比Spark慢了多少。