## 哪些情况会导致stop the world？各种GC回收器STW几次?

## 对象分配的过程？

## 虚拟机怎么保证多线程分配对象的正确性？

## 常用的JVM调优工具？

## 常用的JVM启动参数？

## 什么是内存震荡？为什么要避免内存震荡？
> -Xms和-Xmx不一致时会导致JVM反复增加和缩减堆内存，这会导致频繁GC(吞吐变低)，但是每次GC的时间相对于稳定的堆却有所下降(响应时间变小)
> 当堆的空闲比例低于-XX:MinHeapFreeRatio(默认40)扩展，高于-XX:MaxHeapFreeRatio(默认70)压缩
> 只有-Xms和-Xmx不一致时上述参数才会生效

## OOM排查

>1. 一次性申请太多，一次查1000W数据
>
>   更改申请对象数量
>
>2. 内存资源耗尽未释放
>
>   未释放对象，频繁建立RPC客户端、JDBC线程而不释放
>
>3. 本身资源不够
>
>   jmap -heap 进程号          查看堆信息
>
>系统已经OOM挂了:提前设置-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=,内存溢出时会导入到指定文件名称（如果没设置神仙难救，根据堆栈信息也可以但是实际系统很复杂很难找到），然后结合visualVM进行定位，找到较大的对象，然后根据GCroot进行查找
>
>系统还没挂掉：导出dump文件：jmap -dump:format=b,file=xxx.hprof 进程号，但是注意会引起GC，导致STW  或者可以使用一些工具，arthas（阿里的工具应该是）
>
>结合jvisualvm进行调试：查看最多跟业务有关对象-> 找到GCROOT->查看线程栈

> 

