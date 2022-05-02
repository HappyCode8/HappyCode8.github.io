## 数据结构

1. 字符串

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

2. 列表

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

3. 集合

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

4. 散列

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

5. 有序集合

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