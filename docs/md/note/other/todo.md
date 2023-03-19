| 准备的内容              | 时间                 | 备注 |
| ----------------------- | -------------------- | ---- |
| redis、kafka、mysql、ES | 2.4  2.18 3.4 3.18   |      |
| 链表、树、动态规划      | 2.5  2.19 3.5 3.19   |      |
| 多线程、集合、jvm       | 2.11  2.25 3.11 3.25 |      |
| 项目、设计、hr          | 2.12  2.26 3.12 3.26 |      |

## 项目

>- 配置平台
>  - 优化接口响应时间，做数据预聚合，拆表
>  - 开发业务，做机器人的配置、鉴权等等业务开发
>- 中控
>  - 优化端到端时延，业内一般是1000ms以上，优秀的阿里小蜜可以做到600-700ms，谷歌800ms，我们800ms，人工小于500ms
>    - 1000ms优化到800ms
>      - 非技术手段，切除语音包的空白，80ms
>      - 缩短语音包，可以更快地返回VAD信号（表明说完），80ms
>      - 缓存优化（双级缓存+一致性哈希），18次降低到5次，40ms
>      - 总共优化掉80+80+40=200
>  - 灰度（指定某机器人优先路由到特定机器上）
>  - 防骚扰（网关上拒绝）
>  - 限流（租户级别的限流）
>  - 降级（网关上直接返回）
>  - AB实验（网关上做计数，然后做流量的染色）
>  - 一致性哈希：使用treemap做存储<虚拟节点，实际服务器>，每次对于所得到的请求哈希，然后获得treemap离这个哈希最近的值，然后取实际服务器。之所以要做一致性哈希，主要就是为了防止服务发布、扩容、缩容时采用哈希策略所导致的找不到服务器。
>- 数仓
>  - 整合数据，统一存储
>  - 梳理维度，整合维度

## 自我介绍

>20年研究生毕业于XXX，毕业以后一直在XXX做java开发的相关工作，做的主要的业务是XXX，就是用XXX，具体到我们部门是XXX，我个人所做的主要工作是做一些配置平台类的开发，主要包括XXX，也做中控相关的开发，主要是用来连接XXX与XXX，将XXX平台的语音流与数据流导入到我们的平台内部。

## 其它

> - 离职原因
>
>   一方面，现在的工作都比较熟悉了，不想待在舒适区了，想看看其他的业务与技术，尝试更多的可能性，另一方面，我所在的部门是算法部门，工程工作受限，工作中很多偏工具类的算法支撑工作比较琐碎，比较薄弱没有深度，缺乏积累。最后，在公司也学到了很多，对行业岗位也有了更深的理解，让我有信心开始新的职场阶段。
>
> - 优缺点
>
>   跨部门沟通有所欠缺，过去没有意识到这个问题，随着工作深入发现沟通能力有时很重要，因此专门看了一些相关的书籍，向有经验的人学习，慢慢尝试改变自己的工作方式，最近已经有所好转。
>
> - 职业发展
>
>   行业专家，两条腿走路，深入掌握技术，掌握行业技术解决方案，在此基础上理解业务，最终成为某块业务的技术专家

- 本周待办
  1. 聊聊项目，好的设计，好的代码
  2. 谈谈什么是零拷贝？
  3. 一共有几种 IO 模型？NIO 和多路复用的区别？
  4. Future 实现阻塞等待获取结果的原理？
  5. ReentrantLock和 Synchronized 的区别？Synchronized 的原理？
  6. 聊聊AOS？ReentrantLock的实现原理？
  7. 乐观锁和悲观锁， 让你来写你怎么实现？
  8. Paxos 协议了解？工作流程是怎么样的？
  9. B+树聊一下？B+树是不是有序？B+树和B-树的主要区别？
  10. TCP的拥塞机制
  11. 工作中有过JVM实践嘛
  12. 数据库分库分表的缺点是啥？
  13. 分布式事务如何解决？TCC 了解？
  14. RocketMQ 如何保证消息的准确性和安全性？
  15. 算法题：三个数求和
  16. 1.TCP/IP 网络模型有几层？分别有什么用？
  17. 2.介绍一下 HTTP 协议吧
  18. 3.GET 和 POST有什么区别？
  19. 4.PING 的作用？
  20. 5.常见的 HTTP 状态码有哪些
  21. 6.HTTP1.1 和 HTTP1.0 的区别有哪些？
  22. 7.HTTPS 和 HTTP 的区别是什么？
  23. 8.HTTP2 和 HTTP1.1 的区别是什么？
  24. 9.HTTP3 和 HTTP2 的区别是什么？
  25. 10.TCP 建立连接的过程是怎样的？
  26. 11.为什么是三次握手？？？
  27. 12.TCP 断开连接的过程是怎样的？
  28. 13.第四次挥手为什么要等待2MSL(60s)
  29. 14.为什么是四次挥手？
  30. 15.TCP 滑动窗⼝是什么？
  31. 16.发送方一直发送数据，但是接收方处理不过来怎么办？（流量控制）
  32. 17.TCP 半连接队列和全连接队列是什么？
  33. 18.粘包/拆包是怎么发生的？怎么解决这个问题？
  34. 19.浏览器地址栏输入网站按回车后发生了什么？

# 面试实际问题

1. 线程池原理
2. redis数据结构以及底层实现
3. java锁、对象头、同步原理
4. java新特性
5. 两个数字字符串相减
6. java hashmap数据结构、扩容原理
7. 带指向父指针的最近公共祖先

最长公共子串，找最接近某个数的和，两次买卖股票问题

拦截器、过滤器、AOP



redis限流、单用户1分钟访问10次

- 基于Redis的setnx的操作，过期时间设置为1分钟，有就加次数，超过10返回，没有新建

mysql深分页、在可重复读下怎么解决幻读

- 



java线程安全https://mp.weixin.qq.com/s/0ZofmJwoeCGUSgSJoo8zwQ

限流https://mp.weixin.qq.com/s/kS-8TYl2XgXsVgy9KWzLAw

生成订单30分钟未支付自动取消https://mp.weixin.qq.com/s/-fmKcw2m2eb6NRAmcXfBhw

ThreadLocalhttps://mp.weixin.qq.com/s/BnMobn2DRaZabBfApipOdg

springcloud实战项目https://mp.weixin.qq.com/s/B2sCOTFGAcdNTLKHG5ByDw

如何减少bughttps://mp.weixin.qq.com/s/ErGEV5BC7uu4qwi6npEuHQ

java中的锁https://mp.weixin.qq.com/s/l6ee7k0n7CCVFgBS4tI2kQ

kafkahttps://mp.weixin.qq.com/s/8KqaDsQoyx9P-Om81GNvlQ

各家面试题https://mp.weixin.qq.com/s/FAbcFwvo9uqtDmzDdy8r5A

手写springmvchttps://mp.weixin.qq.com/s/1wnqzuXSAaJmr_4XIH79XA

定时任务原理https://mp.weixin.qq.com/s/w_26-slajnM57HIbu4b5CA

高性能网关https://mp.weixin.qq.com/s/EX2WQDyudCpoXRfQZlnlGQ

手写RPChttps://mp.weixin.qq.com/s/fc_1aaM2pfZLPnkoH_5h9g

千万级流量处理https://mp.weixin.qq.com/s/lNofjoW2kt0p_QbG9PWQWA

注解和反射https://mp.weixin.qq.com/s/gLlCzVMLWuJxTsucUlchEg

秒杀源码https://github.com/MaJesTySA/miaosha_Shop

美团全链路压测https://tech.meituan.com/2018/09/27/quake-introduction.html

参数偏移 https://blog.csdn.net/iamhuanggua/article/details/85008408

mybatis的日志功能如何设计的https://mp.weixin.qq.com/s/hWE8S8Zzy6m1-YhbqrgYDg

社招一年半面经分享https://mp.weixin.qq.com/s/SOErvCCrmPaAVUphSO2Wqw

jd:

```
熟悉linux、数据结构、算法、设计模式、java、缓存、消息队列、负载均衡、分布式存储、分布式事务
高并发、大数据量、微服务、docker、k8s
```



string为什么是不可变的

代理的原理

IOC和AOP

BIO AIO NIO

