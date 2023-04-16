## ES基本概念

- 分布式为文档存储引擎、搜索引擎和分析引擎

- 分布式，支持PB级数据

- 近实时

  >- 从写入数据到数据可以被搜索到有一个小延迟(大概是 1s) 
  >
  >- 基于 ES 执行搜索和分析可以达到秒级

- Cluster

  >集群包含多个节点，每个节点属于哪个集群都是通过一个配置来决定的，对于中小型应用来 说，刚开始一个集群就一个节点很正常。

- Node

  >Node 是集群中的一个节点，节点也有一个名称，默认是随机分配的。默认节点会去加入一个名 称为 elasticsearch 的集群。如果直接启动一堆节点，那么它们会自动组成一个 elasticsearch 集群，当然一个节点也可以组成 elasticsearch 集群。

- Doucement(类似于数据行) & field(类似于字段)

  >文档是 ES 中最小的数据单元，一个 document 可以是一条客户数据、一条商品分类数据、一条 订单数据，通常用 json 数据结构来表示。每个 index 下的 type，都可以存储多条 document。 一个 document 里面有多个 field，每个 field 就是一个数据字段。
  >
  >```json
  >{
  >	 "product_id": "1", 
  >   "product_name": "iPhone X", 
  >   "product_desc": "苹果手机", 
  >   "category_id": "2", 
  >   "category_name": "电子产品"
  >}
  >```

- Index(类似于数据库)

  > 索引包含了一堆有相似结构的文档数据，比如商品索引。一个索引包含很多 document，一个 索引就代表了一类相似或者相同的 ducument。

- Type(类似于数据表，ES7中已经移除)

  > 类型，每个索引里可以有一个或者多个 type，type 是 index 的一个逻辑分类，比如商品 index 下有多个 type:日化商品 type、电器商品 type、生鲜商品 type。每个 type 下的 document 的 field 可能不太一样。
  >
  > ES7中移除的原因：[来自](https://blog.csdn.net/wangzhen3798/article/details/89503707)
  >
  > 起初，我们说"索引"和关系数据库的“库”是相似的，“类型”和“表”是对等的。
  > 这是一个不正确的对比，导致了不正确的假设。在关系型数据库里,"表"是相互独立的,一个“表”里的列和另外一个“表”的同名列没有关系，互不影响。但在类型里字段不是这样的。
  >
  > 在一个Elasticsearch索引里，所有不同类型的同名字段内部使用的是同一个lucene字段存储。也就是说，上面例子中，user类型的user_name字段和tweet类型的user_name字段是存储在一个字段里的，两个类型里的user_name必须有一样的字段定义。
  >
  > 这可能导致一些问题，例如你希望同一个索引中"deleted"字段在一个类型里是存储日期值，在另外一个类型里存储布尔值。
  >
  > 最后,在同一个索引中，存储仅有小部分字段相同或者全部字段都不相同的文档，会导致数据稀疏，影响Lucene有效压缩数据的能力。

- shard

  >单台机器无法存储大量数据，ES 可以将一个索引中的数据切分为多个 shard，分布在多台服务 器上存储。有了 shard 就可以横向扩展，存储更多数据，让搜索和分析等操作分布到多台服务 器上去执行，提升吞吐量和性能。每个 shard 都是一个 lucene index。

- replica

  > 任何一个服务器随时可能故障或宕机，此时 shard 可能就会丢失，因此可以为每个 shard 创建 多个 replica 副本。replica 可以在 shard 故障时提供备用服务，保证数据不丢失，多个 replica 还可以提升搜索操作的吞吐量和性能。primary shard(建立索引时一次设置，不能修改，默认 5 个)，replica shard(随时修改数量，默认 1 个)，默认每个索引 10 个 shard，5 个 primary shard，5个 replica shard，最小的高可用配置，是 2 台服务器。这么说吧，shard 分为 primary shard 和 replica shard。而 primary shard 一般简称为 shard，而 replica shard 一般简称为 replica。

## ES分布式架构原理

- ES 中存储数据的基本单位是Index

- 一个索引会有Type（7已经废弃，可以认为一个Index只有一个Type）

- 索引里边存放Document

- 每个Document有多个Field

- 一个索引可以拆分成多个shard，每个shard存储部分数据

  >拆分多个 shard 是有好处的，一是 ，比如你数据量是 3T，3 个 shard，每个 shard 就 1T 的数据， 若现在数据量增加到 4T，怎么扩展，很简单，重新建一个有 4 个 shard 的索引，将数据导进 去;二是 ，数据分布在多个 shard，即多台服务器上，所有的操作，都会在多台机器 上并行分布式执行，提高了吞吐量和性能。

- 每个shard都有多个备份

  >接着就是这个 shard 的数据实际是有多个备份，就是说每个 shard 都有一个 primary shard ，负责写入数据，但是还有几个 。 primary shard 写入数据之后，会将数 据同步到其他几个 上去。
  >
  >ES 集群多个节点，会自动选举一个节点为 master 节点，这个 master 节点其实就是干一些管理 的工作的，比如维护索引元数据、负责切换 primary shard 和 replica shard 身份等。要是 master 节点宕机了，那么会重新选举一个节点为 master 节点。
  >
  >如果是非 master节点宕机了，那么会由 master 节点，让那个宕机节点上的 primary shard 的身 份转移到其他机器上的 replica shard。接着你要是修复了那个宕机机器，重启了之后，master 节点会控制将缺失的 replica shard 分配过去，同步后续修改的数据之类的，让集群恢复正常。

## ES 写入数据的过程

- 客户端选择一个 node 发送请求过去，这个 node 就是 coordinating node (协调节点)。
- coordinating node 对 document 进行路由 ，将请求转发给对应的 node(有 primary shard)。
- 实际的node上的primary shard处理请求，然后将数据同步到replica node。
- coordinating node如果发现primary node 和所有 replica node都搞定之后，就返回响应结果给客户端。

## ES 查询数据的过程

可以通过 doc id 来查询，会根据 doc id 进行 hash，判断出来当时把 doc id 分配到了 哪个 shard 上面去，从那个 shard 去查询。

- 客户端发送请求到 一个 node，成为 coordinate node 。
-  coordinate node 对 doc_id 进行哈希路由，将请求转发到对应的 node，此时会使用round-robin ，在 primary shard 以及其所有 replica 中随机选择一个， 让读请求负载均衡。
- 接收请求的 node 返回 document 给 coordinate node。 
- coordinate node 返回 document 给客户端。

## ES搜索数据过程

es 最强大的是做全文检索：

- 客户端发送请求到一个 coordinate node 。
- 协调节点将搜索请求转发到 的 shard 对应的 primary shard 或 replica shard ， 都可以。
- query phase:每个 shard 将自己的搜索结果(其实就是一些 doc id )返回给协调节点， 由协调节点进行数据的合并、排序、分页等操作，产出最终结果。
- fetch phase:接着由协调节点根据 doc id 去各个节点上 的 document 数据， 最终返回给客户端。

## ES写数据底层原理

- 概念
  **segment file**: 存储倒排索引的文件，每个segment本质上就是一个倒排索引，每秒都会生成一个segment文件，当文件过多时es会自动进行segment merge（合并文件），合并时会同时将已经标注删除的文档物理删除

  >数据写入segment时就建立好了倒排索引

  **commit point（重点理解）**: 记录当前所有可用的segment，每个commit point都会维护一个.del文件，即每个.del文件都有一个commit point文件（es删除数据本质是不属于物理删除），当es做删改操作时首先会在.del文件中声明某个document已经被删除，文件内记录了在某个segment内某个文档已经被删除，当查询请求过来时在segment中被删除的文件是能够查出来的，但是当返回结果时会根据commit point维护的那个.del文件把已经删除的文档过滤掉

  **translog日志文件**: 为了防止elasticsearch宕机造成数据丢失保证可靠存储，es会将每次写入数据同时写到translog日志中。

  **os cache**的理解：操作系统里面，磁盘文件其实都有一个东西，叫做os cache，操作系统缓存，就是说数据写入磁盘文件之前，会先进入os cache，先进入操作系统级别的一个内存缓存中去

- 写入过程

  1. 先写入buffer，在buffer里的时候数据是搜索不到的；同时将数据写入translog日志文件

  2. 如果buffer快满了，或者到一定时间，就会将buffer数据refresh到一个新的segment file中，但是此时数据不是直接进入segment file的磁盘文件的，而是先进入os cache的。这个过程就是refresh，每隔1秒钟，es将buffer中的数据写入一个新的segment file，每秒钟会产生一个新的磁盘文件，segment file，这个segment file中就存储最近1秒内buffer中写入的数据但是如果buffer里面此时没有数据，那当然不会执行refresh操作咯，每秒创建换一个空的segment file，如果buffer里面有数据，默认1秒钟执行一次refresh操作，刷入一个新的segment file中。只要buffer中的数据被refresh操作，刷入os cache中，就代表这个数据就可以被搜索到了。为什么叫es是准实时的？NRT，near real-time，准实时。默认是每隔1秒refresh一次的，所以es是准实时的，因为写入的数据1秒之后才能被看到。可以通过es的restful api或者java api，手动执行一次refresh操作，就是手动将buffer中的数据刷入os cache中，让数据立马就可以被搜索到。只要数据被输入os cache中，buffer就会被清空了，因为不需要保留buffer了，数据在translog里面已经持久化到磁盘去一份了

  3. 只要数据进入os cache，此时就可以让这个segment file的数据对外提供搜索了。

  4. 重复1~3步骤，新的数据不断进入buffer和translog，不断将buffer数据写入一个又一个新的segment file中去，每次refresh完buffer清空，translog保留。随着这个过程推进，translog会变得越来越大。当translog达到一定长度的时候，就会触发commit操作。buffer中的数据，每隔1秒就被刷到os cache中去，然后这个buffer就被清空了。所以说这个buffer的数据始终是可以保持住不会填满es进程的内存的。每次一条数据写入buffer，同时会写入一条日志到translog日志文件中去，所以这个translog日志文件是不断变大的，当translog日志文件大到一定程度的时候，就会执行commit操作。

  5. commit操作发生第一步，就是将buffer中现有数据refresh到os cache中去，清空buffer

  6. 将一个commit point写入磁盘文件，里面标识着这个commit point对应的所有segment file

  7. 强行将os cache中目前所有的数据都fsync到磁盘文件中去，translog日志文件的作用是什么？就是在你执行commit操作之前，数据要么是停留在buffer中，要么是停留在os cache中，无论是buffer还是os cache都是内存，一旦这台机器死了，内存中的数据就全丢了。所以需要将数据对应的操作写入一个专门的日志文件，translog日志文件中，一旦此时机器宕机，再次重启的时候，es会自动读取translog日志文件中的数据，恢复到内存buffer和os cache中去。commit操作：

     1、写commit point；

     2、将os cache数据fsync强刷到磁盘上去；

     3、清空translog日志文件

  8. 将现有的translog清空，然后再次重启启用一个translog，此时commit操作完成。默认每隔30分钟会自动执行一次commit，但是如果translog过大，也会触发commit。整个commit的过程，叫做flush操作。我们可以手动执行flush操作，就是将所有os cache数据刷到磁盘文件中去。不叫做commit操作，flush操作。es中的flush操作，就对应着commit的全过程。我们也可以通过es api，手动执行flush操作，手动将os cache中的数据fsync强刷到磁盘上去，记录一个commit point，清空translog日志文件。

  9. translog其实也是先写入os cache的，默认每隔5秒刷一次到磁盘中去，所以默认情况下，可能有5秒的数据会仅仅停留在buffer或者translog文件的os cache中，如果此时机器挂了，会丢失5秒钟的数据。但是这样性能比较好，最多丢5秒的数据。也可以将translog设置成每次写操作必须是直接fsync到磁盘，但是性能会差很多。

  >**总结**
  >
  >  es里的写流程，有4个底层的核心概念，refresh、flush、translog、merge

## 删除、更新数据原理

1. 如果是删除操作，commit的时候会生成一个.del文件，里面将某个doc标识为deleted状态，那么搜索的时候根据.del文件就知道这个doc被删除了
2. 如果是更新操作，就是将原来的doc标识为deleted状态，然后新写入一条数据
3.  buffer每次refresh一次，就会产生一个segment file，所以默认情况下是1秒钟一个segment file，segment file会越来越多，此时会定期执行merge。
4. 每次merge的时候，会将多个segment file合并成一个，同时这里会将标识为deleted的doc给物理删除掉，然后将新的segment file写入磁盘，这里会写一个commit point，标识所有新的segment file，然后打开segment file供搜索使用，同时删除旧的segment file

## 底层的 Lucene

简单来说，lucene 就是一个 jar 包，里面包含了封装好的各种建立倒排索引的算法代码。我们 用 Java 开发的时候，引入 lucene jar，然后基于 lucene 的 api 去开发就可以了。通过 lucene，我们可以将已有的数据建立索引，lucene 会在本地磁盘上面，给我们组织索引的 数据结构。

## 倒排索引

在搜索引擎中，每个文档都有一个对应的文档 ID，文档内容被表示为一系列关键词的集合。例 如，文档 1 经过分词，提取了 20 个关键词，每个关键词都会记录它在文档中出现的次数和出现位置。那么，倒排索引就是 中都出现了关键词。另外，实用的倒排索引还可以记录更多的信息，比如文档频率信息，表示在文档集合中有多少 个文档包含某个单词。

> 要注意倒排索引的两个重要细节:
>
> - 倒排索引中的所有词项对应一个或多个文档; 
> - 倒排索引中的词项根据字典序升序排列

## 性能优化

es 的搜索引擎严重依赖于底层的filesystem cache 的内存，你如果给 filesystem cache 更多的内存，尽量让内存可以容纳所有的idx segment file索引数据文件，那么你搜索的时候就基本都是走内存的，性能会非常高。性能差距究竟可以有多大?我们之前很多的测试和压测，如果走磁盘一般肯定上秒，搜索性能 绝对是秒级别的，1秒、5秒、10秒。但如果是走 filesystem cache ，是走纯内存的，那么 一般来说性能比走磁盘要高一个数量级，基本上就是毫秒级的，从几毫秒到几百毫秒不等。

- 要让 es 性能要好，最佳的情况下，就是你的机器的内存，至少可以容纳你的总数 据量的一半

- 仅仅写入 es 中要用来检索的就可以了，可以把其他的字段数据存在 mysql/hbase 里，一般是建议用es+hbase架 构。

  >hbase 的特点是适用于海量数据在线存储 ，就是对hbase可以写入海量数据，但是不要做复杂的搜索，做很简单的一些根据 id 或者范围进行查询的这么一个操作就可以了。从 es 中根 据 name 和 age 去搜索，拿到的结果可能就 20 个 doc id ，然后根据 doc id 到 hbase 里去 查询每个 doc id 对应的 ，给查出来，再返回给前端。
  >
  >写入 es 的数据最好小于等于，或者是略微大于 es 的 filesystem cache 的内存容量。然后你从 es 检索可能就花费 20ms，然后再根据 es 返回的 id 去 hbase 里查询，查 20 条数据，可能也就耗 费个 30ms，可能你原来那么玩儿，1T 数据都放 es，会每次查询都是 5~10s，现在可能性能就 会很高，每次查询就是 50ms。

- 数据预热

  假如按照上述的方案去做了，es 集群中每个机器写入的数据量还是超过了 filesystem cache 一倍，那么可以做数据预热。

  > 举个例子，拿微博来说，你可以把一些大V，平时看的人很多的数据，你自己提前后台搞个系 统，每隔一会儿，自己的后台系统去搜索一下热数据，刷到 filesystem cache 里去，后面 用户实际上来看这个热数据的时候，他们就是直接从内存里搜索了，很快。或者是电商，你可以将平时查看最多的一些商品，比如说 iphone 8，热数据提前后台搞个程 序，每隔 1 分钟自己主动访问一次，刷到 filesystem cache 里去。对于那些你觉得比较热的、经常会有人访问的数据，最好 ，就 是对热数据每隔一段时间，就提前访问一下，让数据进入 filesystem cache 里面去。这样 下次别人访问的时候，性能一定会好很多。

- 冷热分离

  es 可以做类似于 mysql 的水平拆分，就是说将大量的访问很少、频率很低的数据，单独写一个 索引，然后将访问很频繁的热数据单独写一个索引。最好是将冷数据写入一个索引，热数据写入一个索引，这样可以确保热数据在被预热之后，尽量都让他们留在 filesystem os cache 里， 。

  > 假设你有 6 台机器，2 个索引，一个放冷数据，一个放热数据，每个索引 3 个 shard。3 台机器放热数据 index，另外 3 台机器放冷数据 index。然后这样的话，你大量的时间是在访问 热数据 index，热数据可能就占总数据量的 10%，此时数据量很少，几乎全都保留在filesystem cache 里面了，就可以确保热数据的访问性能是很高的。但是对于冷数据而言， 是在别的 index 里的，跟热数据 index 不在相同的机器上，大家互相之间都没什么联系了。如 果有人访问冷数据，可能大量数据是在磁盘上的，此时性能差点，就 10% 的人去访问冷数据， 90% 的人在访问热数据，也无所谓了。

- 使用好的document模型

  对于 MySQL，我们经常有一些复杂的关联查询。在 es 里该怎么玩儿，es 里面的复杂的关联查 询尽量别用，一旦用了性能一般都不太好。

  最好是先在 Java 系统里就完成关联，将关联好的数据直接写入 es 中。搜索的时候，就不需要 利用 es 的搜索语法来完成 join 之类的关联搜索了。

  document 模型设计是非常重要的，很多操作，不要在搜索的时候才想去执行各种复杂的乱七 八糟的操作。es 能支持的操作就那么多，不要考虑用 es 做一些它不好操作的事情。如果真的 有那种操作，尽量在 document 模型设计的时候，写入的时候就完成。另外对于一些太复杂的 操作，比如 join/nested/parent-child 搜索都要尽量避免，性能都很差的。

- 分页性能优化

  > es 的分页是较坑的，为啥呢?举个例子吧，假如你每页是 10 条数据，你现在要查询第 100 页，实际上是会把每个 shard 上存储的前 1000 条数据都查到一个协调节点上，如果你有个 5 个 shard，那么就有 5000 条数据，接着协调节点对这 5000 条数据进行一些合并、处理，再获取到 最终第 100 页的 10 条数据。

  - 不允许深度分页
  - 类似于app只能滑动下翻
    - scroll api，scroll 会一次性给你生成 ，然后每次滑动向后翻页就是通过 scroll_id 移动，获取下一页下一页这样子，性能会比上面说的那种分页性能要高很多很多，基本上都是毫秒级的。
    - search_after ， search_after 的思想是使用前 一页的结果来帮助检索下一页的数据，显然，这种方式也不允许你随意翻页，你只能一页页往 后翻。初始化时，需要使用一个唯一值的字段作为 sort 字段。

## 简单线上集群配置

>- 5 台机器，每台机器是 6 核 64G 的，集群总内存是 320G。
>- 日增量数据大概是 2000 万条，每天日增量数据大概是 500MB，每月增量数 据大概是 6 亿，15G。目前系统已经运行了几个月，现在 es 集群里数据总量大概是 100G 左 右。
>- 目前线上有 5 个索引(这个结合你们自己业务来，看看自己有哪些数据可以放 es 的)，每 个索引的数据量大概是 20G，所以这个数据量之内，我们每个索引分配的是 8 个 shard，比 默认的5个shard多了3个shard。

## ES的深分页问题

>- From + Size 查询（不推荐）
>
>  - 搜索请求通常跨越多个分片，每个分片必须将其请求的命中内容以及任何先前页面的命中内容加载到内存中
>  - 对于翻页较深的页面或大量结果，这些操作会显著增加内存和 CPU 使用率，从而导致性能下降或节点故障
>
>- Scroll 遍历查询
>
>  需要遍历全量数据场景 
>
>- Search After 查询
>
>  仅需要向后翻页的场景及超过Top 10000 数据需要分页场景
>
>

# 其它

## 安装

```yaml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:7.12.1
    container_name: elasticsearch
    environment:
      - cluster.name=elastic-pro
      - node.name=node_master_9200
      - node.master=true
      - node.data=true
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.type=single-node"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /Users/wyj/Documents/elasticsearch/data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
  kibana:
    image: kibana:7.12.1
    container_name: kibana
    environment:
      - SERVER_NAME=kibana
      - ELASTICSEARCH_URL=elasticsearch:9200
      - XPACK_MONITORING_ENABLED=true
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch
```

Localhost:9200,出现如图所示的信息时启动成功

```json
{
  "name" : "node_master_9200",
  "cluster_name" : "elastic-pro",
  "cluster_uuid" : "qoN8h3UhRyS6qW74LJlq1Q",
  "version" : {
    "number" : "7.12.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "3186837139b9c6b6d23c3200870651f10d3343b7",
    "build_date" : "2021-04-20T20:56:39.040728659Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

localhost:5601使用kibana管理ES，在management的devtools可以运行ES命令

## kibana、es、ik

```shell
docker network create es-net
docker pull elasticsearch:7.12.1
docker run -d \
	--name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1

命令解释：
-e "cluster.name=es-docker-cluster"：设置集群名称
-e "http.host=0.0.0.0"：监听的地址，可以外网访问
-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"：内存大小
-e "discovery.type=single-node"：非集群模式
-v es-data:/usr/share/elasticsearch/data：挂载逻辑卷，绑定es的数据目录
-v es-logs:/usr/share/elasticsearch/logs：挂载逻辑卷，绑定es的日志目录
-v es-plugins:/usr/share/elasticsearch/plugins：挂载逻辑卷，绑定es的插件目录
--privileged：授予逻辑卷访问权
--network es-net ：加入一个名为es-net的网络中
-p 9200:9200：端口映射配置
在浏览器中输入：http://localhost:9200 即可看到elasticsearch的响应结果
```

- 安装kibana

  ```shell
  docker pull kibana:7.12.1
  docker run -d \
  --name kibana \
  -e ELASTICSEARCH_HOSTS=http://es:9200 \
  --network=es-net \
  -p 5601:5601  \
  kibana:7.12.1
  
  --network es-net ：加入一个名为es-net的网络中，与elasticsearch在同一个网络中
  -e ELASTICSEARCH_HOSTS=http://es:9200"：设置elasticsearch的地址，因为kibana已经与elasticsearch在一个网络，因此可以用容器名直接访问elasticsearch
  -p 5601:5601：端口映射配置
  ```

- 安装ik插件

  ```shell
  # 进入容器内部
  docker exec -it elasticsearch /bin/bash
  
  # 在线下载并安装
  ./bin/elasticsearch-plugin  install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip
  
  #退出
  exit
  #重启容器
  docker restart elasticsearch
  ```

  