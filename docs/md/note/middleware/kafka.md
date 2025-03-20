# Kafka常见面试题

## 为什么要用消息队列

> 解耦、异步、削峰
> 
> 削峰：高低峰日志收集
> 
> 解耦：通话数据对接多个业务方
> 
> 异步：复杂操作打点，核心逻辑赶紧返回

## 消息队列有什么缺点

> - 系统可用性降低
>   
>   系统引入的外部依赖越多，越容易挂掉。本来你就是 A 系统调用 BCD 三个系统的接口就好了，
>   
>   ABCD 四个系统还好好的，没啥问题，你偏加个 MQ 进来，万一 MQ 挂了咋整
> 
> - 系统复杂度提高
>   
>   硬生生加个 MQ 进来，你怎么保证消息没有重复消费？怎么处理消息丢失的情况？怎么保证
>   
>   消息传递的顺序性等等问题
> 
> - 一致性问题
>   
>   A 系统处理完了直接返回成功了，人都以为你这个请求就成功了；但是问题是，要是 BCD 三个
>   
>   系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，数据就不一致了。

## kafka的基本概念

> 生产者（producer）
> 
> 消费者（consumer）
> 
> 消费者组（consumer group）：对于同一个topic，会广播给不同的group,一个group中，只有一个consumer可以消费该信息。
> 
> broker：kafak集群中每个kafka的节点
> 
> broker组：按照broker进行分组，同一个partition不会被多个broker同时保存。当一个partiiton非常大的时候，可以通过多个broker同时保存，但不会被保存多份。
> 
> 主题（topic）：一类消息
> 
> 分区（partion）：数据存储的基本单元。一个topic数据会被分散到多个partition,每一个partition都是有序的。消费者数目<= partition的数目
> 
> 集群（cluster）：broker的组合
> 
> 副本（repliaction）：基本单位是partition，所有读和写都从leader进，follower只做备份，且follower必须能够及时复制keader的数据；
> 
> 可用副本（AR）：分区中所有副本,ISR+OR
> 
> 同步中的副本（ISR，In-Sync Replicas）：所有与leader副本保持一定程度同步的副本，超过一段时间未复制数据就会从ISR中移除
> 
> 同步外的副本（OSR，Out-Sync Replicas）：与leader副本滞后过多的副本
> 
> 高水位（HW）:消费者只能拉取这个offset之前的消息
> 
> 日志尾端：（LEO，log end offset）：下一条待写入消息的offset，ISR中每一个都有自己的LEO，最小的LEO为分区的HW

## 如何保证kafka高可用

> 由多个 broker 组成，每个 broker 是一个节点；一个topic可以划分为多个 partition，每个 partition 可以存在于不同的 broker 上，每个partition 就放一部分数据，每个partion可以有多个副本

## 如何保证消息不被重复消费

> 由业务来保证
> 
> - 写库，你先根据主键查一下，如果这数据都有了，你就别插入了，update 一下。
> - 写 Redis，那没问题了，反正每次都是 set，天然幂等性。
> - 不是上面两个场景，那做的稍微复杂一点，需要让生产者发送每条数据的时候，里面加一个全局唯一的 id，类似订单 id 之类的东西，然后你这里消费到了之后，先根据这个 id 去比如 Redis 里查一下，之前消费过吗？如果没有消费过，你就处理，然后这个 id 写 

## 如何保证消费不丢

> - 保证生产者不丢
>   
>   通过合理设置一定不会丢，要求是，你的 leader 接收到消息，所有的 follower 都同步到了消息之后，才认为本次写成功了。如果没满足这个条件，生产者会自动不断的重试，重试无限次。
> 
> - 保证kafka、不丢
>   
>   1. 给 topic 设置 replication.factor 参数：这个值必须大于 1，要求每个 partition 必须有至少 2 个副本。
>   
>   2. 在 Kafka 服务端设置 min.insync.replicas 参数：这个值必须大于 1，这个是要求一个 leader 至少感知到有至少一个 follower 还跟自己保持联系，没掉队，这样才能确保 leader挂了还有一个 follower 。
>   
>   3. 在 producer 端设置 acks=all ：这个是要求每条数据，必须是写入所有 replica 之后，才 能认为是写成功了。 
>   
>   4. 在 producer 端设置 retries=MAX （很大很大很大的一个值，无限次重试的意思）：这个是要求一旦写入失败，就无限重试，卡在这里了。
> 
> - 保证消费者不丢
>   
>   关闭自动提交 offset，在处理完之后自己手动提交 offet，就可以保证数据不会丢。

## 如何保证消息顺序性

> - 只设置一个partion（不推荐）
> - 多生产者生产
> 
> 指定某个 id 作为 key，那么这个订单相关的数据，一定会被分发到同一个 partition 中去，而且这个 partition 中的数据一定是有顺序的。可能的问题是ab交替发其中一个失败后重试成功但是落后另一个，可以设置broker在响应请求之前client不能再向同一个broker发送请求
> 
> - 多线程消费
> 
> 写 N 个内存 queue，具有相同 key 的数据都到同一个内存 queue；然后对于 N 个线程，每个线程分别消费一个内存 queue 即可，这样就能保证顺序性。

## 线上出问题导致消息大量积压

> - 先修复 consumer 的问题，确保其恢复消费速度，然后将现有 consumer 都停掉。
> - 新建一个 topic，partition 是原来的 10 倍，临时建立好原先 10 倍的 queue 数量。
> - 然后写一个临时的分发数据的 consumer 程序，这个程序部署上去消费积压的数据，消费之 后不做耗时的处理，直接均匀轮询写入临时建立好的 10 倍数量的 queue。
> - 接着临时征用 10 倍的机器来部署 consumer，每一批 consumer 消费一个临时 queue 的数据。这种做法相当于是临时将 queue 资源和 consumer 资源扩大 10 倍，以正常的 10 倍速度来消费数据。
> - 等快速消费完积压数据之后，得恢复原先部署的架构，重新用原先的 consumer 机器来消费消息。

## 如何设计一个消息队列

> 可以从以下几方面思考：
> 
> - 支持可伸缩性
> - 怎么落磁盘
> - 支持可用性
> - 支持数据零丢失
> - 怎么尽可能快
> - 连接，客户端等

## kafka的消费语义

> 至少一次(at-least-once): 关闭offset自动提交，消费完了但是没发确认，开机重消费，至少不会丢数据，客户端可以自己去重。
> 
> 至多一次(at-most-once): 开启offset自动提交，下一次再消费的时候就会认为offset已经成功了，直接丢弃消息。就会造成丢数据。
> 
> 仅一次(exactly-once): 开启offset自动提交，可以开启consumer.seek()方法，相当于自己处理分区和offset，可以在此基础上开启事务，保持原子性，只有数据库保存成功再提交offset，保证两者同时成功。

## offset的自动提交与手动提交

> 默认自动提交，可以通过enable.auto.commit配置，auto.commit.interval.ms与其配合使用，指定提交偏移量的频率，默认5秒。值太小增加网络流量，太大故障时会收到大量重复数据。
> 
> 有同步、异步之分，无参的方法适用于所有订阅的主题和分区，带参的只会提交Map中指定的偏移量、分区和主题。使用手工提交的好处是可以直接控制记录何时被视为已处理，防止拉下来但是没实际处理完的情况。

## 如何优化Kafka的写入速度

> 1. 增加线程
> 2. 提高 batch.size
> 3. 增加更多 producer 实例
> 4. 增加 partition 数
> 5. 设置 acks=-1 时，如果延迟增大：可以增大 num.replica.fetchers（follower 同步数据的线程数）来调解
> 6. 跨数据中心的传输：增加 socket 缓冲区设置以及 OS tcp 缓冲区设置。

## ack 为 0， 1， -1 各自的含义？

> 1. 1（默认） 数据发送到Kafka后，经过leader成功接收消息的的确认，就算是发送成功了。在这种情况下，如果leader宕机了，则会丢失数据。
> 2. 0 生产者将数据发送出去就不管了，不去等待任何返回。这种情况下数据传输效率最高，但是数据可靠性确是最低的。
> 3. -1 producer需要等待ISR中的所有follower都确认接收到数据后才算一次发送完成，可靠性最高。当ISR中所有Replica都向Leader发送ACK时，leader才commit，这时候producer才能认为一个请求中的消息都commit了。

## kafka unclean 配置

> unclean.leader.election.enable 为true的话，意味着非ISR集合的broker 也可以参与选举，这样有可能就会丢数据，spark streaming在消费过程中拿到的 end offset 会突然变小，导致 spark streaming job挂掉。如果unclean.leader.election.enable参数设置为true，就有可能发生数据丢失和数据不一致的情况，Kafka的可靠性就会降低；而如果unclean.leader.election.enable参数设置为false，Kafka的可用性就会降低。

## leader crash时，ISR为空怎么办

> kafka在Broker端提供了一个配置参数：unclean.leader.election,这个参数有两个值：
> 
> true（默认）：允许不同步副本成为leader，由于不同步副本的消息较为滞后，此时成为leader，可能会出现消息不一致的情况。
> 
> false：不允许不同步副本成为leader，此时如果发生ISR列表为空，会一直等待旧leader恢复，降低了可用性。

## kafka的message格式是什么样的

> Message=固定长度的header+变长的消息体body
> 
> header=一个字节的magic(文件格式)+四个字节的CRC32(用于判断body消息体是否正常)
> 
> 当magic=1时，会在magic和crc32之间多一个字节的数据：attributes(保存一些相关属性，比如是否压缩、压缩格式等等);如果magic的值为0，那么不存在attributes属性
> 
> body是由N个字节构成的一个消息体，包含了具体的key/value消息

## Kafka为什么这么快

> - 并行处理
>   
>   一方面，由于不同 Partition 可位于不同机器，因此可以充分利用集群优势，实现机器间的并行处理。另一方面，由于 Partition 在物理上对应一个文件夹，即使多个 Partition 位于同一个节点，也可通过配置让同一节点上的不同 Partition 置于不同的磁盘上，从而实现磁盘间的并行处理，充分发挥多磁盘的优势。
> 
> - 顺序写
>   
>   **Kafka 中每个分区是一个有序的，不可变的消息序列**，新的消息不断追加到 partition 的末尾，这个就是顺序写。
> 
> - 利用了现代操作系统分页存储 Page Cache 来利用内存提高 I/O 效率
>   
>   Broker 收到数据后，写磁盘时只是将数据写入 Page Cache，并不保证数据一定完全写入磁盘。从这一点看，可能会造成机器宕机时，Page Cache 内的数据未写入磁盘从而造成数据丢失。但是这种丢失只发生在机器断电等造成操作系统不工作的场景，而这种场景完全可以由 Kafka 层面的 Replication 机制去解决。如果为了保证这种情况下数据不丢失而强制将 Page Cache 中的数据 Flush 到磁盘，反而会降低性能。也正因如此，Kafka 虽然提供了 `flush.messages` 和 `flush.ms` 两个参数将 Page Cache 中的数据强制 Flush 到磁盘，但是 Kafka 并不建议使用。
> 
> - 零拷贝
>   
>   1. 网络数据持久化到磁盘 (Producer 到 Broker)
>   2. 磁盘文件通过网络发送（Broker 到 Consumer)
> 
> - 批处理
>   
>   Kafka 的客户端和 broker 还会在通过网络发送数据之前，在一个批处理中累积多条记录 (包括读和写)。记录的批处理分摊了网络往返的开销，使用了更大的数据包从而提高了带宽利用率。
> 
> - 数据压缩
>   
>   Producer 可将数据压缩后发送给 broker，从而减少网络传输代价，目前支持的压缩算法有：Snappy、Gzip、LZ4。数据压缩一般都是和批处理配套使用来作为优化手段的
> 
> - 文件分段

## 消费者提交消费位移时提交的是当前消费到的最新消息的offset还是offset+1?

> offset+1

## kafka是pull还是push？

> Pull
> 
> push:像Scripe和apache flume是使用push 模式。优点：broker能以最大速率发送消息。缺点：当broker推送的消息远大于消费者的速率时，消费者就会崩溃，当broker想避免消费者崩溃，采取远小于消费者速率推送消息，导致一次推送较小的消息造成浪费。
> 
> pull:消费者主动向broker拉取消息，适合不同消费速率的消费者。pull缺点：当broker没有消息推送时，导致消费者不断等待轮询。为了避免kafka有个参数，可以以让消费者阻塞。

## ZK在kafka中的作用

> ZooKeeper 管理者所有的 Topic 和 Partition。
> 
> Topic 和 Partition 存储在 Node 物理节点中，ZooKeeper负责维护这些 Node。Topic A 的 Partition #1 有3份，分布在各个 Node 上。这样可以增加 Kafka 的可靠性和系统弹性。3个 Partition #1 中，ZooKeeper 会指定一个 Leader，负责接收生产者发来的消息。其他2个 Partition #1 会作为 Follower，Leader 接收到的消息会复制给 Follower。这样，每个 Partition 都含有了全量消息数据。即使某个 Node 节点出现了故障，也不用担心消息的损坏。

## kafka的rebalance

> 触发时机：
> 
> 1. 消费者数量变化
> 
> 2. 分区数量变化（只允许增大）
> 
> 3. 主题创建
>    
>    当消费者订阅主题时使用的是正则表达式，例如 “test.*”，表示订阅所有以 test 开头的主题，当有新的以 test 开头的主题被创建时，则需要通过再均衡将该主题的分区分配给消费者。
> 
> 平衡策略：
> 
> 1. Round Robin：会采用轮询的方式将当前所有的分区依次分配给所有的consumer；
> 2. Range：首先会计算每个consumer可以消费的分区个数，然后按照顺序将指定个数范围的分区分配给各个consumer；
> 3. Sticky：这种分区策略是最新版本中新增的一种策略，其主要实现了两个目的：
>    -- 将现有的分区尽可能均衡的分配给各个consumer，存在此目的的原因在于Round Robin和Range分配策略实际上都会导致某几个consumer承载过多的分区，从而导致消费压力不均衡；
>    -- 如果发生再平衡，那么在重新分配前的基础上会尽力保证当前未宕机的consumer所消费的分区不会被分配给其他的consumer上；
> 
> 尽量避免reblance：
> 
> 首先，Rebalance过程对Consumer Group消费过程有极大的影响。如果你了解JVM的垃圾回收机制，你一定听过万物静止的收集方式，即著名的stop the world，简称STW。在STW期间，所有应用线程都会停止工作，表现为整个应用程序僵在那边一动不动。Rebalance过程也和这个类似，在Rebalance过程中，所有Consumer实例都会停止消费，等待Rebalance完成。这是Rebalance为人诟病的一个方面。
> 
> 其次，目前Rebalance的设计是所有Consumer实例共同参与，全部重新分配所有分区。其实更高效的做法是尽量减少分配方案的变动。例如实例A之前负责消费分区1、2、3，那么Rebalance之后，如果可能的话，最好还是让实例A继续消费分区1、2、3，而不是被重新分配其他的分区。这样的话，实例A连接这些分区所在Broker的TCP连接就可以继续用，不用重新创建连接其他Broker的Socket资源。
> 
> 最后，Rebalance实在是太慢了。曾经，有个国外用户的Group内有几百个Consumer实例，成功Rebalance一次要几个小时！这完全是不能忍受的。最悲剧的是，目前社区对此无能为力，至少现在还没有特别好的解决方案。所谓“本事大不如不摊上”，也许最好的解决方案就是避免Rebalance的发生吧。

## 待处理笔记：

- Kafka是一个消息代理

- Kafka将消息存储在主题中，并从主题检索消息。消息的生产者和消费者之间不会直接连接。此外，Kafka并不会保持有关生产者和消费者的任何状态，它仅作为一个消息交换中心。主题可以视为按名称分隔的日志。

- Kafka是一个日志
  
  Kafka主题底层的技术是日志，它是Kafka追加输入记录的文件。kafka的日志是一种只能追加、完全按照时间顺序排列的记录序列。

- Kafka日志工作原理
  
  当安装Kafka时，其中一个配置项是log.dir，该配置项用来指定Kafka存储日志数据的路径。每个主题都映射到指定日志路径下的一个子目录。子目录数与主题对应的分区数相同，目录名格式为“主题名_分区编号”。每个目录里面存放的都是用于追加传入消息的日志文件，一旦日志文件达到某个规模（磁盘上的记录总数或者记录的大小），或者消息的时间戳间的时间间隔达到了所配置的时间间隔时，日志文件就会被切分，传入的消息将会被追加到一个新的日志文件中
  
  > logs
  > 
  > logs/topicA_0   A有1个分许
  > 
  > logs/topicB_0   B有3个分区
  > 
  > logs/topicB_1
  > 
  > logs/topicB_2

- Kafka和分区
  
  分区是Kafka设计的一个重要部分，它对性能来说必不可少。分区保证了同一个键的数据将会按序被发送给同一个消费者。
  
  对主题作分区的本质是将发送到主题的数据切分到多个平行流之中，这是Kafka能够实现巨大吞吐量的关键。我们解释过每个主题就是一个分布式日志，每个分区类似于一个它自己的日志，并遵循相同的规则。Kafka将每个传入的消息追加到日志末尾，并且所有的消息都严格按时间顺序排列，每条消息都有一个分配给它的偏移量。Kafka不保证跨分区的消息有序，但是能够保证每个分区内的消息是有序的。
  
  除了增加吞吐量，分区还有另一个目的，它允许主题的消息分散在多台机器上，这样给定主题的容量就不会局限于一台服务器上的可用磁盘空间。

- 分区按键对数据进行分组
  
  Kafka处理键/值对格式的数据，如果键为空，那么生产者将采用轮询（round-robin）方式选择分区写入记录。如果键不为空，Kafka会使用以下公式（如下伪代码所示）确定将键/值对发送到哪个分区。通过使用确定性方法来选择分区，使得具有相同键的记录将会按序总是被发送到同一个分区。默认的分区器使用此方法，如果需要使用不同的策略选择分区，则可以提供自定义的分区器。

- 自定义分区器
  
  组合键分区识就需要自定义，可以使用下面配置指定一个自定义分区器
  
  partitioner.class=bbejeck_2.partitioner.PurchaseKeyPartitioner

- 确定恰当的分区数
  
  在创建主题时决定要使用的分区数既是一门艺术也是一门科学。其中一个重要的考虑因素是流入该主题的数据量。更多的数据意味着更多的分区以获得更高的吞吐量，但与生活中的任何事物一样，也要有取舍。
  
  增加分区数的同时也增加了TCP连接数和打开的文件句柄数。此外，消费者处理传入记录所花费的时间也会影响吞吐量。如果消费者线程有重量级处理操作，那么增加分区数可能有帮助，但是较慢的处理操作最终将会影响性能。

- 分布式日志
  
  当对主题进行分区时，Kafka不会将这些分区分布在一台服务上，而是将分区分散到集群中的多台服务器上。由于Kafka是在日志中追加记录，因此Kafka通过分区将这些记录分发到多台服务器上。

- 控制器
  
  Kafka使用ZooKeeper来选择代理控制器，如果代理控制器发生故障或者由于任何原因而不可用时，ZooKeeper从与领导者保持同步的一系列代理（已同步的副本[ISR]）中选出一个新的控制器。控制器的职责是为一个主题的所有分区建立领导者分区和追随者分区的关系。

- 日志管理-日志删除
  
  当一条新消息到达时，如果它的时间戳大于日志中第一个消息的时间戳加上log.roll.ms配置项配置的值时，Kafka就会切分日志。此时，日志被切分，一个新的日志段会被创建并作为一个活跃的日志段，而以前的活跃日志段仍然为消费者提供消息检索，日志切分有两个可选的配置项。
  
  log.roll.ms——这个是主配置项，但没有默认值。
  log.roll.hours——这是辅助配置项，仅当log.roll.ms没有被设置时使用，该配置项默认值是168小时。
  
  与日志切分一样，日志段的删除也基于消息的时间戳，而不仅是时钟时间或文件最后被修改的时间，日志段的删除基于日志中最大的时间戳。用来设置日志段删除策略的3个配置项按优先级依次列出如下，这里按优先级排列意味着排在前面的配置项会覆盖后面的配置项。
  
  log.retention.ms——以毫秒（ms）为单位保留日志文件的时长。
  log.retention.minutes——以分钟（min）为单位保留日志文件的时长。
  log.retention.hours——以小时（h）为单位保留日志文件。
  
  提出这些设置的前提是基于大容量主题的假设，这里大容量是指在一个给定的时间段内保证能够达到文件最大值。另一个配置项log.retention.bytes，可以指定较长的切分时间阈值，以控制I/O操作。最后，为了防止日志切分阈值设置得相对较大而出现日志量显著增加的情况，请使用配置项log.segment.bytes来控制单个日志段的大小。
  
  对于键为空的记录以及独立的记录[12]，删除日志的效果很好。但是，如果消息有键并需要预期的更新操作，那么还有一种方法更适合。

- 日志管理-日志压缩
  
  log.cleanup.policy=compact 根据日志的键值进行数据的更新，压缩支持按照主题配置
  
  删除操作会为给定键设置一个null值，作为一个墓碑标记。任何值为null的键都确保先前与其键相同的记录被去除，之后墓碑标记自身也会被去除。

- 生产者发送消息
  
  ```java
  Properties properties = new Properties();
  //后边可以使用一个逗号分隔的列表
  properties.put("bootstrap.servers", "localhost:9092");
  //KV序列化器
  properties.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
  properties.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
  //all:需要代理接收到所有追随者都以提交记录的确认
  //1:不等待追随者
  //0:不等待任何确认
  properties.put("acks", "1");
  //重试：如果记录的顺讯很重要，需要设置max.in.flight.requests.connect=1，以防止失败的记录在重试发送之前第二批记录成功发送
  properties.put("retries", "3");
  //数据压缩
  properties.put("compression.type", "snappy");
  //分区器类
  properties.put("partitioner.class",PurchaseKeyPartitioner.class.getName());　　
  PurchaseKey key = new PurchaseKey("12334568", new Date());
  
  try(Producer<PurchaseKey, String> producer = new KafkaProducer<>(properties)) {
     ProducerRecord<PurchaseKey, String> record = new ProducerRecord<>("transactions", key, {\"item\":\"book\",\"price\":10.99}");　　
     Callback callback = (metadata, exception) -> {
               if (exception != null) {
                  System.out.println("Encountered exception " + exception); 　　
              }
         };
      //生产者线程安全，一旦生产者将记录放到内部缓冲区，就立即返回Producer.send。缓冲区批量发送记录，具体取决于配置，如果在生产者缓冲区满时尝试发送消息，则可能会有阻塞。这里描述的Producer.send方法接受一个Callback实例，一旦领导者代理确认收到记录，生产者就会触发Callback.onComplete方法                                                                                   
      Future<RecordMetadata> sendFuture = producer.send(record, callback); 　
  }
  ```

- 指定分区
  
  希望所有分区收到的数据量大致相同
  
  ```java
  AtomicInteger partitionIndex = new AtomicInteger(0);  　　
  int currentPartition = Math.abs(partitionIndex.getAndIncrement())%numberPartitions; 
  ProducerRecord<String, String> record =  new ProducerRecord<>("topic", currentPartition, "key", "value");”
  ```

- 时间戳
  
  如果ProducerRecord对象设置了时间戳，那就使用这个时间戳
  
  如果主题设置了时间戳优先用主题的，否则使用代理级别的
  
  代理级别的log.message.timestamp.type配置可以被设置为CreateTime和LogAppendTime中的LogAppendTime被认为是“处理时间”，而CreateTime被认为是“事件时间。

- 消费者读取消息
  
  偏移量唯一标识消息，并表示消息在日志中的起始位置。消费者需要周期性地提交它们已接收到的消息的偏移量。提交一是完全处理了消息，二是发生故障或者重启时该消费者消费的起始位置。故障后消费者从何处开始消费消息取决于具体的配置。auto.offset.reset=
  
  - earliest从最早可用偏移量开始检索
  - latest从最新的偏移量开始检索，本质上是加入时间带你开始消费
  - none不指定重置策略，代理将会向消费者抛出异常

## 消息队列选型

- Kafka：追求高吞吐量，一开始的目的就是用于日志收集和传输，**适合产生大量数据的互联网服务的数据收集业务**，大型公司建议可以选用，**如果有日志采集功能，肯定是首选 kafka。**
- RocketMQ：**天生为金融互联网领域而生，对于可靠性要求很高的场景**，尤其是电商里面的订单扣款，以及业务削峰，在大量交易涌入时，后端可能无法及时处理的情况。RoketMQ 在稳定性上可能更值得信赖，这些业务场景在阿里双 11 已经经历了多次考验，**如果你的业务有上述并发场景，建议可以选择 RocketMQ。**
- RabbitMQ：结合 erlang 语言本身的并发优势，性能较好，社区活跃度也比较高，但是不利于做二次开发和维护，不过 RabbitMQ 的社区十分活跃，可以解决开发过程中遇到的 bug。**如果你的数据量没有那么大，小公司优先选择功能比较完备的 RabbitMQ。**

## 如何自己设计一个消息队列

https://mp.weixin.qq.com/s/hd8z2F1hLbrykxZL6gQb1g

https://mp.weixin.qq.com/s/YJUB2I1Vd87oVaA_07GwOQ

可以参考的面试题：https://mp.weixin.qq.com/s/gm-c3gikWU0MwEJW_QIDFQ

# kafka的一些坑

https://mp.weixin.qq.com/s/YPkE3Tsu3RVbhfVZCBt1pQ

## 背景

一个后厨的划菜系统，用户点菜下单后通过kafka发到该系统，该系统做完逻辑处理以后展示到划菜客户端，厨师就知道哪个订单需要做哪些菜，有些菜做好了，就可以通过该系统出菜。系统自动通知服务员上菜，如果服务员上完菜，修改菜品上菜状态，用户就知道哪些菜已经上了，哪些还没有上。

## 顺序问题

要保证消息顺序，因为订单有下单、支付、完成、撤销等，不可能在还没下单就先读取支付或者撤销的消息，这样的话数据会产生混乱。

- 刚开始将同一个`商户编号`的消息写到同一个partion,topic`中创建了`4`个`partition`，然后部署了`4`个消费者节点，构成`消费者组`，一个`partition对应一个消费者节点。从理论上说，这套方案是能够保证消息顺序的。
- 网络不稳定时，下单消息由于网络处理失败了，后面的支付、完成数据无法入库，因为只有”下单“消息的数据才是完整的数据，其他类型的消息只会更新状态。刚开始采用重试机制，但是同步重试会影响消费速度、降低吞吐量，如此需要异步重试，处理失败的消息需要保存到重试表，消费者在处理消息时，先判断该`订单号`在`重试表`有没有数据，如果有则直接把当前消息保存到`重试表`。如果没有，则进行业务处理，如果出现异常，把该消息保存到`重试表`。
- 消息积压

## 消息积压

- 消息体过大，订单系统发送的消息体只用包含：id和状态等关键信息，后厨显示系统消费消息后，通过id调用订单系统的订单详情查询接口获取数据。后厨显示系统判断数据库中是否有该订单的数据，如果没有则入库，有则更新。
- 路由规则不合理，单partion上有积压，由于某几个商户订单量很大被分到同一个partion，用订单号做路由相对更均匀，不会出现单个订单发消息次数特别多的情况。
- 批量操作引起的连锁反应，上游批量更新商户的订单，调partion、增加consumer都不行（如果一个consumer消费多个partion还是可以的），只有用多线程处理但是频繁调用上游接口导致上游崩溃。对于严格保证消息顺序的场景，可以将线程池改为多个队列（根据id哈希到不同的队列），每个队列用单线程处理。
- 表过大导致的消息积压，对表数据做归档

## 主键冲突

- 代码逻辑会先根据主键从表中查询订单是否存在，如果存在则更新状态，不存在才插入数据，没得问题。这种判断在并发量不大时，是有用的。但是如果在高并发的场景下，两个请求同一时刻都查到订单不存在，一个请求先插入数据，另一个请求再插入数据时就会出现主键冲突的异常。
- 使用数据库悲观锁肯定是不行的，太影响性能。加数据库乐观锁，基于版本号判断，一般用于更新操作，像这种插入操作基本上不会用。剩下的只能用分布式锁了，加分布式锁也可能会影响消费者的消息处理速度。最后选择使用mysql的`INSERT INTO ...ON DUPLICATE KEY UPDATE`语法。

## 数据库主从延迟

- 主从延迟导致的数据读不到，也加了`重试机制`。调用接口查询数据时，如果返回数据为空，或者只返回了订单没有菜品，则加入`重试表`。

## 重复消费

- 自己做幂等

## 多环境消费

- prod的数据被test环境消费了，重置offset

# 其它

## Kafka命令行

```shell
bin/zookeeper-server-start.sh config/zookeeper.properties #启动zk

bin/kafka-server-start.sh config/server.properties #启动kafka服务端

bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092 #创建topic

bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092 #查看topic的描述

bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092#启动生产者

bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092#启动消费者

bin/kafka-topics.sh --zookeeper localhost:2181 --list#查看kafka所有的topic
kafka-topics.sh --zookeeper zookeeper:2181 --list#当在容器中时执行这一条

bin/kafka-topics.sh --zookeeper localhost:2181 --topic hotitems --describe#查看kafka某个topic的信息

bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list#查看消费者组

bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --delete --group {消费组}
#删除某个消费者

bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group console-consumer-68915  #查看某个消费者组的情况
GROUP   console-consumer-68915   #消费者组id            
TOPIC     hotitems      #消费组消费的主题
PARTITION  0                        #分区
CURRENT-OFFSET  -                #当前偏移
LOG-END-OFFSET  11613   #下一条偏移
LAG         -           #
CONSUMER-ID  consumer-console-consumer-68915-1-7f757591-d038-45be-840e-9fd6433aa92d#消费者id                                                        
HOST         /127.0.0.1      #主机
CLIENT-ID            consumer-console-consumer-68915-1    #客户端id

#后台启动kafka
cd /usr/local/kafka_2.12-2.5.0
bin/zookeeper-server-start.sh -daemon config/zookeeper.properties
bin/kafka-server-start.sh -daemon config/server.properties

#关闭kafka
cd /usr/local/kafka_2.12-2.5.0
bin/kafka-server-stop.sh config/server.properties
bin/zookeeper-server-stop.sh
```

## 无消息丢失配置

> 看完这两个案例之后，我来分享一下Kafka无消息丢失的配置，每一个其实都能对应上面提到的问题。
> 
> 1. 不要使用producer.send(msg)，而要使用producer.send(msg, callback)。记住，一定要使用带有回调通知的send方法。
> 2. 设置acks = all。acks是Producer的一个参数，代表了你对“已提交”消息的定义。如果设置成all，则表明所有副本Broker都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。
> 3. 设置retries为一个较大的值。这里的retries同样是Producer的参数，对应前面提到的Producer自动重试。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了retries > 0的Producer能够自动重试消息发送，避免消息丢失。
> 4. 设置unclean.leader.election.enable = false。这是Broker端的参数，它控制的是哪些Broker有资格竞选分区的Leader。如果一个Broker落后原先的Leader太多，那么它一旦成为新的Leader，必然会造成消息的丢失。故一般都要将该参数设置成false，即不允许这种情况的发生。
> 5. 设置replication.factor >= 3。这也是Broker端的参数。其实这里想表述的是，最好将消息多保存几份，毕竟目前防止消息丢失的主要机制就是冗余。
> 6. 设置min.insync.replicas > 1。这依然是Broker端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于1可以提升消息持久性。在实际环境中千万不要使用默认值1。
> 7. 确保replication.factor > min.insync.replicas。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成replication.factor = min.insync.replicas + 1。
> 8. 确保消息消费完成再提交。Consumer端有个参数enable.auto.commit，最好把它设置成false，并采用手动提交位移的方式。就像前面说的，这对于单Consumer多线程处理的场景而言是至关重要的。

> 为什么用消息队列？
> 
> 消息队列有什么优缺点？
> 
> 如何保证kafka高可用？
> 
> 消息是否会丢？如何保证消息不重不丢？
> 
> 如何保证消费顺序性？消费幂等性？
> 
> 消息积压怎么处理？
> 
> 如何设计消息队列？
> 
> kafka基本概念
> 
> kafka消费语义
> 
> offset手动提交与自动提交
> 
> 优化kafka写入速度
> 
> ack为0,1，-1含义
> 
> unclean配置
> 
> leader崩溃时，ISR为空怎么办
> 
> message格式
> 
> kafka为什么快
> 
> 提交消费位移是offset还是offset+1
> 
> kafka是pull还是push
> 
> zk在kafka中的作用
> 
> kafka的reblance
> 
> 消息队列技术选型
> 
> 自己设计消息队列
