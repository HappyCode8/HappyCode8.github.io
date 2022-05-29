## 数据结构

1. 字符串：常用于缓存、计数器、分布式锁

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

2. 列表：链表、队列、微博关注人时间轴列表

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

3. 集合：去重、赞、踩、共同好友

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

4. 散列：用户信息、哈希表

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

5. 有序集合：排行榜

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

6. HyperLogLog

   Redis2.8.9新添的结构，用来做基数统计，他只会根据输入元素来计算基数，而不会存储输入元素本身，比如根据数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。

   ```
   pfadd key element [element...]添加指定元素到HyperLogLog中
   pfcount key [key ...] 返回给定HyperLogLog的基数估算值
   pfmerge destkey sourcekey[sourcekey...] 将多个HyperLogLog合并为一个HyperlogLog
   ```

7. 其余不常用的数据结构

   ```
   Geo、Pub/Sub、BloomFilter，RedisSearch，Redis-ML
   ```

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

## 事务

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

```
multi开始一个事务
.......
exec
```

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

## 持久化

1. 快照持久化

   ​		根据配置，快照将被写入指定的文件里面，如果在新的快照文件创建完毕之前，Redis、系统或者硬件这三者中的任意一个崩溃了，那么redis将丢失最近一次创建快照之后写入的所有数据。

   ​		举例来说，A时间创建了一个快照，并且创建成功，B时间又开始创建快照，并且需要等到C时间才能创建完毕，C-B时间之间有35个键更新，那么当BC之间系统发生崩溃时，redis将丢失A时间以后的所有数据，C时间之后发生崩溃，会丢失35个键值得更新数据。

   - 客户端使用bgsave命令创建快照，redis fork一个子进程，子进程写快照，父进程处理请求
   - 客户端使用save命令创建快照，redis会在快照创建完毕之前不响应任何命令，这用在内存不够执行bgsave或者即使等待持久化完毕再执行操作也无所谓的情况下使用。
   - 如果用户设置了save配置选项，比如save 60 10000，那么从redis最近一次创建快照之后算起，当60秒之内有10000次写入这个条件满足之后，redis会自动触发bgsave命令。如果有多个配置，那么任意一个满足就会触发。
   - 如果收到shutdown命令关闭服务器请求时，或者接收到标准term信号时，会执行一个save命令，阻塞所有客户端，不再执行客户端发送的任何命令，并在save命令执行完毕之后关闭服务器。
   - 当一个redis服务器连接另一个redis服务器，并向对方发送sync命令来开始一次复制操作的时候，如果主服务器没有执行bgsave操作，或者1主服务器并非刚刚执行完bgsave操作，那么主服务器就会执行bgsave命令。

2. AOF持久化

   - always ，每个redis命令都要同步写入硬盘，数据丢失最少，但是需要对硬盘进行大量写入，会受到硬盘性能的限制，旋转式每秒只能处理大约200个命令，固态硬盘每秒也只能处理几万个命令，但是固态硬盘可能会产生严重的写入放大问题，降低固态硬盘的寿命。
   - everysec，每秒1次，与不使用持久化特性时的性能相差无几，即使崩溃最多也就丢失1秒之内的数据。
   - no，让操作系统决定何时进行同步，可能会丢失不定量的数据。

3. 重写/压缩AOF文件
   - AOF文件可能会非常大，一方面会占用大量硬盘空间，另一方面还原执行的操作也非常耗时。
   - 为了解决AOF体积不断增大的问题，用户可以向redis发送bgrewriteaof

4. 复制

   - 如果用户在启动redis服务器的时候指定了一个包含slaveof host port选项的配置文件，那么redis服务器将根据该选项指定的IP地址和端口号来连接主服务器。对于运行中的redis服务器，用户可以通过发送slaveof no one不再接受主服务器的数据更新，也可以执行slaveof host port命令让服务器开始复制一个新的主服务器。

     ![从服务器连接主服务器时的步骤.png](https://s2.loli.net/2022/05/02/NKxVr3TtsJSaP4k.png)

   ![image-20210204153527495.png](https://s2.loli.net/2022/05/02/VSAjD4Mv8UseFg2.png)

5. 主从链

   - 创建多个从服务器会造成网络不可用，当在不同的数据中心或者是在互联网上时尤其如此，但是从服务器也可以有自己的从服务器以此形成主从链。

   - 从->从与主->从唯一区别在于，当从服务器执行4-2中的命令4时，它将断开与自己从服务器的连接，导致自己从服务器需要重新连接并重新同步

   - 当读请求重要性明显高于写请求时，为了解决读需求与主从同步的问题，使用主从链来解决，类似于下图：

     ![image-20210204162606009.png](https://s2.loli.net/2022/05/02/T97byOsk5BgvtMW.png)

6. 处理系统故障

   - 验证快照文件和AOF文件

     redis-check-aof可以在系统遇到故障时进行数据恢复，检查AOF文件状态，加入fix参数以后会对AOF文件进行修复，他会寻找不正确或者不完整的命令，当发现第一个错误命令时删掉之后的所有出错命令，大多数情况下删除的都是AOF文件末尾不完整的写命令。

   - 更换故障主服务器

     - 当A主B从时，A发生故障，用户决定将C作为新的主服务器。首先向机器B发送一个save命令，让他创建一个新的快照文件，接着将这个快照文件发送给C，并在机器C上启动redis，最后让机器B成为机器C的从服务器。

     - 另一种方法是将机器B直接升级为主服务器，并为升级后的主服务器创建从服务器。

7. 



# 问题



## 大量key同一时间过期

> 短暂卡顿、缓存雪崩

## Redis分布式锁

https://mp.weixin.qq.com/s/2xhjrkpZNaA6NH4MV0gz6A

> setnx(key,requestId,"NX","EX",expireTime)
>
> requestId:加解锁是一个客户端，将expireTime避免宕机无法释放锁，整条原子命令避免加锁以后宕机无法加上超时时间
>
> 有效时间设置多长？
>
> 1. 靠业务自己预估但是不靠谱
>
> 2. 看门狗机制实现锁续期
>
> 怎么实现可重入？
>
> 使用hset加减值

## 大量同前缀的key搜索

>keys可以扫描，但是会阻塞造成性能问题
>
>scan指令可以无阻塞的提取出指定模式的key列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用keys指令长。

## 实际情景

> 怎么用redis限制每个人每分钟访问3次

## 击穿、雪崩、穿透

>1. 如果在某个key（通常是热点key）失效的时候，有大量的请求一起过来就是**击穿**
>   - 开一个后台线程监控数据，变化时就缓存进去，这种解决方案只适用于不要求数据严格一致性的情况，因为当后台线程在构建缓存的时候，其他的线程很有可能也在读取数据，这样就会访问到旧数据了。
>   - 当 key 失效的时候，让一个线程读取数据并构建到缓存中，其他线程就先等待，直到缓存构建完后重新读取缓存。当然，采用互斥锁的方案也是有缺陷的，当缓存失效的时候，同一时间只有一个线程读数据库然后回写缓存，其他线程都处于阻塞状态。如果是高并发场景，大量线程阻塞势必会降低吞吐量。这种情况该如何处理呢？我只能说没什么设计是完美的，你又想数据一致，又想保证吞吐量，哪有那么好的事，为了系统能更加健全，必要的时候牺牲下性能也是可以采取的措施，两者之间怎么取舍要根据实际业务场景来决定，万能的技术方案什么的根本不存在。
>
>2. 如果大量的请求访问多个key，刚好key同时失效了就是**雪崩**
>   - 热点key永不过期
>   - key设置不同的失效时间
>   - 互斥锁（只有一个读库更新缓存，别的读缓存）
>   - 主备缓存，备缓存有效期长，获取锁失败时读取备份缓存，更新主缓存时更新备用缓存
>   - 缓存预热
>   - 缓存降级，指缓存失效或缓存服务器挂掉的情况下，不去访问数据库，直接返回默认数据或访问服务的内存数据
>
>3. 如果大量的用户请求缓存中不存在的key（甚至数据库也不存在，比如攻击）就是**穿透**
>   - 缓存空对象，如果有大量的 key 穿透，缓存空对象会占用宝贵的内存空间。空对象的 key 设置了过期时间，这段时间内可能数据库刚好有了该 key 的数据，从而导致数据不一致的情况。
>   - 布隆过滤器， m 比特的位数组（bit array）与 k 个哈希函数（hash function）组成的数据结构，优点是节省空间、时间复杂度低，缺点准确率有误，不能删除元素

## redis为什么这么快

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

  - 压缩链表

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

## 双写一致性

https://mp.weixin.qq.com/s/OiElz_morUKteHnTuhcZUg

# 对文章进行投票例子

https://blog.csdn.net/weixin_33922670/article/details/88902943

## 背景

>- 至少200张票是有趣文章，把所有的有趣文章放到前100位至少一天
>- 随时间流逝评分不断减少以避免票数多的一直停留在列表前列
>
>将文章的支持票数乘以一个常量432（由一天的秒数86400除以文章展示一天所需的支持票200得出），然后加上文章发布时间(从1970.1.1到文章的发布时间的差值，越新的文章这个值越高)得出的结果就是文章评分。
>
>1. 使用散列存储文章信息，key为article:id，value为文章信息包括title、link、poster、time、votes
>2. 使用有序集合存储文章发布时间，key为time:，value为article:id，分值为发布时间（便于根据发布时间排序）
>3. 使用有序集合存储文章id，key为score:，value为article:id，分值为文章评分（便于根据评分排序）
>4. 为了防止一个用户多次投票，用集合记录已投票用户，voted:id为key,value为user:id
>5. 为了让用户只看到特定话题的文章，用集合记录文章，key为groups:话题，value为article:id
>
>为了节约内存，当一篇文章发布期满一周之后用户将不能对它投票，文章评分将被固定而记录已投票用户的名单也将被删除。

## 用户发表文章

- 通过计数器创建新的文章ID
- 添加发布者ID到记录文章已投票用户名单的集合里面，并设置过期时间让文章期满一周之后自动删除这个集合
- 存储文章相关信息
- 将文章的初始评分和发布时间分别添加到两个相应的有序集合中

## 面试题

https://mp.weixin.qq.com/s/SMkFzjSBvPvqA_ibFT6xCA

## 安装

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



