# Spark优化思路

谈到优化就一定要了解一个job的执行过程，先聊下spark的job执行吧

​		spark中的计算全部基于RDD的概念 。

​		 job 提交后由Driver依据RDD的依赖关系，按照宽依赖的数量将job切分为Stage 。每个stage中task的数量由最后一个RDD的分区数决定，最后将task分配给Executor执行，最终输出结果。

​		在这个过程中进行优化的几个点

#### 基本调节

1.Executor负责执行计算

  	增加Executor的个数，每个executor cpu核数、以及内存，给予其更强的能力，其计算执行的更快

2.宽依赖

​	每次分段都伴随着shuffle的过程存在IO严重影响速度，那我们如何避免shuffle的出现，或者想办法减少shuffle时的IO呢

- ​	对算子进行优化，尽量避免使用会产生shuffle的算子，或者使用性能更佳的算子（reduceByKey代替groupByKey）
- shuffle过程先写入内存，提升内存可以减少IO的次数，甚至IO都不会发生 shuffle及已经完成（128m数据分配1G内存）

​	IO不仅发生在shuffle 中还有可以出现在 Driver与Executor之间，当运算需要由Driver给Executor发送数据时要怎么优化呢

- ​		默认情况加Driver会给每个Task发送一份数据，无疑其中的IO时巨大的，spark为我们提供的广播	变量的方式，为每个Executor发送一份数据。减少了 部份的IO，但是往往Executor的数量也很多效	率提升很有限。

#### RDD优化

1. 优化算子间的计算逻辑，避免相同的算子和计算逻辑对RDD进行重复计算
2. RDD持久化：当多次计算都依赖于同一个RDD时，必须对多次使用的RDD进行持久化，通过持久化将公共RDD的数据缓存到内存/磁盘中，之后对于公共RDD的计算都会从内存/磁盘中直接获取RDD数据。（持久化是要考虑序列化的方式）

#### 并行度调节

spark的并行度指task的数量。

​	并行度低，会导致资源浪费。例如，20个Executor，每个Executor分配3个CPU core，而Spark作业有40个task，这样每个Executor分配到的task个数是2个，这就使得每个Executor有一个CPU core空闲，导致资源的浪费。

​	理想的并行度，应该让并行度与资源向匹配，就是在资源允许的前提下，设置尽可能大的并行度。