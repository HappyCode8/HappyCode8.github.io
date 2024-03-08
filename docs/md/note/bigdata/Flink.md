# 基本概念

- DataStream
  
  Flink用类DataStream来表示程序中的流式数据。用户可以认为它们是含有重复数据的不可修改的集合(collection)，DataStream中元素的数量是无限的。对DataStream可以使用一些算子，例如KeyBy这样的算子，对它进行处理转换之后，它会转换成另外一种数据流，也称为keyedstream。那么基于keyedstream，我们进一步可以使用窗口算子，这主要是Flink程序设计中对数据流的一些处理方式。

- DataSet 
  
  Flink系统可对数据集进行转换（例如，过滤，映射，联接，分组)，数据集可从读取文件或从本地集合创建。结果通过接收器( Sink)返回，接收器可以将数据写入(分布式）文件或标准输出（例如命令行终端)

- Flink程序
  
  Flink程序由Source、Transformation和Sink三部分组成，其中Source主要负责数据的读取，支持HDFS、kafka和文本等;Transformation主要负责对数据的转换操作; Sink负责最终数据的输出，支持HDFS、kafka和文本输出等。在各部分之间流转的数据称为流( stream ) 。
  
  首先是用户提交Flink程序，这个Flink程序就会转换成逻辑数据流图。JobClient接收到逻辑数据流图之后，然后连同jar包以及一些依赖包就会提交给了JobManger，JobManger接收到逻辑数据流图之后会转成物理数据流图，这个物理数据流图是真实的可执行的，能够具体的将任务放置在TaskManager上，在TaskManager中会将它所拥有的资源划分成一个一个的TaskSlot。每个TaskSlot实际上就相当于是jvm，它的一个具体的线程。每个TaskSlot占用了TaskManager的一部分资源，这里的资源主要是以内存进行划分的，TaskSlot不对cpu的资源进行划分，因此没有对cpu的资源进行隔离。

- Operator
  
  在Flink中主要有三类Operator：
  
  - Source Operator:数据源操作，比如文件、socket、Kafka等。
  
  - Transformation Operator:数据转换操作，比如map，flatMap，reduce等算子。
  
  - Sink Operator:数据存储操作。比如数据存储到HDFS、Mysql、Kafka等等。

- 时间timeevent
  
  每个事件的时间可以分为三种:
  
  - event time，即事件发生时的时间
  
  - ingestion time，即事件到达流处理系统的时间
  
  - processing time，即事件被系统处理的时间

- 时间窗口timewindow
  
  可以根据窗口实现原理的不同分成三类:
  
  - 滚动窗口（Tumbling Window ) ，适合做Bl统计等（做每个时间段的聚合计算）假设要对传感器输出的数值求和。一分钟滚动窗口收集最近一分钟的数值，并在一分钟结束时输出总和
  
  - 滑动窗口( Sliding Window)，适用场景:对最近一个时间段内的统计（求某接口最近5min的失败率来决定是否要报警）。
  
  - 会话窗口( Session Window)，会话窗口由一系列事件组合一个指定时间长度的timeout间隙组成，类似于web应用的session，也就是一段时间没有接收到新数据就会生成新的窗口。特点:时间无对齐。

- 水位线watermark
  
  对于无穷数据集，我们缺乏一种有效的方式来判断数据完整性，就是有事件延迟了，对于延迟的元素，我们不可能无限期的等下去，必须要有一种机制来保证一个特定的时间后，必须触发Window进行计算。这个特别的机制,就是Watermark,Watermark可以理解成一个延迟触发机制，我们可以设置Watermark的延时时长t，每次系统会校验已经到达的数据中最大的maxEventTime，然后认定eventTime小于maxEventTime - t的所有数据都已经到达，如果有窗口的停止时间等于maxEventTime – t，那么这个窗口被触发执行。watermark 用来让程序自己平衡延迟和结果正确性。举例：
  
  > 有序流的watermark：16 15 14 13 12 11 (w10) 10 9 8 7 6 w(5) 5 4 3 2 1
  > 
  > 无序流的watermark：15 16 14 13 (w10) 12 8 11 9 10 6 w(5) 7 3 2 5 4 1
  
  设置的允许最大延迟到达时间t为2s，所以时间戳为7s的事件对应的Watermark 是 5s，时间戳为12s的事件的Watermark是10s，如果我们的窗口是1s~5s，窗口2是6s~10s，那么时间戳为7s的事件到达时的Matermarker.恰好触发窗口1，时间戳为 12s的事件到达时的Watermark恰好触发窗口2。
  
  Watermark就是触发前一窗口的“关窗时间”，一旦触发关门那么以当前时刻为准在窗口范围内的所有所有数据都会收入窗中。只要没有达到水位那么不管现实中的时间推进了多久都不会触发关窗。
  
  延迟事件是乱序事件的特例，和一般乱序事件不同的是它们的乱序程度超出了水位线( Watermark)的预计，导致窗口在它们到达之前已经关闭。
  
  延迟事件出现时窗口已经关闭并产出了计算结果，对于此种情况处理的方法有3种:
  
  - 重新激活已经关闭的窗口并重新计算以修正结果（Allowed Lateness）。
  - 将延迟事件收集起来另外处理（Side Output）。
  - 将延迟事件视为错误消息并丢弃。
  
  Flink默认的处理方式是第3种直接丢弃，其他两种方式分别使用Side Output和AllowedLateness。
  
  - Side Output机制可以将延迟事件单独放入一个数据流分支，这会作为Window计算结果的副产品，以便用户获取并对其进行特殊处理。
  
  - Allowed Lateness机制允许用户设置一个允许的最大延迟时长。Flink会在窗口关闭后一直保存窗口的状态直至超过允许延迟时长，这期间的延迟事件不会被丢弃，而是默认会触发窗口重新计算。因为保存窗口状态需要额外内存，并且如果窗口计算使用了ProcessWindowFunction APl还可能使得每个延迟事件触发一次窗口的全量计算，代价比较大，所以允许延迟时长不宜设得太长，延迟事件也不宜过多。

- 检查点Checkpoint
  
  - Flink中基于异步轻量级的分布式快照技术提供了Checkpoints容错机制，分布式快照可以将同一时间点Task/Operator的状态数据全局统一快照处理。Flink会在输入的数据集上间隔性地生成checkpoint barrier，通过棚栏( barrier)将间隔时间段内的数据划分到相应的checkpoint中。当应用出现异常时，Operator就能够从上一次快照中恢复所有算子之前的状态，从而保证数据的一致性。
    
    对于状态占用空间比较小的应用，快照产生过程非常轻量，高频率创建且对Flink任务性能影响相对较小。Checkpoint过程中状态数据一般被保存在一个可配置的环境中，通常是在JobManager节点或HDFS上。
  
  - 检查点常见配置参数
    
    - exactly-once和at-least-once语义选择
    
    - Checkpoint超时时间
    
    - 检查点之间最小时间间隔
    
    - 最大并行执行的检查点数量
    
    - 外部检查点

- 安全点savepoint
  
  Savepoints是检查点的一种特殊实现，底层其实是使用Checkpoints的机制。Savepoints是用户以手工命令的方式触发，并将结果持久化到指定的存储路径中，目的是帮助用户在升级和维护集群过程中保存系统中的状态数据，避免因为停机运维或者升级应用等正常终止应用的操作而导致系统无法恢复到原有的计算状态的情况，从而无法实现端到端的Exactly-Once语义保证。
  
  checkpoint的侧重点是“容错”，即Flink作业意外失败并重启之后，能够直接从早先打下的checkpoint恢复运行，且不影响作业逻辑的准确性。而savepoint的侧重点是“维护”，即Flink作业需要在人工干预下手动重启、升级、迁移或A/B测试时，先将状态整体写入可靠存储，维护完毕之后再从savepoint恢复现场。
  
  savepoint是“通过checkpoint机制”创建的，所以savepoint本质上是特殊的checkpoint。
  
  checkpoint面向Flink Runtime本身，由Flink的各个TaskManager定时触发快照并自动清理，一般不需要用户干预；savepoint面向用户，完全根据用户的需要触发与清理。

- 状态后端
  
  Task在本地内存中保存一份状态数据，但在分布式系统中，某个Task在任意时间点都可能发生故障，因此Task上的本地状态数据可以被认为是脆弱的。Flink定期将本地的状态数据持久化保存到一个存储空间上。用户可以选择以怎样的方式来保存这些状态数据，这种机制被称为状态后端（State Backend）。Flink提供了三种状态后端：内存、文件系统和RocksDB。
  
  内存肯定是读写性能最优的方式，单个节点的内存有限，因此这种状态后端会对状态数据的大小有限制。相比内存，本地磁盘的速度更慢，其所能承担的数据量更大，RocksDB 就是一种基于本地磁盘的状态后端。此外，Flink还允许将数据存储到分布式文件系统，如Hadoop的HDFS和AWS的S3上，分布式文件系统的数据存储能力非常大，足以应付海量数据的存储需求。我们将在后续文章中详细介绍三种状态后端的使用方法。

- 数据交换
  
  - 前向传播（Forward）：前一个算子子任务将数据直接传递给后一个算子子任务，数据不存在跨分区的交换，也避免了因数据交换产生的各类开销，图 3中Source和和flatMap之间就是这样的情形。
  
  - 全局广播（Broadcast）:将某份数据发送到所有分区上，这种策略涉及到了数据拷贝和网络通信，因此非常消耗资源。
  
  - 基于Key的数据重分布：数据以(Key, Value)形式存在，该策略将所有数据做一次重新分布，并保证相同Key的数据被发送到同一个分区上。图 3中keyBy算子将单词作为Key，把某个单词都发送到同一分区，以方便后续算子来统计这个单词出现的频次。
  
  - 随机策略（Random）：该策略将所有数据随机均匀地发送到多个分区上，以保证数据平均分配到不同分区上。该策略通常为了防止数据倾斜到某些分区，导致部分分区数据稀疏，部分分区数据拥堵，甚至超过该分区上算子的处理能力。

- 执行图
  
  - StreamGraph：是根据用户通过 DataStream API 编写的代码生成的最初的图，用来表示程序的拓扑结构。在这张图中，节点就是用户调用的算子，边表示数据流。
  
  - JobGraph：JobGraph是提交给 JobManager 的数据结构。StreamGraph经过优化后生成了 JobGraph，主要的优化为，将多个符合条件的节点链接在一起作为一个节点，这样可以减少数据交换所需要的序列化、反序列化以及传输消耗。这个链接的过程叫做算子链。
  
  - ExecutionGraph：JobManager 根据 JobGraph 生成ExecutionGraph。ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。
  
  - 物理执行图：JobManager 根据 ExecutionGraph 对作业进行调度后，在各个TaskManager 上部署任务形成的图，物理执行并不是一个具体的数据结构。

- TaskManager
  
  TaskManager是一个Java虚拟机进程，在TaskManager中可以并行运行多个Task。在程序执行之前，经过优化，部分Subtask被链接在一起，组成一个Task。每个Task是一个线程，需要TaskManager为其分配相应的资源，TaskManager使用任务槽位（Task Slot）给任务分配资源，简称槽位（Slot）。
  
  每个槽位的资源是整个TaskManager资源的子集，比如这里的TaskManager下有3个槽位，每个槽位占用TaskManager所管理的1/3的内存，在第一个槽位内运行的任务不会与在第二个槽位内运行的任务互相争抢内存资源。注意，在分配资源时，Flink并没有将CPU资源明确分配给各个槽位。
  
  Flink允许用户设置TaskManager中槽位的数目，这样用户就可以确定以怎样的粒度将任务做相互隔离。如果每个TaskManager只包含一个槽位，那么运行在该槽位内的任务将独享JVM。如果TaskManager包含多个槽位，那么多个槽位内的任务可以共享JVM资源，比如共享TCP连接、心跳信息、部分数据结构等。如无特殊需要，可以将槽位数目设置为TaskManager下可用的CPU核心数，那么平均下来，每个槽位都能获得至少一个CPU核心。
  
  一个TaskManager是一个进程，TaskManager可以管理一至多个Task，每个Task是一个线程，占用一个槽位。

# 原理



以一个wordcount程序举例[深入浅出：10行Flink WordCount程序背后的万字深度解析_windows环境 flink wordcount-CSDN博客](https://blog.csdn.net/zhousenshan/article/details/134362640)：

```scala
// Source
val consumer = new FlinkKafkaConsumer...
val stream = env.addSource(consumer)
// Transformation
val wordCount = stream
.flatMap(line=>line.split("\s"))
.map(word=>(word, 1))
.keyBy(0)
.timeWindow(Time.seconds(5))
.sum(1)
// Sink
val sink = new RedisSink...
wordCount.addSink(sink)
```

参数配置如下：

> 并行度：2

如果不合并source算子的话，会生成如下一些task

> 2个source
> 
> 2个flatmap、map（这两可以合到一起是因为没有跨分区的数据交换）
> 
> 2个keyby、timewindow、sum（这些可以合到一起是因为重分区完以后没有发生跨分区的数据交换）
> 
> 1个sink
> 
> 一共生成2+2+2+1=7个task，占用7个slot

合并算子的话，其实source跟flatmap、map也可以合到一起，所以合并以后一共有2+2+1=5个task，占用5个slot。开启槽位共享以后还是有5个task，Flink允许将独占一个槽位的任务与同一个作业中的其他任务共享槽位。于是可以将一个作业从开头到结尾的所有Subtask都放置在一个槽位中，这样5个task可以放在2个slot以后，{[1个source、(1个flatmap、map)]，[1个keyby、timewindow、sum]，1个sink}，{[1个source、(1个flatmap、map)]}，{}代表slot，[]代表task，()代表subtask

# 安装与启动

```bash
安装：brew install apache-flink
找到安装目录：brew info apache-flink
进入安装目录执行：  ./libexec/bin/start-cluster.sh
web页面(http://localhost:8081/)
例子：https://www.cnblogs.com/duniqb/p/14070809.html
```

无法关闭：https://blog.csdn.net/qq_37135484/article/details/102474087

使用docker安装

docker-compose.yml

```yml
version: "2.1"
services:
  jobmanager:
    image: flink
    expose:
      - "6123"
    ports:
      - "8081:8081"
    command: jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager

  taskmanager:
    image: flink
    expose:
      - "6121"
      - "6122"
    depends_on:
      - jobmanager
    command: taskmanager
    links:
      - "jobmanager:jobmanager"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
```

运行

```
docker-compose up -d
```

浏览器打开 http://127.0.0.1:8081 可以看到dashboard

# 实践

## DIM层的构建

创建配置表，当配置发生变化时改变此表，然后使用FlinkCDC监控变化，然后发送到kafka，通过HbaseAPI，将维度数据写入对应的HBase维表中

```sql
DROP TABLE IF EXISTS `tms_config_dim`;
CREATE TABLE `tms_config_dim` (
  `source_table` varchar(256) NOT NULL COMMENT '数据源表',
  `sink_table` varchar(256) DEFAULT NULL COMMENT '目标表',
  `sink_family` varchar(256) DEFAULT NULL COMMENT '目标表列族',
  `sink_columns` varchar(256) DEFAULT NULL COMMENT '目标表列',
  `sink_pk` varchar(256) DEFAULT NULL COMMENT '主键字段',
  PRIMARY KEY (`source_table`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='物流实时配置表';
```

- 安装与启动：
  
  ```bash
  安装：brew install apache-flink
  找到安装目录：brew info apache-flink
  进入安装目录执行：  ./libexec/bin/start-cluster.sh
  web页面(http://localhost:8081/)
  例子：https://www.cnblogs.com/duniqb/p/14070809.html
  ```
  
  无法关闭：https://blog.csdn.net/qq_37135484/article/details/102474087
  
  使用docker安装
  
  docker-compose.yml
  
  ```yml
  version: "2.1"
  services:
    jobmanager:
      image: flink
      expose:
        - "6123"
      ports:
        - "8081:8081"
      command: jobmanager
      environment:
        - JOB_MANAGER_RPC_ADDRESS=jobmanager
  
    taskmanager:
      image: flink
      expose:
        - "6121"
        - "6122"
      depends_on:
        - jobmanager
      command: taskmanager
      links:
        - "jobmanager:jobmanager"
      environment:
        - JOB_MANAGER_RPC_ADDRESS=jobmanager
  ```
  
  运行
  
  ```
  docker-compose up -d
  ```
  
  浏览器打开 http://127.0.0.1:8081 可以看到dashboard
  
  # 基于Flink的实时数仓的构建
  
  ## DIM层的构建
  
  创建配置表，当配置发生变化时改变此表，然后使用FlinkCDC监控变化，然后发送到kafka，通过HbaseAPI，将维度数据写入对应的HBase维表中
  
  ```sql
  DROP TABLE IF EXISTS `tms_config_dim`;
  CREATE TABLE `tms_config_dim` (
    `source_table` varchar(256) NOT NULL COMMENT '数据源表',
    `sink_table` varchar(256) DEFAULT NULL COMMENT '目标表',
    `sink_family` varchar(256) DEFAULT NULL COMMENT '目标表列族',
    `sink_columns` varchar(256) DEFAULT NULL COMMENT '目标表列',
    `sink_pk` varchar(256) DEFAULT NULL COMMENT '主键字段',
    PRIMARY KEY (`source_table`)
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='物流实时配置表';
  ```