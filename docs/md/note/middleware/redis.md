

# 常见问题

>redis有哪些数据结构？有哪些底层数据结构？高层数据结构分别在底层怎么实现的？
>
>为什么快（至少5个原因）？单线程与多线程？
>
>淘汰策略？持久化（AOF、RDB）？
>
>主从？哨兵？集群？
>
>

# Redis命令

## 发布订阅

```
subscribe channel [channel ...] 订阅给定的一个或多个频道
unscribe 退订给定的一个或多个频道，如果执行时没有给定任何频道，那么退订所有频道
publish channel message 向给定频道发送信息
psubscribe pattern [pattern ...] 订阅与给定模式相匹配的所有频道
punsubscribe 退订给定的模式，如果执行时没有给定任何模式，那么退订所有模式
```

但是要注意发布订阅带来的风险

1. 如果生产的很快，但是消费的很慢，不断积压的消息会使得输出缓冲区越来越大，最终导致redis速度变慢甚至崩溃。
2. 如果客户端在执行订阅的过程中断线，那么客户端将丢失所有在断线期间的所有消息。

## 排序

```
sort source-key [by pattern][limit offset count][get pattern [get pattern ...]][ASC|DESC][alpha][store dest-key]根据给定的选项、对输入列表、集合或者有序集合进行排序，然后返回或者存储排序的结果
```

## 键的过期时间

```
persist key-name 移除键的过期时间
ttl key-name 查看给定键距离过期还有多少时间
expire key-name seconds 让给定的键在指定的时间之后过期
pttl key-name 查看给定键距离过期时间还有多少毫秒
pexpire key-name milliseconds 让给定键在指定的毫秒数之后过期
pexpireat key-name timestamp 讲一个毫秒级精度的时间戳设置为给定键的过期时间
```

启动redis

```
redis-server
```

# 基础

## 什么是Redis

- 基于键值对（key-value）的NoSQL数据库
- Redis中的value支持多种数据结构，因此 Redis可以满足很多的应用场景
- Redis还可以将内存的数据利用快照和日志的形式保存到硬盘上，这样在发生类似断电或者机器故障的时候，内存中的数据不会“丢失”
- Redis还提供了键过期、发布订阅、事务、流水线、Lua脚本等附加功能

总之，Redis是一款强大的性能利器。

## Redis可以用来干什么

> - 缓存
>   
>   这是Redis应用最广泛地方，基本所有的Web应用都会使用Redis作为缓存，来降低数据源压力，提高响应速度
> 
> - 计数器
>   
>   计数器 Redis天然支持计数功能，而且计数性能非常好，可以用来记录浏览量、点赞量等等
> 
> - 排行榜
>   
>   排行榜 Redis提供了列表和有序集合数据结构，合理地使用这些数据结构可以很方便地构建各种排行榜系统
> 
> - 社交网络
>   
>   社交网络 赞/踩、粉丝、共同好友/喜好、推送、下拉刷新
> 
> - 消息队列
>   
>   消息队列 Redis提供了发布订阅功能和阻塞队列的功能，可以满足一般消息队列功能
> 
> - 分布式锁
>   
>   分布式锁 分布式环境下，利用Redis实现分布式锁，也是Redis常见的应用

Redis的应用一般会结合项目去问，以一个电商项目的用户服务为例：

- Token存储：用户登录成功之后，使用Redis存储Token
- 登录失败次数计数：使用Redis计数，登录失败超过一定次数，锁定账号
- 地址缓存：对省市区数据的缓存
- 分布式锁：分布式环境下登录、注册等操作加分布式锁

## Redis 有哪些数据结构

![图片](https://img-blog.csdnimg.cn/img_convert/16e8c78a3e37494871d42d81c53f4d13.png)-

- string

字符串最基础的数据结构。字符串类型的值实际可以是字符串（简单的字符串、复杂的字符串（JSON、XML））、数字 （整数、浮点数），甚至是二进制（图片、音频、视频），但是值最大不能超过512MB。字符串主要有以下几个典型使用场景：

> 缓存功能、计数、共享Session、限速、分布式锁

```
set hello world   增
get hello         查
del hello         删
incr key  键存储的值加1
decr key  键存储的值减1
incrby key amount 将键存储的值加上整数amount
decrby key amount 将键存储的值减去整数amount
incrbyfloat key amount 将键存储的值加上浮点数amount，2.6以上可用
append key value 将value追加到末尾
getrange key start end 获取包括start和end在内的子串
setrange key offset value 将从start偏移量开始的子串设置为给定值
getbit key offset 将字符串看做二进制，并得到偏移量为offset的二进制位
setbit key offset value 将字符串看做二进制，并将偏移量为offset的二进制位设置为value
bittop operation key1 key2 key3.... 执行二进制运算（and,or,xor,not）结果存于key1
```

- hash

哈希类型是指键值本身又是一个键值对结构，哈希主要有以下典型应用场景：

> 缓存用户信息、缓存对象

```
hset hash x1 y1 增
hget hash x1 查
hgetall hash 得到键值得KV对
hdel hash x1 删
hlen key 返回散列包含的键值对数量
hexists key-name key 检查给定键是否存在于散列中
hkeys key-name 获取散列中的所有键
hvals key-name 获取散列中的所有值
hincrby key-name key increment 将键key上存储的值加上整数increment
hincrbyfloat key-name key increment 将键key上存储的值加上浮点数increment
```

- list

列表（list）类型是用来存储多个有序的字符串。列表是一种比较灵活的数据结构，它可以充当栈和队列的角色。列表主要有以下几种使用场景：

>  消息队列、文章列表、微博关注人时间列表

```
rpush x 1     右增1
rpush x 2     右增1 2
lpush x 3     左增3 1 2
lrange 0 -1   取出列表所有值
lindex x 0    索引0的字符 3
lpop x        左边弹出
rpop x        右边弹出
ltrim key start end 裁剪列表从start到end
blpop key1 key2 ... timeout 从第一个非空列表中弹出位于最左端的元素或者在timeout秒之内阻塞并等待可弹出的元素出现
brpop key1 key2 ... timeout从第一个非空列表中弹出位于最右端的元素或者在timeout秒之内阻塞并等待可弹出的元素出现
rpoplpush key1 key2 从key1弹出最右端的元素并插入key2的最左端
brpoplpush key1 key2 timeout 从key1弹出最有端的元素并插入key2的最左端，如果key1为空，则在timeout秒之内阻塞并等待可弹出的元素出现

对于阻塞弹出命令和弹出推入命令，最常见的用例就是消息传递和任务队列
```

- set

集合（set）类型也是用来保存多个的字符串元素，但和列表类型不一 样的是，集合中不允许有重复元素，并且集合中的元素是无序的。集合主要有如下使用场景：

> 去重、赞、踩、共同好友

```
sadd x 1    向集合添加一个元素
smembers x  查看集合中的所有元素
srem x 1    集合删除一个元素
sismember x 1   检查给定元素是否存在于集合中
scard key 返回集合包含的元素数量
srandmember key count 从集合随机返回一个或多个元素，count为正数时元素不会重复，复数时可能会重复
spop key 随机移除一个元素
smove key1 key2 item 如果key1中存在item元素，将其移到key2
sdiffstore key1 key2 [key3 ... ] 将存在于key2但是不存在于其它集合的元素存在key1里
sinter key1 [key2 ...] 返回同时存在于所有集合的元素
sinterstore key1 key2 [key3 ...] 将同时存在于所有集合的元素存在key1里
sunion key1 [key2 ...] 返回至少存在于一个集合的元素
sunionstore key1 key2 [key2 ...] 将至少存在在于一个集合的元素存在key1中
```

- sorted set

有序集合中的元素可以排序。但是它和列表使用索引下标作为排序依据不同的是，它给每个元素设置一个权重（score）作为排序的依据。有序集合主要应用场景：

> 排行榜、用户点赞统计、用户排序

```
zadd zset 1 x1    增，包含一个分值
zrange zset 0 -1  按照分值大小排序
zrangbyscore zset 0 1 获取给定分值的所有元素
zrem zset x1 删
zcard key-name 返回数量
zincrby key-name increment member 将member成员的分值加上increment
zcount key-name min max 返回分值介于min和max之间的成员数量
zrank key-name member 返回成员member在有序结合中的排名
zscore key-name menmber 返回成员member的分值
zrevrank key-name member 返回member的排名，成员按照分值从大到小排列
zrevrange key-name start stop [withscores] 返回给定排名范围的成员，成员按照分值从大到小排列
zrangebyscore key min max [withscores] [limit offset count] 返回分值介于min与max之间的所有成员
zrevrangebyscore key max min [withscores] [limit offset count] 返回分值介于min和max之间的所有成员，并按照分值从大到小的顺序来返回他们
zremrangebyrank key-name start stop 移除有序集合中排名介于start和stop之间的所有成员
zremrangebyscore key-name min max 移除有序结合中分值介于min和max之间的所有成员
zinterstore，zunionstore执行类似于集合的交集、并集运算
```

- HyperLogLog
  
  Redis2.8.9新添的结构，用来做基数统计，他只会根据输入元素来计算基数，而不会存储输入元素本身，比如根据数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。
  
  ```
  pfadd key element [element...]添加指定元素到HyperLogLog中
  pfcount key [key ...] 返回给定HyperLogLog的基数估算值
  pfmerge destkey sourcekey[sourcekey...] 将多个HyperLogLog合并为一个HyperlogLog
  ```

- 其余不常用的数据结构
  
  ```
  Geo、Pub/Sub、BloomFilter，RedisSearch，Redis-ML
  ```

## Redis为什么快

- 基于内存实现

- IO多路复用模型，非阻塞IO
  
  Redis 线程不会阻塞在某一个特定的监听或已连接套接字上，也就是说，不会阻塞在某一个特定的客户端请求处理上。正因为此，Redis 可以同时和多个客户端连接并处理请求，从而提升并发性。

- 单线程模型，避免了非必要的上下文切换和竞争条件
  
  1. 不会因为线程创建导致的性能消耗；
  2. 避免上下文切换引起的 CPU 消耗，没有多线程切换的开销；
  3. 避免了线程之间的竞争问题，比如添加锁、释放锁、死锁等，不需要考虑各种锁问题。
  4. 代码更清晰，处理逻辑简单。
  
  **单线程是否没有充分利用 CPU 资源呢？**
  
  官方答案：因为 Redis 是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈最**有可能是机器内存的大小或者网络带宽**。既然单线程容易实现，而且 CPU 不会成为瓶颈，那就顺理成章地采用单线程的方案了。

- 高效的数据结构
  
  - 动态字符串
    
    1. 因为保存了字符串长度，可以O(1)获取字符串长度，C语言需要O(n)
    
    2. **空间预分配**
       
       已使用长度、空闲长度，如果对 SDS 修改后，len 的长度小于 1M，那么程序将分配和 len 相同长度的未使用空间。举个例子，如果 len=10，重新分配后，buf 的实际长度会变为 10(已使用空间)+10(额外空间)+1(空字符)=21。如果对 SDS 修改后 len 长度大于 1M，那么程序将分配 1M 的未使用空间。
    
    3. **惰性空间释放**
       
       当对 SDS 进行缩短操作时，程序并不会回收多余的内存空间，而是使用 free 字段将这些字节数量记录下来不释放，后面如果需要 append 操作，则直接使用 free 中未使用的空间，减少了内存的分配。
    
    4. **二进制安全**
       
       在 Redis 中不仅可以存储 String 类型的数据，也可能存储一些二进制数据。
       
       二进制数据并不是规则的字符串格式，其中会包含一些特殊的字符如 '\0'，在 C 中遇到 '\0' 则表示字符串的结束，但在 SDS 中，标志字符串结束的是 len 属性。
  
  - 双向链表
    
    1. 双端：链表节点带有 prev 和 next 指针，获取某个节点的前置节点和后置节点的复杂度都是 O（1）。
    2. 无环：表头节点的 prev 指针和表尾节点的 next 指针都指向 NULL，对链表的访问以 NULL 为终点。
    3. 带表头指针和表尾指针：通过 list 结构的 head 指针和 tail 指针，程序获取链表的表头节点和表尾节点的复杂度为 O（1）。
    4. 带链表长度计数器：程序使用 list 结构的 len 属性来对 list 持有的链表节点进行计数，程序获取链表中节点数量的复杂度为 O（1）。
    5. 多态：链表节点使用 void* 指针来保存节点值，并且可以通过 list 结构的 dup、free、match 三个属性为节点值设置类型特定函数，所以链表可以用于保存各种不同类型的值。
  
  - 压缩链表（新版本已经废弃，由listpack维护）
    
    1. 压缩列表包含了整个压缩列表的字节数、列表尾的偏移量、列表中entry个数、entry数组以及压缩列表的结尾
    2. 如果我们要查找定位第一个元素和最后一个元素，可以通过表头三个字段的长度直接定位，复杂度是 O(1)。而查找其他元素时，就没有这么高效了，只能逐个查找，此时的复杂度就是 O(N)
    3. 压缩列表是 List 、hash、 sorted Set 三种数据类型底层实现之一。
  
  - quicklist
    
    1. 后续版本对列表数据结构进行了改造，使用 quicklist 代替了 ziplist 和 linkedlist。**quicklist 是 ziplist 和 linkedlist 的混合体，它将 linkedlist 按段切分，每一段使用 ziplist 来紧凑存储，多个 ziplist 之间使用双向指针串接起来。**
  
  - 跳表
    
    sorted set 类型的排序功能是通过「跳跃列表」数据结构来实现。
  
  - 整数数组
    
    当一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis 就会使用整数集合作为集合键的底层实现。结构如下：
    
    ```c
    typedef struct intset{
         //编码方式
         uint32_t encoding;
         //集合包含的元素数量
         uint32_t length;
         //保存元素的数组
         int8_t contents[];
    }intset;
    ```
    
    contents 数组是整数集合的底层实现：整数集合的每个元素都是 contents 数组的一个数组项（item），各个项在数组中按值的大小从小到大有序地排列，并且数组中不包含任何重复项。length 属性记录了整数集合包含的元素数量，也即是 contents 数组的长度。

- 根据实际类型选择合理的数据编码
  
  Redis 使用对象（redisObject）来表示数据库中的键值，当我们在 Redis 中创建一个键值对时，至少创建两个对象，一个对象是用做键值对的键对象，另一个是键值对的值对象。
  
  例如我们执行 SET MSG XXX 时，键值对的键是一个包含了字符串“MSG“的对象，键值对的值对象是包含字符串"XXX"的对象。
  
  **redisObject**
  
  ```c
  typedef struct redisObject{
      //类型
     unsigned type:4;
     //编码
     unsigned encoding:4;
     //指向底层数据结构的指针
     void *ptr;
      //...
   }robj;
  ```
  
  其中 type 字段记录了对象的类型，包含字符串对象、列表对象、哈希对象、集合对象、有序集合对象。
  
  对于每一种数据类型来说，底层的支持可能是多种数据结构，什么时候使用哪种数据结构，这就涉及到了编码转化的问题。
  
  那我们就来看看，不同的数据类型是如何进行编码转化的：
  
  **String**：存储数字的话，采用 int 类型的编码，如果是非数字的话，采用 raw 编码；
  
  **List**：List 对象的编码可以是 ziplist 或 linkedlist，字符串长度 < 64 字节且元素个数 < 512 使用 ziplist 编码，否则转化为 linkedlist 编码；
  
  注意：这两个条件是可以修改的，在 redis.conf 中：
  
  ```
  list-max-ziplist-entries 512
  list-max-ziplist-value 64
  ```
  
  **Hash**：Hash 对象的编码可以是 ziplist 或 hashtable。
  
  当 Hash 对象同时满足以下两个条件时，Hash 对象采用 ziplist 编码：
  
  - **Hash 对象保存的所有键值对的键和值的字符串长度均小于 64 字节。**
  - **Hash 对象保存的键值对数量小于 512 个。**
  
  否则就是 hashtable 编码。
  
  **Set**：Set 对象的编码可以是 intset 或 hashtable，intset 编码的对象使用整数集合作为底层实现，把所有元素都保存在一个整数集合里面。
  
  保存元素为整数且元素个数小于一定范围使用 intset 编码，任意条件不满足，则使用 hashtable 编码；
  
  **Zset**：Zset 对象的编码可以是 ziplist 或 zkiplist，当采用 ziplist 编码存储时，每个集合元素使用两个紧挨在一起的压缩列表来存储。
  
  Ziplist 压缩列表第一个节点存储元素的成员，第二个节点存储元素的分值，并且按分值大小从小到大有序排列。
  
  - **Zset 保存的元素个数小于 128。**
  - **Zset 元素的成员长度都小于 64 字节。**
  
  如果不满足以上条件的任意一个，ziplist 就会转化为 zkiplist 编码。注意：这两个条件是可以修改的，在 redis.conf 中

- 官方数据，redis的QPS可以达到100000，实际20主40从，200G内存，300WQPS，1.5亿key占用50G

- redis整体是一个hash，hash里保存的是一个entry，entry保存着指针，时间复杂度是O(1),哈希冲突时通过链式解决冲突，过长时就会导致查找性能变差可能，所以 Redis 为了追求快，使用了两个全局哈希表。用于 rehash 操作，增加现有的哈希桶数量，减少哈希冲突。

- **渐进式 rehash**，每次处理客户端请求的时候，先从 hash 表 1 中第一个索引开始，将这个位置的 所有数据拷贝到 hash 表 2 中，就这样将 rehash 分散到多次请求过程中，避免耗时阻塞。

- 总结
  
  1. 纯内存操作，一般都是简单的存取操作，线程占用的时间很多，时间的花费主要集中在 IO 上，所以读取速度快。
  2. 整个 Redis 就是一个全局 哈希表，他的时间复杂度是 O(1)，而且为了防止哈希冲突导致链表过长，Redis 会执行 rehash 操作，扩充 哈希桶数量，减少哈希冲突。并且防止一次性 重新映射数据过大导致线程阻塞，采用 渐进式 rehash。巧妙的将一次性拷贝分摊到多次请求过程后总，避免阻塞。
  3. Redis 使用的是非阻塞 IO：IO 多路复用，使用了单线程来轮询描述符，将数据库的开、关、读、写都转换成了事件，Redis 采用自己实现的事件分离器，效率比较高。
  4. 采用单线程模型，保证了每个操作的原子性，也减少了线程的上下文切换和竞争。
  5. Redis 全程使用 hash 结构，读取速度快，还有一些特殊的数据结构，对数据存储进行了优化，如压缩表，对短数据进行压缩存储，再如，跳表，使用有序的数据结构加快读取的速度。

## Redis为什么早期选择单线程

因为Redis是基于内存的操作，CPU成为Redis的瓶颈的情况很少见，Redis的瓶颈最有可能是内存的大小或者网络限制。如果想要最大程度利用CPU，可以在一台机器上启动多个Redis实例。

**PS：**网上有这样的回答，吐槽官方的解释有些敷衍，其实就是历史原因，开发者嫌多线程麻烦，后来这个CPU的利用问题就被抛给了使用者。

同时FAQ里还提到了， Redis 4.0 之后开始变成多线程，除了主线程外，它也有后台线程在处理一些较为缓慢的操作，例如清理脏数据、无用连接的释放、大 Key 的删除等等。

## Redis6.0使用多线程是怎么回事

Redis不是说用单线程的吗？怎么6.0成了多线程的？Redis6.0的多线程是用多线程来处理数据的**读写和协议解析**，但是Redis**执行命令**还是单线程的。这样做的⽬的是因为Redis的性能瓶颈在于⽹络IO⽽⾮CPU，使⽤多线程能提升IO读写的效率，从⽽整体提⾼Redis的性能。

<img title="" src="https://img-blog.csdnimg.cn/img_convert/ca0ba12d865ead001e4224e015523a4e.png" alt="" data-align="inline">

# 持久化

## Redis持久化⽅式有哪些

1. 快照持久化
   
   ​        根据配置，快照将被写入指定的文件里面，如果在新的快照文件创建完毕之前，Redis、系统或者硬件这三者中的任意一个崩溃了，那么redis将丢失最近一次创建快照之后写入的所有数据。
   
   ​        举例来说，A时间创建了一个快照，并且创建成功，B时间又开始创建快照，并且需要等到C时间才能创建完毕，C-B时间之间有35个键更新，那么当BC之间系统发生崩溃时，redis将丢失A时间以后的所有数据，C时间之后发生崩溃，会丢失35个键值得更新数据。
   
   - 客户端使用bgsave命令创建快照，redis fork一个子进程，子进程写快照，父进程处理请求
   - 客户端使用save命令创建快照，redis会在快照创建完毕之前不响应任何命令，这用在内存不够执行bgsave或者即使等待持久化完毕再执行操作也无所谓的情况下使用。
   - 如果用户设置了save配置选项，比如save 60 10000，那么从redis最近一次创建快照之后算起，当60秒之内有10000次写入这个条件满足之后，redis会自动触发bgsave命令。如果有多个配置，那么任意一个满足就会触发。
   - 如果收到shutdown命令关闭服务器请求时，或者接收到标准term信号时，会执行一个save命令，阻塞所有客户端，不再执行客户端发送的任何命令，并在save命令执行完毕之后关闭服务器。
   - 当一个redis服务器连接另一个redis服务器，并向对方发送sync命令来开始一次复制操作的时候，如果主服务器没有执行bgsave操作，或者1主服务器并非刚刚执行完bgsave操作，那么主服务器就会执行bgsave命令。

2. AOF持久化
   
   - always ，每个redis命令都要同步写入硬盘，数据丢失最少，但是需要对硬盘进行大量写入，会受到硬盘性能的限制，旋转式每秒只能处理大约200个命令，固态硬盘每秒也只能处理几万个命令，但是固态硬盘可能会产生严重的写入放大问题，降低固态硬盘的寿命。
   - everysec，每秒1次，与不使用持久化特性时的性能相差无几，即使崩溃最多也就丢失1秒之内的数据。
   - no，让操作系统决定何时进行同步，可能会丢失不定量的数据。

## RDB 和 AOF 各自有什么优缺点

- RDB（Redis Database Backup file）
  
  > 优点：
  > 
  > 1. 只有一个紧凑的二进制文件 `dump.rdb`，非常适合备份、全量复制的场景。
  > 2. **容灾性好**，可以把RDB文件拷贝道远程机器或者文件系统张，用于容灾恢复。
  > 3. **恢复速度快**，RDB恢复数据的速度远远快于AOF的方式
  > 
  > 缺点：
  > 
  > 1. **实时性低**，RDB 是间隔一段时间进行持久化，没法做到实时持久化/秒级持久化。如果在这一间隔事件发生故障，数据会丢失。
  > 2. **存在兼容问题**，Redis演进过程存在多个格式的RDB版本，存在老版本Redis无法兼容新版本RDB的问题。

- AOF（Append Only File）
  
  > 优点
  > 
  > 1. **实时性好**，aof 持久化可以配置 `appendfsync` 属性，有 `always`，每进行一次命令操作就记录到 aof 文件中一次。
  > 2. 通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。
  > 
  > 缺点
  > 
  > 1. AOF 文件比 RDB **文件大**，且 **恢复速度慢**。
  > 2. **数据集大** 的时候，比 RDB **启动效率低**。

## RDB和AOF如何选择

- 一般来说， 如果想达到足以媲美数据库的 **数据安全性**，应该 **同时使用两种持久化功能**。在这种情况下，当 Redis 重启的时候会优先载入 AOF 文件来恢复原始的数据，因为在通常情况下 AOF 文件保存的数据集要比 RDB 文件保存的数据集要完整。
- 如果 **可以接受数分钟以内的数据丢失**，那么可以 **只使用 RDB 持久化**。
- 有很多用户都只使用 AOF 持久化，但并不推荐这种方式，因为定时生成 RDB 快照（snapshot）非常便于进行数据备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快，除此之外，使用 RDB 还可以避免 AOF 程序的 bug。
- 如果只需要数据在服务器运行的时候存在，也可以不使用任何持久化方式。

## Redis的数据恢复

当Redis发生了故障，可以从RDB或者AOF中恢复数据。恢复的过程也很简单，把RDB或者AOF文件拷贝到Redis的数据目录下，如果使用AOF恢复，配置文件开启AOF，然后启动redis-server即可。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**Redis** 启动时加载数据的流程：

1. AOF持久化开启且存在AOF文件时，优先加载AOF文件。
2. AOF关闭或者AOF文件不存在时，加载RDB文件。
3. 加载AOF/RDB文件成功后，Redis启动成功。
4. AOF/RDB文件存在错误时，Redis启动失败并打印错误信息。

## Redis 4.0 的混合持久化

重启 Redis 时，我们很少使用 `RDB` 来恢复内存状态，因为会丢失大量数据。我们通常使用 AOF 日志重放，但是重放 AOF 日志性能相对 `RDB` 来说要慢很多，这样在 Redis 实例很大的情况下，启动需要花费很长的时间。

**Redis 4.0** 为了解决这个问题，带来了一个新的持久化选项——**混合持久化**。将 `rdb` 文件的内容和增量的 AOF 日志文件存在一起。这里的 AOF 日志不再是全量的日志，而是 **自持久化开始到持久化结束** 的这段时间发生的增量 AOF 日志，通常这部分 AOF 日志很小：![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

于是在 Redis 重启的时候，可以先加载 `rdb` 的内容，然后再重放增量 AOF 日志就可以完全替代之前的 AOF 全量文件重放，重启效率因此大幅得到提升。

# 高可用

Redis保证高可用主要有三种方式：主从、哨兵、集群。

## 主从复制

**主从复制**，是指将一台 Redis 服务器的数据，复制到其他的 Redis 服务器。前者称为 **主节点(master)**，后者称为 **从节点(slave)**。且数据的复制是 **单向** 的，只能由主节点到从节点。Redis 主从复制支持 **主从同步** 和 **从从同步** 两种，后者是 Redis 后续版本新增的功能，以减轻主节点的同步负担。主从复制主要的作用：

- **数据冗余：** 主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- **故障恢复：** 当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复 *(实际上是一种服务的冗余)*。
- **负载均衡：** 在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务 *（即写 Redis 数据时应用连接主节点，读 Redis 数据时应用连接从节点）*，分担服务器负载。尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。
- **高可用基石：** 除了上述作用以外，主从复制还是哨兵和集群能够实施的 **基础**，因此说主从复制是 Redis 高可用的基础。

## Redis主从有几种常见的拓扑结构

Redis的复制拓扑结构可以支持单层或多层复制关系，根据拓扑复杂性可以分为以下三种：一主一从、一主多从、树状主从结构。

- 一主一从结构
  
  一主一从结构是最简单的复制拓扑结构，用于主节点出现宕机时从节点提供故障转移支持

- 一主多从结构
  
  一主多从结构（又称为星形拓扑结构）使得应用端可以利用多个从节点实现读写分离（见图6-5）。对于读占比较大的场景，可以把读命令发送到从节点来分担主节点压力

- 树状主从结构
  
  树状主从结构（又称为树状拓扑结构）使得从节点不但可以复制主节点数据，同时可以作为其他从节点的主节点继续向下层复制。通过引入复制中间层，可以有效降低主节点负载和需要传送给从节点的数据量。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## Redis的主从复制原理

Redis主从复制的工作流程大概可以分为如下几步：![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

1. 保存主节点（master）信息 这一步只是保存主节点信息，保存主节点的ip和port。
2. 主从建立连接 从节点（slave）发现新的主节点后，会尝试和主节点建立网络连接。
3. 发送ping命令 连接建立成功后从节点发送ping请求进行首次通信，主要是检测主从之间网络套接字是否可用、主节点当前是否可接受处理命令。
4. 权限验证 如果主节点要求密码验证，从节点必须正确的密码才能通过验证。
5. 同步数据集 主从复制连接正常通信后，主节点会把持有的数据全部发送给从节点。
6. 命令持续复制 接下来主节点会持续地把写命令发送给从节点，保证主从数据一致性。

## 说说主从数据同步的方式

Redis在2.8及以上版本使用psync命令完成主从数据同步，同步过程分为：全量复制和部分复制。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)主从数据同步方式

**全量复制**一般用于初次复制场景，Redis早期支持的复制功能只有全量复制，它会把主节点全部数据一次性发送给从节点，当数据量较大时，会对主从节点和网络造成很大的开销。

全量复制的完整运行流程如下：![图片](https://img-blog.csdnimg.cn/img_convert/17d38dfdfef11c2046afd2094b0ffd31.png)

1. 发送psync命令进行数据同步，由于是第一次进行复制，从节点没有复制偏移量和主节点的运行ID，所以发送psync-1。
2. 主节点根据psync-1解析出当前为全量复制，回复+FULLRESYNC响应。
3. 从节点接收主节点的响应数据保存运行ID和偏移量offset
4. 主节点执行bgsave保存RDB文件到本地
5. 主节点发送RDB文件给从节点，从节点把接收的RDB文件保存在本地并直接作为从节点的数据文件
6. 对于从节点开始接收RDB快照到接收完成期间，主节点仍然响应读写命令，因此主节点会把这期间写命令数据保存在复制客户端缓冲区内，当从节点加载完RDB文件后，主节点再把缓冲区内的数据发送给从节点，保证主从之间数据一致性。
7. 从节点接收完主节点传送来的全部数据后会清空自身旧数据
8. 从节点清空数据后开始加载RDB文件
9. 从节点成功加载完RDB后，如果当前节点开启了AOF持久化功能， 它会立刻做bgrewriteaof操作，为了保证全量复制后AOF持久化文件立刻可用。

**部分复制**部分复制主要是Redis针对全量复制的过高开销做出的一种优化措施， 使用psync{runId}{offset}命令实现。当从节点（slave）正在复制主节点 （master）时，如果出现网络闪断或者命令丢失等异常情况时，从节点会向 主节点要求补发丢失的命令数据，如果主节点的复制积压缓冲区内存在这部分数据则直接发送给从节点，这样就可以保持主从节点复制的一致性。![图片](https://img-blog.csdnimg.cn/img_convert/82db731510777e0864eed4c2b099615c.png)

1. 当主从节点之间网络出现中断时，如果超过repl-timeout时间，主节点会认为从节点故障并中断复制连接
2. 主从连接中断期间主节点依然响应命令，但因复制连接中断命令无法发送给从节点，不过主节点内部存在的复制积压缓冲区，依然可以保存最近一段时间的写命令数据，默认最大缓存1MB。
3. 当主从节点网络恢复后，从节点会再次连上主节点
4. 当主从连接恢复后，由于从节点之前保存了自身已复制的偏移量和主节点的运行ID。因此会把它们当作psync参数发送给主节点，要求进行部分复制操作。
5. 主节点接到psync命令后首先核对参数runId是否与自身一致，如果一 致，说明之前复制的是当前主节点；之后根据参数offset在自身复制积压缓冲区查找，如果偏移量之后的数据存在缓冲区中，则对从节点发送+CONTINUE响应，表示可以进行部分复制。
6. 主节点根据偏移量把复制积压缓冲区里的数据发送给从节点，保证主从复制进入正常状态。

## 主从复制存在哪些问题

主从复制虽好，但也存在一些问题：

- 一旦主节点出现故障，需要手动将一个从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令其他从节点去复制新的主节点，整个过程都需要人工干预。
- 主节点的写能力受到单机的限制。
- 主节点的存储能力受到单机的限制。

第一个问题是Redis的高可用问题，第二、三个问题属于Redis的分布式问题。

## Redis Sentinel（哨兵）

主从复制存在一个问题，没法完成自动故障转移。所以我们需要一个方案来完成自动故障转移，它就是Redis Sentinel（哨兵）。

![图片](https://img-blog.csdnimg.cn/img_convert/cefe4d98f3058c3faa3e3f6720c9d828.png)

Redis Sentinel ，它由两部分组成，哨兵节点和数据节点：

- **哨兵节点：** 哨兵系统由一个或多个哨兵节点组成，哨兵节点是特殊的 Redis 节点，不存储数据，对数据节点进行监控。
- **数据节点：** 主节点和从节点都是数据节点；

在复制的基础上，哨兵实现了 **自动化的故障恢复** 功能，下面是官方对于哨兵功能的描述：

- **监控（Monitoring）：** 哨兵会不断地检查主节点和从节点是否运作正常。
- **自动故障转移（Automatic failover）：** 当 **主节点** 不能正常工作时，哨兵会开始 **自动故障转移操作**，它会将失效主节点的其中一个 **从节点升级为新的主节点**，并让其他从节点改为复制新的主节点。
- **配置提供者（Configuration provider）：** 客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。
- **通知（Notification）：** 哨兵可以将故障转移的结果发送给客户端。

其中，监控和自动故障转移功能，使得哨兵可以及时发现主节点故障并完成转移。而配置提供者和通知功能，则需要在与客户端的交互中才能体现。

## Redis Sentinel（哨兵）实现原理

哨兵模式是通过哨兵节点完成对数据节点的监控、下线、故障转移。

- **定时监控**![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)Redis Sentinel通过三个定时监控任务完成对各个节点发现和监控：

- 1. 每隔10秒，每个Sentinel节点会向主节点和从节点发送info命令获取最新的拓扑结构
  2. 每隔2秒，每个Sentinel节点会向Redis数据节点的__sentinel__：hello 频道上发送该Sentinel节点对于主节点的判断以及当前Sentinel节点的信息
  3. 每隔1秒，每个Sentinel节点会向主节点、从节点、其余Sentinel节点发送一条ping命令做一次心跳检测，来确认这些节点当前是否可达

- **主观下线和客观下线**主观下线就是哨兵节点认为某个节点有问题，客观下线就是超过一定数量的哨兵节点认为主节点有问题。
  
  ![图片](https://img-blog.csdnimg.cn/img_convert/90b7c298af0f5a000e167533becb87ba.png)
1. 主观下线 每个Sentinel节点会每隔1秒对主节点、从节点、其他Sentinel节点发送ping命令做心跳检测，当这些节点超过 down-after-milliseconds没有进行有效回复，Sentinel节点就会对该节点做失败判定，这个行为叫做主观下线。
2. 客观下线 当Sentinel主观下线的节点是主节点时，该Sentinel节点会通过sentinel is- master-down-by-addr命令向其他Sentinel节点询问对主节点的判断，当超过 <quorum>个数，Sentinel节点认为主节点确实有问题，这时该Sentinel节点会做出客观下线的决定
- **领导者Sentinel节点选举**Sentinel节点之间会做一个领导者选举的工作，选出一个Sentinel节点作为领导者进行故障转移的工作。Redis使用了Raft算法实现领导者选举。

- **故障转移**
  
  领导者选举出的Sentinel节点负责故障转移，过程如下：![图片](https://mmbiz.qpic.cn/mmbiz_png/PMZOEonJxWf5IyvQkjc4vibibgKwWma1iatgaXAicWYwiaVdF0ljoRyDtcsAVY2T6ejosguCF7doMfMJKEicpib4PPVSQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

- 1. 在从节点列表中选出一个节点作为新的主节点，这一步是相对复杂一些的一步
  2. Sentinel领导者节点会对第一步选出来的从节点执行slave of no one命令让其成为主节点
  3. Sentinel领导者节点会向剩余的从节点发送命令，让它们成为新主节点的从节点
  4. Sentinel节点集合会将原来的主节点更新为从节点，并保持着对其关注，当其恢复后命令它去复制新的主节点

## 领导者Sentinel节点选举

Redis使用了Raft算法实 现领导者选举，大致流程如下：

![图片](https://img-blog.csdnimg.cn/img_convert/058865800e125140d1269aa918c310a0.png)

1. 每个在线的Sentinel节点都有资格成为领导者，当它确认主节点主观 下线时候，会向其他Sentinel节点发送sentinel is-master-down-by-addr命令， 要求将自己设置为领导者。
2. 收到命令的Sentinel节点，如果没有同意过其他Sentinel节点的sentinel is-master-down-by-addr命令，将同意该请求，否则拒绝。
3. 如果该Sentinel节点发现自己的票数已经大于等于max（quorum， num（sentinels）/2+1），那么它将成为领导者。
4. 如果此过程没有选举出领导者，将进入下一次选举。

## 新的主节点怎样被挑选出来

选出新的主节点，大概分为这么几步：

![图片](https://img-blog.csdnimg.cn/img_convert/2b37fc086759dc94b3c659085cf1c5dd.png)

1. 过滤：“不健康”（主观下线、断线）、5秒内没有回复过Sentinel节 点ping响应、与主节点失联超过down-after-milliseconds*10秒。
2. 选择slave-priority（从节点优先级）最高的从节点列表，如果存在则返回，不存在则继续。
3. 选择复制偏移量最大的从节点（复制的最完整），如果存在则返 回，不存在则继续。
4. 选择runid最小的从节点。

## Redis 集群

前面说到了主从存在高可用和分布式的问题，哨兵解决了高可用的问题，而集群就是终极方案，一举解决高可用和分布式问题。

![图片](https://img-blog.csdnimg.cn/img_convert/4494b073838b3616f621359a9a87d102.png)

1. **数据分区：** 数据分区 *(或称数据分片)* 是集群最核心的功能。集群将数据分散到多个节点，一方面 突破了 Redis 单机内存大小的限制，**存储容量大大增加**；**另一方面** 每个主节点都可以对外提供读服务和写服务，**大大提高了集群的响应能力**。
2. **高可用：** 集群支持主从复制和主节点的 **自动故障转移** *（与哨兵类似）*，当任一节点发生故障时，集群仍然可以对外提供服务。

## 集群中数据如何分区

分布式的存储中，要把数据集按照分区规则映射到多个节点，常见的数据分区规则三种：

#### 方案一：节点取余分区

节点取余分区，非常好理解，使用特定的数据，比如Redis的键，或者用户ID之类，对响应的hash值取余：hash（key）%N，来确定数据映射到哪一个节点上。

不过该方案最大的问题是，当节点数量变化时，如扩容或收缩节点，数据节点映射关 系需要重新计算，会导致数据的重新迁移。

![图片](https://img-blog.csdnimg.cn/img_convert/6fe5ce5d0a7e79e560de9712e58c0712.png)

#### 方案二：一致性哈希分区

将整个 Hash 值空间组织成一个虚拟的圆环，然后将缓存节点的 IP 地址或者主机名做 Hash 取值后，放置在这个圆环上。当我们需要确定某一个 Key 需 要存取到哪个节点上的时候，先对这个 Key 做同样的 Hash 取值，确定在环上的位置，然后按照顺时针方向在环上“行走”，遇到的第一个缓存节点就是要访问的节点。

比如说下面 这张图里面，Key 1 和 Key 2 会落入到 Node 1 中，Key 3、Key 4 会落入到 Node 2 中，Key 5 落入到 Node 3 中，Key 6 落入到 Node 4 中。

![图片](https://img-blog.csdnimg.cn/img_convert/b6064b065716cc205b91587bd908c5d3.png)

这种方式相比节点取余最大的好处在于加入和删除节点只影响哈希环中 相邻的节点，对其他节点无影响。

但它还是存在问题：

- 缓存节点在圆环上分布不平均，会造成部分缓存节点的压力较大
- 当某个节点故障时，这个节点所要承担的所有访问都会被顺移到另一个节点上，会对后面这个节点造成力。

#### 方案三：虚拟槽分区

这个方案一致性哈希分区的基础上，引入了 **虚拟节点** 的概念。Redis 集群使用的便是该方案，其中的虚拟节点称为 **槽（slot）**。槽是介于数据和实际节点之间的虚拟概念，每个实际节点包含一定数量的槽，每个槽包含哈希值在一定范围内的数据。

![图片](https://img-blog.csdnimg.cn/img_convert/271fea31e638dbb4773e008b06229a26.png)

在使用了槽的一致性哈希分区中，槽是数据管理和迁移的基本单位。槽解耦了数据和实际节点 之间的关系，增加或删除节点对系统的影响很小。仍以上图为例，系统中有 4个实际节点，假设为其分配 `16` 个槽(0-15)；

- 槽 0-3 位于 node1；4-7 位于 node2；以此类推....

如果此时删除 `node2`，只需要将槽 4-7 重新分配即可，例如槽 4-5 分配给 `node1`，槽 6 分配给 `node3`，槽 7 分配给 `node4`，数据在其他节点的分布仍然较为均衡。

## 能说说Redis集群的原理

Redis集群通过数据分区来实现数据的分布式存储，通过自动故障转移实现高可用。

#### 集群创建

数据分区是在集群创建的时候完成的。

![图片](https://img-blog.csdnimg.cn/img_convert/7f3f9b46fce0c0b478e22a0de75da1b9.png)

**设置节点**Redis集群一般由多个节点组成，节点数量至少为6个才能保证组成完整高可用的集群。每个节点需要开启配置cluster-enabled yes，让Redis运行在集群模式下。

![图片](https://img-blog.csdnimg.cn/img_convert/1aaa2bd4efa8acbc14c4e84e9981fa52.png)

**节点握手**节点握手是指一批运行在集群模式下的节点通过Gossip协议彼此通信， 达到感知对方的过程。节点握手是集群彼此通信的第一步，由客户端发起命 令：cluster meet{ip}{port}。完成节点握手之后，一个个的Redis节点就组成了一个多节点的集群。

**分配槽（slot）**Redis集群把所有的数据映射到16384个槽中。每个节点对应若干个槽，只有当节点分配了槽，才能响应和这些槽关联的键命令。通过 cluster addslots命令为节点分配槽。

![](https://img-blog.csdnimg.cn/img_convert/55fcc0e3cc18f96c63157c09f921aacf.png)

#### 故障转移

Redis集群的故障转移和哨兵的故障转移类似，但是Redis集群中所有的节点都要承担状态维护的任务。

**故障发现**Redis集群内节点通过ping/pong消息实现节点通信，集群中每个节点都会定期向其他节点发送ping消息，接收节点回复pong 消息作为响应。如果在cluster-node-timeout时间内通信一直失败，则发送节 点会认为接收节点存在故障，把接收节点标记为主观下线（pfail）状态。

![图片](https://img-blog.csdnimg.cn/img_convert/0b12fa9d40e7a53ee4033e1499860dfe.png)

当某个节点判断另一个节点主观下线后，相应的节点状态会跟随消息在集群内传播。通过Gossip消息传播，集群内节点不断收集到故障节点的下线报告。当 半数以上持有槽的主节点都标记某个节点是主观下线时。触发客观下线流程。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**故障恢复**

故障节点变为客观下线后，如果下线节点是持有槽的主节点则需要在它 的从节点中选出一个替换它，从而保证集群的高可用。

![图片](https://img-blog.csdnimg.cn/img_convert/bd4c4ad34c9fc2b1d8da5531c1c44ef5.png)

1. 资格检查 每个从节点都要检查最后与主节点断线时间，判断是否有资格替换故障 的主节点。

2. 准备选举时间 当从节点符合故障转移资格后，更新触发故障选举的时间，只有到达该 时间后才能执行后续流程。

3. 发起选举 当从节点定时任务检测到达故障选举时间（failover_auth_time）到达后，发起选举流程。

4. 选举投票 持有槽的主节点处理故障选举消息。投票过程其实是一个领导者选举的过程，如集群内有N个持有槽的主节 点代表有N张选票。由于在每个配置纪元内持有槽的主节点只能投票给一个 从节点，因此只能有一个从节点获得N/2+1的选票，保证能够找出唯一的从节点。
   
   ![图片](https://img-blog.csdnimg.cn/img_convert/3b55a4ff16e3cb7a37a17967ec4b61f1.png)

5. 替换主节点 当从节点收集到足够的选票之后，触发替换主节点操作。

> **部署Redis集群至少需要几个物理节点？**

在投票选举的环节，故障主节点也算在投票数内，假设集群内节点规模是3主3从，其中有2 个主节点部署在一台机器上，当这台机器宕机时，由于从节点无法收集到 3/2+1个主节点选票将导致故障转移失败。这个问题也适用于故障发现环节。因此部署集群时所有主节点最少需要部署在3台物理机上才能避免单点问题。

## 集群的伸缩

Redis集群提供了灵活的节点扩容和收缩方案，可以在不影响集群对外服务的情况下，为集群添加节点进行扩容也可以下线部分节点进行缩容。

![图片](https://img-blog.csdnimg.cn/img_convert/18fe6ac1f831f5e307c5c4bcad4072e5.png)

其实，集群扩容和缩容的关键点，就在于槽和节点的对应关系，扩容和缩容就是将一部分`槽`和`数据`迁移给新节点。例如下面一个集群，每个节点对应若干个槽，每个槽对应一定的数据，如果希望加入1个节点希望实现集群扩容时，需要通过相关命令把一部分槽和内容迁移给新节点。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)缩容也是类似，先把槽和数据迁移到其它节点，再把对应的节点下线。

![](https://img-blog.csdnimg.cn/img_convert/93dbaf8e90f457d5e4d410b58a6fef21.png)

# 缓存设计

## 缓存击穿、缓存穿透、缓存雪崩

> 1. 如果在某个key（通常是热点key）失效的时候，有大量的请求一起过来就是**击穿**
>
>    - 开一个后台线程监控数据，变化时就缓存进去，这种解决方案只适用于不要求数据严格一致性的情况，因为当后台线程在构建缓存的时候，其他的线程很有可能也在读取数据，这样就会访问到旧数据了。
>    - 当 key 失效的时候，让一个线程读取数据并构建到缓存中，其他线程就先等待，直到缓存构建完后重新读取缓存。当然，采用互斥锁的方案也是有缺陷的，当缓存失效的时候，同一时间只有一个线程读数据库然后回写缓存，其他线程都处于阻塞状态。如果是高并发场景，大量线程阻塞势必会降低吞吐量。这种情况该如何处理呢？我只能说没什么设计是完美的，你又想数据一致，又想保证吞吐量，哪有那么好的事，为了系统能更加健全，必要的时候牺牲下性能也是可以采取的措施，两者之间怎么取舍要根据实际业务场景来决定，万能的技术方案什么的根本不存在。
>
>    ```java
>    @RestController
>    @Slf4j
>    public class testRedis {
>        @Autowired
>        private StringRedisTemplate stringRedisTemplate;
>        private AtomicInteger atomicInteger = new AtomicInteger();
>    
>        @PostConstruct
>        public void init() {
>            //初始化一个热点数据到Redis中，过期时间设置为5秒
>            stringRedisTemplate.opsForValue().set("hotsopt", getExpensiveData(), 5, TimeUnit.SECONDS);
>            //每隔1秒输出一下回源的QPS
>            Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {
>                log.info("DB QPS : {}", atomicInteger.getAndSet(0));
>            }, 0, 1, TimeUnit.SECONDS);
>        }
>    
>        @GetMapping("city")
>        public String wrong() throws InterruptedException {
>            String data = stringRedisTemplate.opsForValue().get("hotsopt");
>            if (StringUtils.isEmpty(data)) {
>                data = getExpensiveData();
>                //重新加入缓存，过期时间还是5秒
>                stringRedisTemplate.opsForValue().set("hotsopt", data, 5, TimeUnit.SECONDS);
>            }
>            return data;
>        }
>    
>        private String getExpensiveData() {
>            atomicInteger.incrementAndGet();
>            return "important data";
>        }
>    }
>    ```
>
> 2. 如果大量的请求访问多个key，刚好key同时失效了就是**雪崩**
>
>    - 热点key永不过期
>    - key设置不同的失效时间
>    - 互斥锁（只有一个读库更新缓存，别的读缓存）
>    - 主备缓存，备缓存有效期长，获取锁失败时读取备份缓存，更新主缓存时更新备用缓存
>    - 缓存预热
>    - 缓存降级，指缓存失效或缓存服务器挂掉的情况下，不去访问数据库，直接返回默认数据或访问服务的内存数据
>
>    ```java
>    /**
>     * @author wyj
>     * 模拟缓存雪崩，从结果可以看出每过大约30秒就会产生一次回源
>     */
>    
>    @RestController
>    @Slf4j
>    public class testRedis {
>    
>        @Autowired
>        private StringRedisTemplate stringRedisTemplate;
>        private AtomicInteger atomicInteger = new AtomicInteger();
>    
>        @PostConstruct
>        public void wrongInit() {
>            //初始化1000个城市数据到Redis，所有缓存数据有效期30秒
>            IntStream.rangeClosed(1, 1000).forEach(i -> stringRedisTemplate.opsForValue().set("city" + i, getCityFromDb(i), 30, TimeUnit.SECONDS));
>            log.info("Cache init finished");
>    
>            //每秒一次，输出数据库访问的QPS
>            Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {
>                log.info("DB QPS : {}", atomicInteger.getAndSet(0));
>            }, 0, 1, TimeUnit.SECONDS);
>        }
>    
>        @GetMapping("city")
>        public String city() {
>            //随机查询一个城市
>            int id = ThreadLocalRandom.current().nextInt(1000) + 1;
>            String key = "city" + id;
>            String data = stringRedisTemplate.opsForValue().get(key);
>            if (data == null) {
>                //回源到数据库查询
>                data = getCityFromDb(id);
>                if (!StringUtils.isEmpty(data)){
>                    stringRedisTemplate.opsForValue().set(key, data, 30, TimeUnit.SECONDS);
>                }
>            }
>            return data;
>        }
>    
>        private String getCityFromDb(int cityId) {
>            //模拟查询数据库，查一次增加计数器加一
>            atomicInteger.incrementAndGet();
>            return "citydata" + System.currentTimeMillis();
>        }
>    }
>    /*
>    2022-09-11 17:15:47.957                : DB QPS : 113
>    2022-09-11 17:15:48.957                : DB QPS : 153
>    .....
>    2022-09-11 17:16:19.956                : DB QPS : 205
>    2022-09-11 17:16:20.958                : DB QPS : 186
>    
>    在并发情况下，总共 1000 条数据回源达到了 1002 次，说明有一些条目出现了并发回源
>    */
>    ```
>
> 3. 如果大量的用户请求缓存中不存在的key（甚至数据库也不存在，比如攻击）就是**穿透**
>
>    ```java
>    public class testRedis {
>                
>        @Autowired
>        private StringRedisTemplate stringRedisTemplate;
>        private AtomicInteger atomicInteger = new AtomicInteger();
>                
>        @PostConstruct
>        public void init() {
>            Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {
>                log.info("DB QPS : {}", atomicInteger.getAndSet(0));
>            }, 0, 1, TimeUnit.SECONDS);
>                
>            //bloomFilter = BloomFilter.create(Funnels.integerFunnel(), 10000, 0.01);
>            //IntStream.rangeClosed(1, 10000).forEach(bloomFilter::put);
>        }
>                
>        @GetMapping("city")
>        public String wrong(@RequestParam("id") int id) {
>            String key = "user" + id;
>            String data = stringRedisTemplate.opsForValue().get(key);
>            //无法区分是无效用户还是缓存失效
>            if (StringUtils.isEmpty(data)) {
>                data = getCityFromDb(id);
>                stringRedisTemplate.opsForValue().set(key, data, 30, TimeUnit.SECONDS);
>            }
>            return data;
>        }
>                
>        private String getCityFromDb(int id) {
>            atomicInteger.incrementAndGet();
>            //注意，只有ID介于0（不含）和10000（包含）之间的用户才是有效用户，可以查询到用户信息
>            if (id > 0 && id <= 10000) {
>                return "userdata";
>            }
>            //否则返回空字符串
>            return "";
>        }
>    }
>    ```
>    
>    - 缓存空对象，如果有大量的 key 穿透，缓存空对象会占用宝贵的内存空间。空对象的 key 设置了过期时间，这段时间内可能数据库刚好有了该 key 的数据，从而导致数据不一致的情况。
>    
>    ```java
>                
>    @GetMapping("right")
>    public String right(@RequestParam("id") int id) {
>        String key = "user" + id;
>        String data = stringRedisTemplate.opsForValue().get(key);
>        if (StringUtils.isEmpty(data)) {
>            data = getCityFromDb(id);
>            //校验从数据库返回的数据是否有效
>            if (!StringUtils.isEmpty(data)) {
>                stringRedisTemplate.opsForValue().set(key, data, 30, TimeUnit.SECONDS);
>            }
>            else {
>                //如果无效，直接在缓存中设置一个NODATA，这样下次查询时即使是无效用户还是可以命中缓存
>                stringRedisTemplate.opsForValue().set(key, "NODATA", 30, TimeUnit.SECONDS);
>            }
>        }
>        return data;
>    }
>    //这种方式可能会把大量无效的数据加入缓存中，如果担心大量无效数据占满缓存的话还可以考虑方案二，即使用布隆过滤器做前置过滤。
>    ```
>    
>    - 布隆过滤器， m 比特的位数组（bit array）与 k 个哈希函数（hash function）组成的数据结构，优点是节省空间、时间复杂度低，缺点准确率有误，不能删除元素
>    
>    ```java
>    @RestController
>    @Slf4j
>    public class testRedis {
>                
>        @Autowired
>        private StringRedisTemplate stringRedisTemplate;
>        private AtomicInteger atomicInteger = new AtomicInteger();
>        private BloomFilter<Integer> bloomFilter;
>                
>        @PostConstruct
>        public void init() {
>            Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(() -> {
>                log.info("DB QPS : {}", atomicInteger.getAndSet(0));
>            }, 0, 1, TimeUnit.SECONDS);
>                
>            bloomFilter = BloomFilter.create(Funnels.integerFunnel(), 10000, 0.01);
>            IntStream.rangeClosed(1, 10000).forEach(bloomFilter::put);
>        }
>                
>        @GetMapping("city")
>        public String right(@RequestParam("id") int id) {
>            String data = "";
>            if (bloomFilter.mightContain(id)) {
>                String key = "user" + id;
>                data = stringRedisTemplate.opsForValue().get(key);
>                if (StringUtils.isEmpty(data)) {
>                    data = getCityFromDb(id);
>                    stringRedisTemplate.opsForValue().set(key, data, 30, TimeUnit.SECONDS);
>                }
>            }
>            return data;
>        }
>                
>        private String getCityFromDb(int id) {
>            atomicInteger.incrementAndGet();
>            //注意，只有ID介于0（不含）和10000（包含）之间的用户才是有效用户，可以查询到用户信息
>            if (id > 0 && id <= 10000) {
>                return "userdata";
>            }
>            //否则返回空字符串
>            return "";
>        }
>    }
>    ```

## 布隆过滤器

布隆过滤器，它是一个连续的数据结构，每个存储位存储都是一个`bit`，即`0`或者`1`, 来标识数据是否存在。存储数据的时时候，使用K个不同的哈希函数将这个变量映射为bit列表的的K个点，把它们置为1。

我们判断缓存key是否存在，同样，K个哈希函数，映射到bit列表上的K个点，判断是不是1：

- 如果全不是1，那么key不存在；
- 如果都是1，也只是表示key可能存在。

布隆过滤器也有一些缺点：

1. 它在判断元素是否在集合中时是有一定错误几率，因为哈希算法有一定的碰撞的概率。
2. 不支持删除元素。

## 如何保证缓存和数据库数据的⼀致性

[参考](https://mp.weixin.qq.com/s/OiElz_morUKteHnTuhcZUg)

根据CAP理论，在保证可用性和分区容错性的前提下，无法保证一致性，所以缓存和数据库的绝对一致是不可能实现的，只能尽可能保存缓存和数据库的最终一致性。

- 删除缓存而不是更新缓存
  
  > 当一个线程对缓存的key进行写操作的时候，如果其它线程进来读数据库的时候，读到的就是脏数据，产生了数据不一致问题。相比较而言，删除缓存的速度比更新缓存的速度快很多，所用时间相对也少很多，读脏数据的概率也小很多。

- 先更数据，后删缓存
  
  > 先更数据库还是先删缓存？这是一个问题
  > 
  > 更新数据，耗时可能在删除缓存的百倍以上。在缓存中不存在对应的key，数据库又没有完成更新的时候，如果有线程进来读取数据，并写入到缓存，那么在更新成功之后，这个key就是一个脏数据。
  > 
  > 毫无疑问，先删缓存，再更数据库，缓存中key不存在的时间的时间更长，有更大的概率会产生脏数据。
  > 
  > 目前最流行的缓存读写策略cache-aside-pattern就是采用先更数据库，再删缓存的方式。

- 缓存不一致处理
  
  > 如果不是并发特别高，对缓存依赖性很强，其实一定程序的不一致是可以接受的。
  > 
  > 但是如果对一致性要求比较高，那就得想办法保证缓存和数据库中数据一致。
  > 
  > 缓存和数据库数据不一致常见的两种原因：
  > 
  > - 缓存key删除失败
  > - 并发导致写入了脏数据
  > 
  > 可以引入消息队列，把要删除的key或者删除失败的key丢尽消息队列，利用消息队列的重试机制，重试删除对应的key。缺点是对业务代码有一定的侵入性。
  > 
  > 可以用一个服务（比如阿里的 canal）去监听数据库的binlog，获取需要操作的数据。然后用一个公共的服务获取订阅程序传来的信息，进行缓存删除操作。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)这种方式降低了对业务的侵入，但其实整个系统的复杂度是提升的，适合基建完善的大厂。
  > 
  > 还有一种情况，是在缓存不存在的时候，写入了脏数据，这种情况在先删缓存，再更数据库的缓存更新策略下发生的比较多，解决方案是延时双删。简单说，就是在第一次删除缓存之后，过了一段时间之后，再次删除缓存。
  > 
  > 还有一个朴素但是有用的办法，给缓存设置一个合理的过期时间，即使发生了缓存数据不一致的问题，它也不会永远不一致下去，缓存过期的时候，自然又会恢复一致。

## 如何保证本地缓存和分布式缓存的一致

**PS:这道题面试很少问，但实际工作中很常见。**

在日常的开发中，我们常常采用两级缓存：本地缓存+分布式缓存。

所谓本地缓存，就是对应服务器的内存缓存，比如Caffeine，分布式缓存基本就是采用Redis。

那么问题来了，本地缓存和分布式缓存怎么保持数据一致？

Redis缓存，数据库发生更新，直接删除缓存的key即可，因为对于应用系统而言，它是一种中心化的缓存。

但是本地缓存，它是非中心化的，散落在分布式服务的各个节点上，没法通过客户端的请求删除本地缓存的key，所以得想办法通知集群所有节点，删除对应的本地缓存key。

可以采用消息队列的方式：

1. 采用Redis本身的Pub/Sub机制，分布式集群的所有节点订阅删除本地缓存频道，删除Redis缓存的节点，同事发布删除本地缓存消息，订阅者们订阅到消息后，删除对应的本地key。但是Redis的发布订阅不是可靠的，不能保证一定删除成功。
2. 引入专业的消息队列，比如RocketMQ，保证消息的可靠性，但是增加了系统的复杂度。
3. 设置适当的过期时间兜底，本地缓存可以设置相对短一些的过期时间。

## 怎么处理热key

> **什么是热Key？**所谓的热key，就是访问频率比较的key。

比如，热门新闻事件或商品，这类key通常有大流量的访问，对存储这类信息的 Redis来说，是不小的压力。假如Redis集群部署，热key可能会造成整体流量的不均衡，个别节点出现OPS过大的情况，极端情况下热点key甚至会超过 Redis本身能够承受的OPS。

> **怎么处理热key？**

对热key的处理，最关键的是对热点key的监控，可以从这些端来监控热点key:

1. 客户端 客户端其实是距离key“最近”的地方，因为Redis命令就是从客户端发出的，例如在客户端设置全局字典（key和调用次数），每次调用Redis命令时，使用这个字典进行记录。
2. 代理端 像Twemproxy、Codis这些基于代理的Redis分布式架构，所有客户端的请求都是通过代理端完成的，可以在代理端进行收集统计。
3. Redis服务端 使用monitor命令统计热点key是很多开发和运维人员首先想到，monitor命令可以监控到Redis执行的所有命令。

只要监控到了热key，对热key的处理就简单了：

1. 把热key打散到不同的服务器，降低压⼒
2. 加⼊⼆级缓存，提前加载热key数据到内存中，如果redis宕机，⾛内存查询

## 缓存预热怎么做

所谓缓存预热，就是提前把数据库里的数据刷到缓存里，通常有这些方法：

1、直接写个缓存刷新页面或者接口，上线时手动操作

2、数据量不大，可以在项目启动的时候自动进行加载

3、定时任务刷新缓存.

## 大 Key，以及如何在设计上实现大 Key 的拆分

## 热点key重建？问题？解决？

开发的时候一般使用“缓存+过期时间”的策略，既可以加速数据读写，又保证数据的定期更新，这种模式基本能够满足绝大部分需求。

但是有两个问题如果同时出现，可能就会出现比较大的问题：

- 当前key是一个热点key（例如一个热门的娱乐新闻），并发量非常大。
- 重建缓存不能在短时间完成，可能是一个复杂计算，例如复杂的 SQL、多次IO、多个依赖等。在缓存失效的瞬间，有大量线程来重建缓存，造成后端负载加大，甚至可能会让应用崩溃。

> **怎么处理呢？**

要解决这个问题也不是很复杂，解决问题的要点在于：

- 减少重建缓存的次数。
- 数据尽可能一致。
- 较少的潜在危险。

所以一般采用如下方式：

1. 互斥锁（mutex key） 这种方法只允许一个线程重建缓存，其他线程等待重建缓存的线程执行完，重新从缓存获取数据即可。
2. 永远不过期 “永远不过期”包含两层意思：
- 从缓存层面来看，确实没有设置过期时间，所以不会出现热点key过期后产生的问题，也就是“物理”不过期。
- 从功能层面来看，为每个value设置一个逻辑过期时间，当发现超过逻辑过期时间后，会使用单独的线程去构建缓存。

## 无底洞问题解决

> **什么是无底洞问题？**

2010年，Facebook的Memcache节点已经达到了3000个，承载着TB级别的缓存数据。但开发和运维人员发现了一个问题，为了满足业务要求添加了大量新Memcache节点，但是发现性能不但没有好转反而下降了，当时将这 种现象称为缓存的“**无底洞**”现象。

那么为什么会产生这种现象呢?

通常来说添加节点使得Memcache集群性能应该更强了，但事实并非如此。键值数据库由于通常采用哈希函数将 key映射到各个节点上，造成key的分布与业务无关，但是由于数据量和访问量的持续增长，造成需要添加大量节点做水平扩容，导致键值分布到更多的 节点上，所以无论是Memcache还是Redis的分布式，批量操作通常需要从不同节点上获取，相比于单机批量操作只涉及一次网络操作，分布式批量操作会涉及多次网络时间。

> **无底洞问题如何优化呢？**

先分析一下无底洞问题：

- 客户端一次批量操作会涉及多次网络操作，也就意味着批量操作会随着节点的增多，耗时会不断增大。
- 网络连接数变多，对节点的性能也有一定影响。

常见的优化思路如下：

- 命令本身的优化，例如优化操作语句等。
- 减少网络通信次数。
- 降低接入成本，例如客户端使用长连/连接池、NIO等。

# 运维

## Redis报内存不足怎么处理

Redis 内存不足有这么几种处理方式：

- 修改配置文件 redis.conf 的 maxmemory 参数，增加 Redis 可用内存
- 也可以通过命令set maxmemory动态设置内存上限
- 修改内存淘汰策略，及时释放内存空间
- 使用 Redis 集群模式，进行横向扩容。

## Redis的过期数据回收策略有哪些

- 惰性删除
  
  > 惰性删除指的是当我们查询key的时候才对key进⾏检测，如果已经达到过期时间，则删除。显然，他有⼀个缺点就是如果这些过期的key没有被访问，那么他就⼀直⽆法被删除，⽽且⼀直占⽤内存

- 定期删除
  
  > 定期删除指的是Redis每隔⼀段时间对数据库做⼀次检查，删除⾥⾯的过期key。由于不可能对所有key去做轮询来删除，所以Redis会每次随机取⼀些key去做检查和删除

## Redis有哪些内存溢出控制/内存淘汰策略

Redis所用内存达到maxmemory上限时会触发相应的溢出控制策略，Redis支持六种策略：

> 1. noeviction：默认策略，不会删除任何数据，拒绝所有写入操作并返 回客户端错误信息，此 时Redis只响应读操作。
> 2. volatile-lru：根据LRU算法删除设置了超时属性（expire）的键，直 到腾出足够空间为止。如果没有可删除的键对象，回退到noeviction策略。
> 3. allkeys-lru：根据LRU算法删除键，不管数据有没有设置超时属性， 直到腾出足够空间为止。
> 4. allkeys-random：随机删除所有键，直到腾出足够空间为止。
> 5. volatile-random：随机删除过期键，直到腾出足够空间为止。
> 6. volatile-ttl：根据键值对象的ttl属性，删除最近将要过期数据。如果没有，回退到noeviction策略。
> 6. **allkeys-lfu（Redis 4.0 以上），针对所有 Key，优先删除最少使用的 Key；**
> 6. **volatile-lfu（Redis 4.0 以上），针对带有过期时间的 Key，优先删除最少使用的 Key。**

## Redis阻塞怎么解决

Redis发生阻塞，可以从以下几个方面排查：

- **API或数据结构使用不合理**
  
  通常Redis执行命令速度非常快，但是不合理地使用命令，可能会导致执行速度很慢，导致阻塞，对于高并发的场景，应该尽量避免在大对象上执行算法复杂 度超过O（n）的命令。
  
  对慢查询的处理分为两步：

- 1. 发现慢查询：slowlog get{n}命令可以获取最近 的n条慢查询命令；
  2. 发现慢查询后，可以从两个方向去优化慢查询：1）修改为低算法复杂度的命令，如hgetall改为hmget等，禁用keys、sort等命 令 2）调整大对象：缩减大对象数据或把大对象拆分为多个小对象，防止一次命令操作过多的数据。

- **CPU饱和的问题**
  
  单线程的Redis处理命令时只能使用一个CPU。而CPU饱和是指Redis单核CPU使用率跑到接近100%。
  
  针对这种情况，处理步骤一般如下：

  1. 判断当前Redis并发量是否已经达到极限，可以使用统计命令redis-cli-h{ip}-p{port}--stat获取当前 Redis使用情
  2. 如果Redis的请求几万+，那么大概就是Redis的OPS已经到了极限，应该做集群化水品扩展来分摊OPS压力
  3. 如果只有几百几千，那么就得排查命令和内存的使用
  
- **持久化相关的阻塞**
  
  对于开启了持久化功能的Redis节点，需要排查是否是持久化导致的阻塞。

- 1. fork阻塞 fork操作发生在RDB和AOF重写时，Redis主线程调用fork操作产生共享 内存的子进程，由子进程完成持久化文件重写工作。如果fork操作本身耗时过长，必然会导致主线程的阻塞。
  2. AOF刷盘阻塞 当我们开启AOF持久化功能时，文件刷盘的方式一般采用每秒一次，后台线程每秒对AOF文件做fsync操作。当硬盘压力过大时，fsync操作需要等 待，直到写入完成。如果主线程发现距离上一次的fsync成功超过2秒，为了 数据安全性它会阻塞直到后台线程执行fsync操作完成。
  3. HugePage写操作阻塞 对于开启Transparent HugePages的 操作系统，每次写命令引起的复制内存页单位由4K变为2MB，放大了512 倍，会拖慢写操作的执行时间，导致大量写操作慢查询。

## 大key问题

Redis使用过程中，有时候会出现大key的情况， 比如：

- 单个简单的key存储的value很大，size超过10KB
- hash， set，zset，list 中存储过多的元素（以万为单位）

> **大key会造成什么问题呢？**

- 客户端耗时增加，甚至超时
- 对大key进行IO操作时，会严重占用带宽和CPU
- 造成Redis集群中数据倾斜
- 主动删除、被动删等，可能会导致阻塞

> **如何找到大key?**

- bigkeys命令：使用bigkeys命令以遍历的方式分析Redis实例中的所有Key，并返回整体统计信息与每个数据类型中Top1的大Key
- redis-rdb-tools：redis-rdb-tools是由Python写的用来分析Redis的rdb快照文件用的工具，它可以把rdb快照文件生成json文件或者生成报表用来分析Redis的使用详情。

> **如何处理大key?**

- **删除大key**

  - 当Redis版本大于4.0时，可使用UNLINK命令安全地删除大Key，该命令能够以非阻塞的方式，逐步地清理传入的Key。

  - 当Redis版本小于4.0时，避免使用阻塞式命令KEYS，而是建议通过SCAN命令执行增量迭代扫描key，然后判断进行删除。

- **压缩和拆分key**

  - 当vaule是string时，比较难拆分，则使用序列化、压缩算法将key的大小控制在合理范围内，但是序列化和反序列化都会带来更多时间上的消耗。

  - 当value是string，压缩之后仍然是大key，则需要进行拆分，一个大key分为不同的部分，记录每个部分的key，使用multiget等操作实现事务读取。
  - 当value是list/set等集合类型时，根据预估的数据规模来进行分片，不同的元素计算后分到不同的片。

## Redis常见性能问题和解决方案

1. Master 最好不要做任何持久化工作，包括内存快照和 AOF 日志文件，特别是不要启用内存快照做持久化。
2. 如果数据比较关键，某个 Slave 开启 AOF 备份数据，策略为每秒同步一次。
3. 为了主从复制的速度和连接的稳定性，Slave 和 Master 最好在同一个局域网内。
4. 尽量避免在压力较大的主库上增加从库。
5. Master 调用 BGREWRITEAOF 重写 AOF 文件，AOF 在重写的时候会占大量的 CPU 和内存资源，导致服务 load 过高，出现短暂服务暂停现象。
6. 为了 Master 的稳定性，主从复制不要用图状结构，用单向链表结构更稳定，即主从关为：Master<–Slave1<–Slave2<–Slave3…，这样的结构也方便解决单点故障问题，实现 Slave 对 Master 的替换，也即，如果 Master 挂了，可以立马启用 Slave1 做 Master，其他不变。

# Redis应用

## 使用Redis 如何实现异步队列

我们知道redis支持很多种结构的数据，那么如何使用redis作为异步队列使用呢？一般有以下几种方式：

- **使用list作为队列，lpush生产消息，rpop消费消息**

这种方式，消费者死循环rpop从队列中消费消息。但是这样，即使队列里没有消息，也会进行rpop，会导致Redis CPU的消耗。可以通过让消费者休眠的方式的方式来处理，但是这样又会又消息的延迟问题。

- **使用list作为队列，lpush生产消息，brpop消费消息**

brpop是rpop的阻塞版本，list为空的时候，它会一直阻塞，直到list中有值或者超时。这种方式只能实现一对一的消息队列。

- **使用Redis的pub/sub来进行消息的发布/订阅**

发布/订阅模式可以1：N的消息发布/订阅。发布者将消息发布到指定的频道频道（channel），订阅相应频道的客户端都能收到消息。但是这种方式不是可靠的，它不保证订阅者一定能收到消息，也不进行消息的存储。所以，一般的异步队列的实现还是交给专业的消息队列。

## Redis 如何实现延时队列

- **使用zset，利用排序实现**

可以使用 zset这个结构，用设置好的时间戳作为score进行排序，使用 zadd score1 value1 ....命令就可以一直往内存中生产消息。再利用 zrangebysocre 查询符合条件的所有待处理的任务，通过循环执行队列任务即可。

## Redis 支持事务吗？

Redis提供了简单的事务，但它对事务ACID的支持并不完备。

multi命令代表事务开始，exec命令代表事务结束，它们之间的命令是原子顺序执行的：

```shell
127.0.0.1:6379> multi 
OK
127.0.0.1:6379> sadd user:a:follow user:b 
QUEUED 
127.0.0.1:6379> sadd user:b:fans user:a 
QUEUED
127.0.0.1:6379> sismember user:a:follow user:b 
(integer) 0
127.0.0.1:6379> exec 1) (integer) 1
2) (integer) 1
```

Redis事务的原理，是所有的指令在 exec 之前不执行，而是缓存在 服务器的一个事务队列中，服务器一旦收到 exec 指令，才开执行整个事务队列，执行完毕后一次性返回所有指令的运行结果。因为Redis执行命令是单线程的，所以这组命令顺序执行，而且不会被其它线程打断。

**Redis事务的注意点有哪些？**

需要注意的点有：

- Redis 事务是不支持回滚的，不像 MySQL 的事务一样，要么都执行要么都不执行；
- Redis 服务端在执行事务的过程中，不会被其他客户端发送来的命令请求打断。直到事务命令全部执行完毕才会执行其他客户端的命令。

**Redis 事务为什么不支持回滚？**

Redis 的事务不支持回滚。

如果执行的命令有语法错误，Redis 会执行失败，这些问题可以从程序层面捕获并解决。但是如果出现其他问题，则依然会继续执行余下的命令。

这样做的原因是因为回滚需要增加很多工作，而不支持回滚则可以**保持简单、快速的特性**。

## Redis和Lua脚本的使用

Redis的事务功能比较简单，平时的开发中，可以利用Lua脚本来增强Redis的命令。

Lua脚本能给开发人员带来这些好处：

- Lua脚本在Redis中是原子执行的，执行过程中间不会插入其他命令。
- Lua脚本可以帮助开发和运维人员创造出自己定制的命令，并可以将这 些命令常驻在Redis内存中，实现复用的效果。
- Lua脚本可以将多条命令一次性打包，有效地减少网络开销。

比如这一段很（烂）经（大）典（街）的秒杀系统利用lua扣减Redis库存的脚本：

```lua
   -- 库存未预热
   if (redis.call('exists', KEYS[2]) == 1) then
        return -9;
    end;
    -- 秒杀商品库存存在
    if (redis.call('exists', KEYS[1]) == 1) then
        local stock = tonumber(redis.call('get', KEYS[1]));
        local num = tonumber(ARGV[1]);
        -- 剩余库存少于请求数量
        if (stock < num) then
            return -3
        end;
        -- 扣减库存
        if (stock >= num) then
            redis.call('incrby', KEYS[1], 0 - num);
            -- 扣减成功
            return 1
        end;
        return -2;
    end;
    -- 秒杀商品库存不存在
    return -1;
```

## Redis的管道

Redis 提供三种将客户端多条命令打包发送给服务端执行的方式：

Pipelining(管道) 、 Transactions(事务) 和 Lua Scripts(Lua 脚本) 。

**Pipelining**（管道）

Redis 管道是三者之中最简单的，当客户端需要执行多条 redis 命令时，可以通过管道一次性将要执行的多条命令发送给服务端，其作用是为了降低 RTT(Round Trip Time) 对性能的影响，比如我们使用 nc 命令将两条指令发送给 redis 服务端。

Redis 服务端接收到管道发送过来的多条命令后，会一直执命令，并将命令的执行结果进行缓存，直到最后一条命令执行完成，再所有命令的执行结果一次性返回给客户端 。

**Pipelining的优势**

在性能方面， Pipelining 有下面两个优势：

- **节省了RTT**：将多条命令打包一次性发送给服务端，减少了客户端与服务端之间的网络调用次数
- **减少了上下文切换**：当客户端/服务端需要从网络中读写数据时，都会产生一次系统调用，系统调用是非常耗时的操作，其中设计到程序由用户态切换到内核态，再从内核态切换回用户态的过程。当我们执行 10 条 redis 命令的时候，就会发生 10 次用户态到内核态的上下文切换，但如果我们使用 Pipeining 将多条命令打包成一条一次性发送给服务端，就只会产生一次上下文切换。

## Redis实现分布式锁

[参考](https://mp.weixin.qq.com/s/2xhjrkpZNaA6NH4MV0gz6A)

Redis是分布式锁本质上要实现的目标就是在 Redis 里面占一个“茅坑”，当别的进程也要来占时，发现已经有人蹲在那里了，就只好放弃或者稍后再试。

- **V1：setnx命令**

占坑一般是使用 setnx(set if not exists) 指令，只允许被一个客户端占坑。先来先占， 用完了，再调用 del 指令释放茅坑。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
> setnx lock:fighter true
OK
... do something critical ...
> del lock:fighter
(integer) 1
```

但是有个问题，如果逻辑执行到中间出现异常了，可能会导致 del 指令没有被调用，这样就会陷入死锁，锁永远得不到释放。

- **V2:锁超时释放**

所以在拿到锁之后，再给锁加上一个过期时间，比如 5s，这样即使中间出现异常也可以保证 5 秒之后锁会自动释放。!

```
> setnx lock:fighter true
OK
> expire lock:fighter 5
... do something critical ...
> del lock:fighter
(integer) 1
```

但是以上逻辑还有问题。如果在 setnx 和 expire 之间服务器进程突然挂掉了，可能是因为机器掉电或者是被人为杀掉的，就会导致 expire 得不到执行，也会造成死锁。

这种问题的根源就在于 setnx 和 expire 是两条指令而不是原子指令。如果这两条指令可以一起执行就不会出现问题。

- **V3:set指令**

这个问题在Redis 2.8 版本中得到了解决，这个版本加入了 set 指令的扩展参数，使得 setnx 和expire 指令可以一起执行。

```
set lock:fighter3 true ex 5 nx OK ... do something critical ... > del lock:codehole
```

上面这个指令就是 setnx 和 expire 组合在一起的原子指令，这个就算是比较完善的分布式锁了。

当然实际的开发，没人会去自己写分布式锁的命令，因为有专业的轮子——**Redisson**。

## 怎么用redis限制每个人每分钟访问30次

> 1. 以人的id为key，设置过期时间为1分钟，如果有值获取时就判断是否大于3次
> 2. 方案1的问题在于无法检测不均匀的访问，比如第1分钟的前30秒访问了1次，后30秒访问了29次，第2分钟的前30秒访问了29次，最后30秒访问了1次，这样两分钟都没超，但是中间1分钟超了。要精确的保证同一个用户每分钟最多访问30次，需要记录下来用户每次访问的时间。因此对每个用户我们使用一个List列表类型的键来记录他最近30次访问的时间，一旦键中的元素超过30个，就判断最早的元素距离现在的时间是否小于1分钟。如果是表示用户最近1分钟访问次数超过了30次，如果不是就将现在的时间加入到队列中，同时把最早的元素删除。

# 底层结构

 [redis数据结构](https://mp.weixin.qq.com/s/CeuO2n1YZwBXw56WvafctA)

## Redis底层数据结构

Redis有**动态字符串(sds)**、**链表(list)**、**字典(ht)**、**跳跃表(skiplist)**、**整数集合(intset)**、**压缩列表(ziplist)** 等底层数据结构。Redis并没有使用这些数据结构来直接实现键值对数据库，而是基于这些数据结构创建了一个对象系统，来表示所有的key-value。

![图片](https://img-blog.csdnimg.cn/img_convert/dea072135388cbf839beb6fb61c908a4.png)我们常用的数据类型和编码对应的映射关系：

![图片](https://img-blog.csdnimg.cn/img_convert/51279a6bef37783c385a55c45d0fba21.png)

简单看一下底层数据结构，如果对数据结构掌握不错的话，理解这些结构应该不是特别难：

1. **字符串**：redis没有直接使⽤C语⾔传统的字符串表示，⽽是⾃⼰实现的叫做简单动态字符串SDS的抽象类型。
   
   C语⾔的字符串不记录⾃身的⻓度信息，⽽SDS则保存了⻓度信息，这样将获取字符串⻓度的时间由O(N)降低到了O(1)，同时可以避免缓冲区溢出和减少修改字符串⻓度时所需的内存重分配次数。
   
   ![](https://img-blog.csdnimg.cn/3bcc9212255544abbd9801232f0bcb23.png)

2. **链表linkedlist**：redis链表是⼀个双向⽆环链表结构，很多发布订阅、慢查询、监视器功能都是使⽤到了链表来实现，每个链表的节点由⼀个listNode结构来表示，每个节点都有指向前置节点和后置节点的指针，同时表头节点的前置和后置节点都指向NULL。
   
   > Redis 的链表实现优点如下：
   > 
   > - listNode 链表节点的结构里带有 prev 和 next 指针，**获取某个节点的前置节点或后置节点的时间复杂度只需O(1)，而且这两个指针都可以指向 NULL，所以链表是无环链表**；
   > - list 结构因为提供了表头指针 head 和表尾节点 tail，所以**获取链表的表头节点和表尾节点的时间复杂度只需O(1)**；
   > - list 结构因为提供了链表节点数量 len，所以**获取链表中的节点数量的时间复杂度只需O(1)**；
   > - listNode 链表节使用 void* 指针保存节点值，并且可以通过 list 结构的 dup、free、match 函数指针为节点设置该节点类型特定的函数，因此**链表节点可以保存各种不同类型的值**；
   > 
   > 链表的缺陷也是有的：
   > 
   > - 链表每个节点之间的内存都是不连续的，意味着**无法很好利用 CPU 缓存**。能很好利用 CPU 缓存的数据结构就是数组，因为数组的内存是连续的，这样就可以充分利用 CPU 缓存来加速访问。
   > - 还有一点，保存一个链表节点的值都需要一个链表节点结构头的分配，**内存开销较大**。
   > 
   > 因此，Redis 3.0 的 List 对象在数据量比较少的情况下，会采用「压缩列表」作为底层数据结构的实现，它的优势是节省内存空间，并且是内存紧凑型的数据结构。
   > 
   > 不过，压缩列表存在性能问题（具体什么问题，下面会说），所以 Redis 在 3.2 版本设计了新的数据结构 quicklist，并将 List 对象的底层数据结构改由 quicklist 实现。
   > 
   > 然后在  Redis 5.0 设计了新的数据结构 listpack，沿用了压缩列表紧凑型的内存布局，最终在最新的 Redis 版本，将 Hash 对象和 Zset 对象的底层数据结构实现之一的压缩列表，替换成由  listpack 实现。
   
   ![](https://img-blog.csdnimg.cn/ac4c42aba7054591b53a0828e41360f8.png)

3. **字典dict**：⽤于保存键值对的抽象数据结构。Redis使⽤hash表作为底层实现，一个哈希表里可以有多个哈希表节点，而每个哈希表节点就保存了字典里中的一个键值对。每个字典带有两个hash表，供平时使⽤和rehash时使⽤，hash表使⽤链地址法来解决键冲突，被分配到同⼀个索引位置的多个键值对会形成⼀个单向链表，在对hash表进⾏扩容或者缩容的时候，为了服务的可⽤性，rehash的过程不是⼀次性完成的，⽽是渐进式的。![图片](https://img-blog.csdnimg.cn/c9a9169169f44242b5da2b454f26fd34.png)

4. **跳跃表skiplist**：跳跃表是有序集合的底层实现之⼀，Redis中在实现有序集合键和集群节点的内部结构中都是⽤到了跳跃表。Redis跳跃表由zskiplist和zskiplistNode组成，zskiplist⽤于保存跳跃表信息（表头、表尾节点、⻓度等），zskiplistNode⽤于表示表跳跃节点，每个跳跃表节点的层⾼都是1-32的随机数，在同⼀个跳跃表中，多个节点可以包含相同的分值，但是每个节点的成员对象必须是唯⼀的，节点按照分值⼤⼩排序，如果分值相同，则按照成员对象的⼤⼩排序。
   
   ![](https://img-blog.csdnimg.cn/78847a9a10af4e71b927950caf787ecf.png)

5. **整数集合intset**：⽤于保存整数值的集合抽象数据结构，不会出现重复元素，底层实现为数组。
   
   ![图片](https://img-blog.csdnimg.cn/39c6943ea6964d608afab7978aeead3c.png)

6. **压缩列表ziplist**：压缩列表是为节约内存⽽开发的顺序性数据结构，它可以包含任意多个节点，每个节点可以保存⼀个字节数组或者整数值。
   
   > 压缩列表的优点：
   > 
   > - 内存紧凑型的数据结构，占用一块连续的内存空间，不仅可以利用 CPU 缓存，而且会针对不同长度的数据，进行相应编码，这种方法可以有效地节省内存开销。
   > 
   > 压缩列表的缺点：
   > 
   > - 不能保存过多的元素，否则查询效率就会降低；
   > - 新增或修改某个元素时，压缩列表占用的内存空间需要重新分配，甚至可能引发连锁更新的问题。
   
   ![图片](https://img-blog.csdnimg.cn/0403bf5285144e24bf8886a871c1c7d1.png)

## Redis 的 SDS 和 C 中字符串相比有什么优势

C 语言使用了一个长度为 `N+1` 的字符数组来表示长度为 `N` 的字符串，并且字符数组最后一个元素总是 `\0`，这种简单的字符串表示方式 不符合 Redis 对字符串在安全性、效率以及功能方面的要求。

![图片](https://img-blog.csdnimg.cn/97baeed80d4642bfadea3e5ed1cd8ab6.png)

> **C语言的字符串可能有什么问题？**

这样简单的数据结构可能会造成以下一些问题：

- **获取字符串长度复杂度高** ：因为 C 不保存数组的长度，每次都需要遍历一遍整个数组，时间复杂度为O(n)；
- 不能杜绝 **缓冲区溢出/内存泄漏** 的问题 : C字符串不记录自身长度带来的另外一个问题是容易造成缓存区溢出（buffer overflow），例如在字符串拼接的时候，新的
- C 字符串 **只能保存文本数据** → 因为 C 语言中的字符串必须符合某种编码（比如 ASCII），例如中间出现的 `'\0'` 可能会被判定为提前结束的字符串而识别不了；

> **Redis如何解决？优势？**

![图片](https://img-blog.csdnimg.cn/d4ad8d28f90f45d998a2564193bc0fb1.png)

简单来说一下 Redis 如何解决的：

1. **多增加 len 表示当前字符串的长度**：这样就可以直接获取长度了，复杂度 O(1)；
2. **自动扩展空间**：当 SDS 需要对字符串进行修改时，首先借助于 `len` 和 `alloc` 检查空间是否满足修改所需的要求，如果空间不够的话，SDS 会自动扩展空间，避免了像 C 字符串操作中的溢出情况；
3. **有效降低内存分配次数**：C 字符串在涉及增加或者清除操作时会改变底层数组的大小造成重新分配，SDS 使用了 **空间预分配** 和 **惰性空间释放** 机制，简单理解就是每次在扩展时是成倍的多分配的，在缩容是也是先留着并不正式归还给 OS；
4. **二进制安全**：C 语言字符串只能保存 `ascii` 码，对于图片、音频等信息无法保存，SDS 是二进制安全的，写入什么读取就是什么，不做任何过滤和限制；

## 字典是如何实现的？Rehash 了解吗？

字典是 Redis 服务器中出现最为频繁的复合型数据结构。除了 **hash** 结构的数据会用到字典外，整个 Redis 数据库的所有 `key` 和 `value` 也组成了一个 **全局字典**，还有带过期时间的 `key` 也是一个字典。*(存储在 RedisDb 数据结构中)*

> **字典结构是什么样的呢？**

**Redis** 中的字典相当于 Java 中的 **HashMap**，内部实现也差不多类似，采用哈希与运算计算下标位置；通过 **"数组 + 链表" 的链地址法** 来解决哈希冲突，同时这样的结构也吸收了两种不同数据结构的优点。![图片](https://mmbiz.qpic.cn/mmbiz_png/PMZOEonJxWf5IyvQkjc4vibibgKwWma1iatzkdCKqvnlkGKWBx9TT0ac3ibPbscVcXQhpOpwiaxiaSWsRyicBFqMbxYnw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> **字典是怎么扩容的？**

字典结构内部包含 **两个 hashtable**，通常情况下只有一个哈希表 ht[0] 有值，在扩容的时候，把ht[0]里的值rehash到ht[1]，然后进行 **渐进式rehash** ——所谓渐进式rehash，指的是这个rehash的动作并不是一次性、集中式地完成的，而是分多次、渐进式地完成的。

待搬迁结束后，h[1]就取代h[0]存储字典的元素。

## 跳跃表是如何实现的？原理？

PS:跳跃表是比较常问的一种结构。

跳跃表（skiplist）是一种有序数据结构，它通过在每个节点中维持多个指向其它节点的指针，从而达到快速访问节点的目的。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

> **为什么使用跳跃表?**

首先，因为 zset 要支持随机的插入和删除，所以它 **不宜使用数组来实现**，关于排序问题，我们也很容易就想到 **红黑树/ 平衡树** 这样的树形结构，为什么 Redis 不使用这样一些结构呢？

1. **性能考虑：** 在高并发的情况下，树形结构需要执行一些类似于 rebalance 这样的可能涉及整棵树的操作，相对来说跳跃表的变化只涉及局部；
2. **实现考虑：** 在复杂度与红黑树相同的情况下，跳跃表实现起来更简单，看起来也更加直观；

基于以上的一些考虑，Redis 基于 **William Pugh** 的论文做出一些改进后采用了 **跳跃表** 这样的结构。

本质是解决查找问题。

> **跳跃表是怎么实现的？**

跳跃表的节点里有这些元素：

- 层跳跃表节点的level数组可以包含多个元素，每个元素都包含一个指向其它节点的指针，程序可以通过这些层来加快访问其它节点的速度，一般来说，层的数量越多，访问其它节点的速度就越快。
  
  每次创建一个新的跳跃表节点的时候，程序都根据幂次定律，随机生成一个介于1和32之间的值作为level数组的大小，这个大小就是层的“高度”

- **前进指针**每个层都有一个指向表尾的前进指针（level[i].forward属性），用于从表头向表尾方向访问节点。
  
  我们看一下跳跃表从表头到表尾，遍历所有节点的路径：
  
  ![图片](https://img-blog.csdnimg.cn/bc97d05691b4407785a7b2acb81de9d7.png)

- **跨度**层的跨度用于记录两个节点之间的距离。跨度是用来计算排位（rank）的：在查找某个节点的过程中，将沿途访问过的所有层的跨度累计起来，得到的结果就是目标节点在跳跃表中的排位。
  
  例如查找，分值为3.0、成员对象为o3的节点时，沿途经历的层：查找的过程只经过了一个层，并且层的跨度为3，所以目标节点在跳跃表中的排位为3。![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

- **分值和成员**节点的分值（score属性）是一个double类型的浮点数，跳跃表中所有的节点都按分值从小到大来排序。
  
  节点的成员对象（obj属性）是一个指针，它指向一个字符串对象，而字符串对象则保存这一个SDS值。

## 压缩列表

压缩列表是 Redis **为了节约内存** 而使用的一种数据结构，是由一系列特殊编码的连续内存快组成的顺序型数据结构。

一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。

![图片](https://img-blog.csdnimg.cn/e389ae8a9c534bddb7cbe3d44eafce32.png)压缩列表由这么几部分组成：

- **zlbyttes**:记录整个压缩列表占用的内存字节数
- **zltail**:记录压缩列表表尾节点距离压缩列表的起始地址有多少字节
- **zllen**:记录压缩列表包含的节点数量
- **entryX**:列表节点
- **zlend**:用于标记压缩列表的末端

![图片](https://img-blog.csdnimg.cn/09b2ae777dab4f7ea655b87ff167a100.png)

## 快速列表 quicklist

Redis 早期版本存储 list 列表数据结构使用的是压缩列表 ziplist 和普通的双向链表 linkedlist，也就是说当元素少时使用 ziplist，当元素多时用 linkedlist。

但考虑到链表的附加空间相对较高，`prev` 和 `next` 指针就要占去 `16` 个字节（64 位操作系统占用 `8` 个字节），另外每个节点的内存都是单独分配，会家具内存的碎片化，影响内存管理效率。

后来 Redis 新版本（3.2）对列表数据结构进行了改造，使用 `quicklist` 代替了 `ziplist` 和 `linkedlist`，quicklist是综合考虑了时间效率与空间效率引入的新型数据结构。

quicklist由list和ziplist结合而成，它是一个由ziplist充当节点的双向链表。

> 在向 quicklist 添加一个元素的时候，不会像普通的链表那样，直接新建一个链表节点。而是会检查插入位置的压缩列表是否能容纳该元素，如果能容纳就直接保存到 quicklistNode 结构里的压缩列表，如果不能容纳，才会新建一个新的 quicklistNode 结构。
> 
> quicklist 会控制 quicklistNode 结构里的压缩列表的大小或者元素个数，来规避潜在的连锁更新的风险，但是这并没有完全解决连锁更新的问题。

![图片](https://mmbiz.qpic.cn/mmbiz_png/PMZOEonJxWf5IyvQkjc4vibibgKwWma1iaticzSicZ6us2RWa52icIPTPicKDgbibXZvTD7cjvJpOCqXTtibP878hPem0Gg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## listpack

listpack 没有压缩列表中记录前一个节点长度的字段了，listpack 只记录当前节点的长度，当我们向 listpack 加入一个新元素的时候，不会影响其他节点的长度字段的变化，从而避免了压缩列表的连锁更新问题。

- 从左向右遍历时，会先取到链表的头，然后根据每个记录的编码类型、数据和entry-len总长取到下一个的指针

- 从右向左遍历时，先从头中获得总长直接定位到尾部，从右向左，逐个字节地读取当前列表项的 entry-len，得到 entry-len 本身长度，这样一来，我们就可以得到前一项的总长度，而 lpPrev 函数也就可以将指针指向前一项的起始位置了。所以按照这个方法，listpack 就实现了从右向左的查询功能。

  >- 如何判断 entry-len 是否结束了呢？
  >
  >  这就依赖于 entry-len 的编码方式了。entry-len 每个字节的最高位，是用来表示当前字节是否为 entry-len 的最后一个字节，这里存在两种情况，分别是：
  >
  >  最高位为 1，表示 entry-len 还没有结束，当前字节的左边字节仍然表示 entry-len 的内容；
  >  最高位为 0，表示当前字节已经是 entry-len 最后一个字节了。
  >  而 entry-len 每个字节的低 7 位，则记录了实际的长度信息。

## 为什么 Redis 要用跳表来实现有序集合，而不是红黑树

Redis 中的有序集合支持的核心操作主要有下面这几个：

- 插入一个数据；
- 删除一个数据；
- 查找一个数据；
- 按照区间查找数据（比如查找值在[100, 356]之间的数据）;
- 迭代输出有序序列。

其中，插入、删除、查找以及迭代输出有序序列这几个操作，红黑树也可以完成，时间复杂度跟跳表是一样的。但是，按照区间来查找数据这个操作，红黑树的效率没有跳表高。

对于按照区间查找数据这个操作，跳表可以做到 O(logn) 的时间复杂度定位区间的起点，然后在原始链表中顺序往后遍历就可以了。这样做非常高效。

当然，Redis 之所以用跳表来实现有序集合，还有其他原因，比如，跳表更容易代码实现。虽然跳表的实现也不简单，但比起红黑树来说还是好懂、好写多了，而简单就意味着可读性好，不容易出错。还有，跳表更加灵活，它可以通过改变索引构建策略，有效平衡执行效率和内存消耗。

不过，跳表也不能完全替代红黑树。因为红黑树比跳表的出现要早一些，很多编程语言中的 Map 类型都是通过红黑树来实现的。我们做业务开发的时候，直接拿来用就可以了，不用费劲自己去实现一个红黑树，但是跳表并没有一个现成的实现，所以在开发中，如果你想使用跳表，必须要自己实现。

# 其它

## 大量同前缀key搜索

使用 `keys` 指令可以扫出指定模式的 key 列表。但是要注意 keys 指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用 `scan` 指令，`scan` 指令可以无阻塞的提取出指定模式的 `key` 列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用 `keys` 指令长。

## 线上情况

20主，40从，单机10G，总量200G，使用的volatile-lru（在设置了过期时间的key种使用LRU），配置单节点QPS大于6W告警（实际见过200wQPS（单节点10w）没有明显问题），平时高峰期也就40w，2.5亿key（占用60%），集群上又逻辑划分catogory，每个人在自己的category下存取数据。

对于Lua脚本的要求

>lua脚本本身过于灵活，使用过程中存在一定风险（如使用不当可能会导致服务端卡死）。建议优先考虑其他非lua脚本方案实现需求。若确实是需要使用，请先联系集群DBA评估使用的合理性。
>
>- 注意事项：
>- 1. 使用这个lua脚本的每个集群，都需要做一次lua脚本的注册。
>- 2. lua脚本中，仅允许操作一个key（集群模式下，不支持lua操作多key，业务如果必须多key，则要考虑其他方案，lua行不通）。
>
>

## 其它常见面试题总结

[m1](https://mp.weixin.qq.com/s/SMkFzjSBvPvqA_ibFT6xCA)	[Redis 详解 五种数据结构（底层实现原理）、使用场景、淘汰策略、持久化、高可用、缓存的雪崩、击穿、穿透](https://copyfuture.com/blogs-details/20211203091032764J)

## 基于docker安装与启动

```yaml
version: '3'
services:
  redis:
    image: redis
    container_name: redis
    restart: always
    # 优先使用命令行参数，其次是redis.conf中的参数
    # command: redis-server /usr/local/etc/redis/redis.conf  #--requirepass "root123"
    volumes:
      - /Users/wyj/Documents/redis/data:/data
      - /Users/wyj/Documents/redis/data/redis.conf:/usr/local/etc/redis/redis.conf
    ports:
      - "6379:6379"
```
