# 多线程

http://book.bugstack.cn/#s/50S6w8vw

## 进程与线程

- 最初的计算机：用户每输入一个指令，计算机就做出一个操作，当用户在思考或者输入时。计算机就在等待。
- 批处理操作系统：把一系列的指令写下来形成清单一次性交给计算机执行
- 进程的提出：采用CPU轮转让操作系统并发，每个程序执行在一个进程中
- 线程的提出：进程包含多个线程，每个线程单独负责一个子任务

## 多进程与多线程都可以实现并发，为什么要使用多线程？

- 进程间通信比较复杂，线程间资源共享简单
- 进程是重量级的，线程是轻量级的，多线程开销更小

## 并发问题的根源

- CPU缓存引起的可见性
- 分时复用引起的原子性
- 重排序引起的有序性
  - 编译器优化的重排序（编译时禁止特定类型的重排序）
  - 指令级并行重排序（生成指令序列时插入内存屏障）
  - 内存系统重排序（生成指令序列时插入内存屏障）

## happens_before规则

- 单一线程（线程内前发生于后）
- 管程锁定（unlock发生于后边的lock）
- volatile规则（对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作）
- 线程启动规则（Thread 对象的 start() 方法调用先行发生于此线程的每一个动作）
- 线程加入规则（Thread 对象的结束先行发生于 join() 方法返回）
- 线程中断规则（对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生）
- 对象终结规则（一个对象的初始化完成(构造函数执行结束)先行发生于它的 finalize() 方法的开始）
- 传递性（如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C）

## 线程安全是非真即假的吗

- 无状态

  ```java
  public class NoStatusService {
  
      public void add(String status) {
          System.out.println("add status:" + status);
      }
  
      public void update(String status) {
          System.out.println("update status:" + status);
      }
  }
  ```

- 不可变（不可变(Immutable)的对象一定是线程安全的）final,String,枚举等

  ```java
  public class NoChangeService {    
    public static final String DEFAULT_NAME = "abc";
    public void add(String status) {        
      System.out.println("add status:" + status);    
    }
  }
  ```

- 安全发布

  如果类中有公共资源，但是没有对外开放访问权限，即对外安全发布，也没有线程安全问题

  ```java
  public class SafePublishService {
      private String name;
  
      public String getName() {
          return name;
      }
  
      public void add(String status) {
          System.out.println("add status:" + status);
      }
  }
  ```

- 绝对线程安全

- 相对线程安全（Vector、HashTable、Collections 的 synchronizedCollection() 方法包装的集合）

- 线程兼容（ArrayList 和 HashMap自己同步）

- 线程对立（无论调用端是否采取了同步措施，都无法在多线程环境中并发使用的代码，由于 Java 语言天生就具备多线程特性，线程对立这种排斥多线程的代码是很少出现的，而且通常都是有害的，应当尽量避免）

## 线程安全的实现方法

- 互斥同步（Synchronized、ReentrantLock）

- 非阻塞同步

  - CAS
  - Atomic类
  - ABA

- 无同步

  - 栈封闭

  - 线程本地存储（ThreadLocal）

    ```java
    public class ThreadLocalExample {
        public static void main(String[] args) {
            ThreadLocal threadLocal = new ThreadLocal();
            Thread thread1 = new Thread(() -> {
                threadLocal.set(1);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(threadLocal.get());
                threadLocal.remove();
            });
            Thread thread2 = new Thread(() -> {
                threadLocal.set(2);
                threadLocal.remove();
            });
            thread1.start();
            thread2.start();
        }
    }
    ```

  - 可重入代码

    > 这种代码也叫做纯代码(Pure Code)，可以在代码执行的任何时刻中断它，转而去执行另外一段代码(包括递归调用它本身)，而在控制权返回后，原来的程序不会出现任何错误。
    >
    > 可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不调用非可重入的方法等。

## 线程状态及其转换

Java线程有6个状态

```java
public enum State {
        NEW,
        RUNNABLE,
        BLOCKED,
        WAITING,
        TIMED_WAITING,
        TERMINATED;
 }
```

- 处于NEW状态的是尚未调用Start的方法

```java
var Thread=new Thread();
System.out.println(Thread.getState());
```

- 处于RUNNABLE状态的线程在Java虚拟机中允许，也有可能在等待其它系统资源，它包含了传统操作系统的ready和running两个状态

- 处于BLOCKED状态的线程正在等待锁的释放以进入同步区

- 处于WAITING状态的线程变成RUNNABLE状态需要其它线程唤醒，调用以下方法进入等待

  - Object.wait()
  - Thread.join()
  - LockSupport.park()

  使用notify、notifyAll唤醒

- 处于TIMED_WAITING的线程等待一个具体的时间，时间到后会被自动唤醒

  - Thread.sleep
  - Object.wait()带超时时间
  - Thread.join()带超时时间
  - LockSupport.parkNanos() 方法
  - LockSupport.parkUntil() 方法

- 处于TERNINATED的线程终止状态，此时线程已经执行完毕

## 多线程实现方式

- 继承Thread类实现多线程

```java
public class Demo {
    public static class MyThread extends Thread{
        @Override
        public void run(){
            System.out.println("MyThread");
        }
    }
    public static void main(String[] args) {
        var thread=new MyThread();
        thread.start();//调用start方法以后，虚拟机会先创建一个线程，然后等到这个线程第一次得到时间片时再调用run方法
    }
}
```

- 实现Runnable接口

```java
public class Demo {
    public static class MyThread implements Runnable{
        @Override
        public void run(){
            System.out.println("MyThread");
        }
    }
    public static void main(String[] args) {
       new Thread(new MyThread()).start();
       new Thread(()->{
           System.out.println("java8线程方法");
       }).start();
    }
}
```

java8以后可以使用函数式接口：

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}

public static void main(String[] args) {
       new Thread(()->{
           System.out.println("java8线程方法");
       }).start();
 }
```

- Callable接口

```java
public class MyCallable implements Callable<Integer> {
    public Integer call() {
        return 123;
    }
}

public static void main(String[] args) throws ExecutionException, InterruptedException {
    MyCallable mc = new MyCallable();
    FutureTask<Integer> ft = new FutureTask<>(mc);
    Thread thread = new Thread(ft);
    thread.start();
    System.out.println(ft.get());
}
```

## 基础线程机制

- Excutor是一个接口，包含execute方法

- ExecutorService继承了Excutor接口，包含shutdown、submit方法

- Executors是一个工具类包含三种线程池

  - fixed（n核心，n线程）
  - catche（0，max）
  - single（1，1）
  - Scheduled

- Dameon

- sleep

- Yield

## 线程中断

- InterruptedException

  > 调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。
  >
  > 但是不能中断 I/O 阻塞和 synchronized 锁阻塞

- interrupted

  > 如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。
  >
  > 但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。
  >
  > ```java
  > public class InterruptExample {
  > 
  > private static class MyThread2 extends Thread {
  >     @Override
  >     public void run() {
  >         while (!interrupted()) {
  >             // ..
  >         }
  >         System.out.println("Thread end");
  >     }
  > }
  > }
  > 
  > public static void main(String[] args) throws InterruptedException {
  > Thread thread2 = new MyThread2();
  > thread2.start();
  > thread2.interrupt();
  > }
  > ```

- Executor 的中断操作

  调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

## 线程互斥同步

- synchronized
  - 代码块（作用于对象）
  - 方法（作用于对象）
  - 类、静态方法（作用于类）
- ReentrantLock
  - 等待可中断、可公平、可绑定多个条件

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

## 线程协作

- join
- wait() notify() notifyAll()
  - 属于 Object 的一部分，而不属于 Thread，只能用在同步方法或者同步控制块中使用
  - 使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。
- await() signal() signalAll()

## wait() 和 sleep() 的区别

>  sleep是Thread的静态方法，因此作用于当前线程，用来暂停一个指定的时间内不运行，他不释放资源，主要依靠超时唤醒。
>
>  wait是Object的实例方法，因此作用于对象本身，用来与同步块中其它线程间的通信，他释放了资源，只能在同步上下文中调用wait方法，唤醒时依赖其他线程调用对象的notify()或者notifyAll()方法。
>
>  **为什么wait必须要在同步块中？**
>
>  由于Lost Wake-Up Problem的存在：
>
>  ```java
>  class BlockingQueue {
>  Queue<String> buffer = new LinkedList<String>();
>  
>  public void give(String data) {
>  buffer.add(data);
>  notify();//2
>  }
>  
>  public String take() throws InterruptedException {
>  while (buffer.isEmpty())//1成功判空
>      wait();//3
>  return buffer.remove();
>  }
>  }
>  //按照1->3->2的顺序执行，消费者就会消费不到消息，解决这个问题的方法就是：总是让give/notify和take/wait为原子操作。也就是说wait/notify是线程之间的通信，他们存在竞态，我们必须保证在满足条件的情况下才进行wait。换句话说，如果不加锁的话，那么wait被调用的时候可能wait的条件已经不满足了(如上述)。由于错误的条件下进行了wait，那么就有可能永远不会被notify到，所以我们需要强制wait/notify在synchronized中
>  
>  //1处的代码如果用if的话可能会产生虚假唤醒，所谓虚假唤醒就是只生产了一个但是却唤醒了所有在此条件上等待的线程，AB两消费线程都在执行完1以后执行了wait条件，释放了同步锁，此时程序阻塞在1上，生产了一个产品以后，A消费以后B没有再判断就继续执行了
>  ```

## 进程与线程的区别

- 本质区别是是否单独占有内存地址空间及其他系统资源
- 进程单独占有内存地址空间，所以进程间存在内存隔离，数据是分开的，数据共享复杂，但是同步简单；线程相反，数据共享简单，但是同步复杂
- 进程稳定，一个崩溃不会影响其他，线程崩溃可能影响整个程序的稳定
- 进程创建和销毁不但需要保存寄存器和栈信息，还需要资源分配回收以及页调度，开销较大，线程只需要保存寄存器和栈信息，开销较小
- 进程是操作系统进行资源分配的基本单位，线程是操作系统进行调度的基本单位

## 上下文切换

- 上下文是指某一时间点CPU寄存器和程序计数器的内容，上下文切换是把这些内容保存到内存中，上下文恢复是把内存中的内容拿出来恢复到寄存器和程序计数器，上下文切换是计算密集型的，会消耗大量CPU时间，故线程也不是越多越好。

## Callable、Future、FutureTask

Callable也是一个函数式接口，Callable在`java.util.concurrent`包中，而Thread在`java.lang`包中

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

实际使用方法：

```java
public class DemoCallable implements Callable<Integer> {

    @Override
    public Integer call() throws Exception {
        Thread.sleep(2000);
        return 0;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        var executor = Executors.newCachedThreadPool();
        var demoCallable = new DemoCallable();
        var result = executor.submit(demoCallable);
        System.out.println(result.get());//get会阻塞，最好设置超时
    }
}
```

Future接口只有几个比较简单的方法：

```java
package java.util.concurrent;

public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

cancle试图取消一个线程的执行，注意是试图取消，并不一定能取消成功。因为任务可能已完成、已取消、或者一些其它因素不能取消，存在取消失败的可能。boolean类型的返回值是“是否取消成功”的意思。参数paramBoolean表示是否采用中断的方式取消线程执行。所以有时候，为了让任务有能够取消的功能，就使用Callable来代替Runnable。如果为了可取消性而使用Future但又不提供可用的结果，则可以声明Future<?>形式类型、并返回null作为底层任务的结果。

FutureTask实现了RunnableFuture接口，RunnableFuture接口同时继承了Runnable、Future接口

```java
public class FutureTask<V> implements RunnableFuture<V>
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask的实现

```java
public class DemoFutureTask implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        Thread.sleep(3000);
        return 0;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        var executor = Executors.newCachedThreadPool();
        var futureTask = new FutureTask<>(new DemoFutureTask());
        executor.submit(futureTask);
        System.out.println(futureTask.get());
    }
}
```

在很多高并发的环境下，有可能Callable和FutureTask会创建多次。FutureTask能够在高并发环境下确保任务只执行一次。

## 线程组和线程优先级

- 线程组，每个线程都要属于一个线程组，如果没有就默认父线程设置为自己的线程组

```java
public class Demo {
    public static void main(String[] args) {
        Thread testThread = new Thread(() -> {
            System.out.println("testThread当前线程组名字：" +
                    Thread.currentThread().getThreadGroup().getName());
            System.out.println("testThread线程名字：" + 
                    Thread.currentThread().getName());
        });
        testThread.start();
        System.out.println("执行main方法线程名字：" + 
                Thread.currentThread().getName());
    }
}
```

- 线程的优先级

```java
public class Demo {
    public static void main(String[] args) {
        Thread a = new Thread();
        System.out.println("默认的线程优先级：" + a.getPriority());
        Thread b = new Thread();
        b.setPriority(8);
        System.out.println("设置过的线程优先级：" + b.getPriority());
    }
}
```

但是注意这个优先级不太可靠，这个优先级只是给操作系统的一个建议，操作系统未必采纳，真正的调用顺序是由操作系统的线程调度算法决定的

```java
public class Demo {
    public static void main(String[] args) {
        IntStream.rangeClosed(1,10).forEach(it->{
            var thread=new Thread(()->
                System.out.println("当前线程："+Thread.currentThread().getName()+
                        ",线程优先级："+Thread.currentThread().getPriority())
            );
            thread.setPriority(it);
            thread.start();
        });
    }
}

当前线程：Thread-4,线程优先级：5
当前线程：Thread-3,线程优先级：4
当前线程：Thread-1,线程优先级：2
当前线程：Thread-7,线程优先级：8
当前线程：Thread-5,线程优先级：6
当前线程：Thread-9,线程优先级：10
当前线程：Thread-6,线程优先级：7
当前线程：Thread-8,线程优先级：9
当前线程：Thread-0,线程优先级：1
当前线程：Thread-2,线程优先级：3
```

如果某个线程优先级大于线程组的优先级，那么线程优先级会失效用线程组的优先级代替

- 线程组做统一异常处理

```java
public class Demo {
    public static void main(String[] args) {
        var threadGroup=new ThreadGroup("group1"){
            @Override
            public void uncaughtException(Thread t, Throwable e) {
                System.out.println(t.getName()+":"+e.getMessage());
            }
        };

        var thread1 = new Thread(threadGroup, () -> {
             throw new RuntimeException("测试异常");
        });
        thread1.start();
    }
}
```

- 总结来说，线程组是一个树状的结构，每个线程组下面可以有多个线程或者线程组。线程组可以起到统一控制线程的优先级和检查线程的权限的作用。

## ThreadLocal

- 原理

  ```java
  class ThreadLocal{
      static class ThreadLocalMap {
          static class Entry extends WeakReference<ThreadLocal<?>>{
               Object value;
          }
          private Entry[] table;
      }
   }
  
   class Thread{
             ThreadLocal.ThreadLocalMap threadLocals = null;
   }
   //所以每一个Thread有一个threadLocals数组，每一个数组元素存的是一个ThreadLocalMap，每一个ThreadLocalMap里边<ThreadLocal的hashcode，value>，解决hash冲突的时候用的是向下搜寻相当于再散列。
   //使用InheritableThreadLocal可以实现多个线程访问ThreadLocal的值
   //内存泄露原因是这样的，Thread一直持有threadLocals，也就是一直持有数组，数组持有value的强引用但是key是弱引用，因而可能有内存泄露。
  ```

- 应用

  > 1. java Web是一个单例多线程的模型。
  >
  > 即通常情况下，Web程序中的每一个Bean都是单例，而用户的每次请求都会对应一个独立的线程，当多个用户同时访问Web程序时，表现出来的就是单例多线程。
  > 在这种模型中，如果需要在一次请求周期中保存某些用户信息，那这个信息绝对不能放到类的成员变量中去。因为类的成员变量是隶属于同一个Bean的，而Bean是被多个线程所共享的，会导致多个线程更改同一个成员变量的情况，程序表现为一会正常一会不正常。这也是实践中经常遇到的bug。
  >
  > 对于这种场景，就要使用ThreadLocal类了。通常的做法是在请求进入Bean之前把相关的信息保存到ThreadLocal变量中，待后续需要的时候从ThreadLocal中获取即可。
  >
  > 2. jdbc = new DataBaseConnection();//第1行
  >    Connection con = jdbc.getConnection();//第2行
  >    con.setAutoCommit(false);//第3行
  >    con.executeUpdate(...);//第4行
  >    con.executeUpdate(...);//第5行
  >    con.executeUpdate(...);//第6行
  >    con.commit();//第7行
  >    Spring框架包揽了事务准备阶段和事务提交阶段的代码，使得程序员专注于设计业务处理阶段的代码。Spring框架可以使用AOP（Aspect Oriented Programming）来动态的织入准备阶段和事务提交阶段的代码。但如何才能让三个阶段使用同一个数据源连接呢？这是很重要的。
  >
  > 在这个场景中，我们实际上是希望让软件结构中纵向的三个阶段使用同样的一个参数con，而这三个阶段之间又无法进行显式的参数传递。解决方案是---ThreadLocal。Spring框架使用ThreadLocal记录每个线程在进行事务操作时所使用的数据库连接，在事务准备阶段，将数据库连接放入ThreadLocal中，在业务处理阶段和事务提交阶段直接从ThreadLocal中获取对应的连接即可。这个是ThreadLocal的经典应用。
  >
  > 3. 在spring事务中，保证一个线程下，一个事务的多个操作拿到的是一个Connection。
  > 4. 在hiberate中管理session。
  > 5. 在JDK8之前，为了解决SimpleDateFormat的线程安全问题。
  > 6. 获取当前登录用户上下文。
  > 7. 临时保存权限数据。
  > 8. 使用MDC保存日志信息。

- 为什么Entry的key是弱引用，value是强引用

  key是弱引用是因为有线程池，由于Thread变量 -> Thread对象 -> ThreadLocalMap -> Entry -> key -> ThreadLocal对象的强引用，即使线程执行完也不会GC，产生内存泄露

  value有可能没被业务系统中的其他地方引用

- 假如ThreadLocalMap中存在很多key为null的Entry，但后面的程序，一直都没有调用过有效的ThreadLocal的`get`、`set`或`remove`方法。

  那么，Entry的value值一直都没被清空。

  所以会存在这样一条`强引用链`：Thread变量 -> Thread对象 -> ThreadLocalMap -> Entry -> value -> Object。

  其结果就是：Entry和ThreadLocalMap将会长期存在下去，会导致`内存泄露`。

- 调用remove方法把kv都设置为null就会解决内存泄露

- ThreadLocal如何定位数据

  1. 通过key的hashCode取余计算出一个下标。
  2. 通过下标，在数组中定位具体Entry，如果key正好是我们所需要的key，说明找到了，则直接返回数据。
  3. 如果第2步没有找到我们想要的数据，则从数组的下标位置，继续往后面找。
  4. 如果第3步中找key的正好是我们所需要的key，说明找到了，则直接返回数据。
  5. 如果还是没有找到数据，再继续往后面找。如果找到最后一个位置，还是没有找到数据，则再从头，即下标为0的位置，继续从前往后找数据。
  6. 直到找到第一个Entry为空为止。

- ThreadLocal如何扩容

  1. 老size + 1 = 新size
  2. 如果新size大于等于老size的2/3时，需要考虑扩容。
  3. 扩容前先尝试回收一次key为null的值，腾出一些空间。
  4. 如果回收之后发现size还是大于等于老size的1/2时，才需要真正的扩容。
  5. 每次都是按2倍的大小扩容

- 父子线程如何共享数据

  换成InheritableThreadLocal之后，在子线程中能够正常获取父线程中设置的值。

  其实，在Thread类中除了成员变量threadLocals之外，还有另一个成员变量：inheritableThreadLocals。

  Thread类的部分代码如下：

  ```java
  ThreadLocal.ThreadLocalMap threadLocals = null;
  ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
  ```

  最关键的一点是，在它的`init`方法中会将父线程中往ThreadLocal设置的值，拷贝一份到子线程中。

  我们应该使用InheritableThreadLocal，具体代码如下：

  ```java
  private static void fun1() {
      InheritableThreadLocal<Integer> threadLocal = new InheritableThreadLocal<>();
      threadLocal.set(6);
      System.out.println("父线程获取数据：" + threadLocal.get());
  
      ExecutorService executorService = Executors.newSingleThreadExecutor();
  
      threadLocal.set(6);
      executorService.submit(() -> {
          System.out.println("第一次从线程池中获取数据：" + threadLocal.get());
      });
  
      threadLocal.set(7);
      executorService.submit(() -> {
          System.out.println("第二次从线程池中获取数据：" + threadLocal.get());
      });
  }
  ```

  执行结果：

  ```properties
  父线程获取数据：6
  第一次从线程池中获取数据：6
  第二次从线程池中获取数据：6
  ```

  由于这个例子中使用了单例线程池，固定线程数是1。

  第一次submit任务的时候，该线程池会自动创建一个线程。因为使用了InheritableThreadLocal，所以创建线程时，会调用它的init方法，将父线程中的inheritableThreadLocals数据复制到子线程中。所以我们看到，在主线程中将数据设置成6，第一次从线程池中获取了正确的数据6。

  之后，在主线程中又将数据改成7，但在第二次从线程池中获取数据却依然是6。

  因为第二次submit任务的时候，线程池中已经有一个线程了，就直接拿过来复用，不会再重新创建线程了。所以不会再调用线程的init方法，所以第二次其实没有获取到最新的数据7，还是获取的老数据6。

  那么，这该怎么办呢？

  答：使用`TransmittableThreadLocal`，它并非JDK自带的类，而是阿里巴巴开源jar包中的类。

  可以通过如下pom文件引入该jar包：

  ```xml
  <dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>transmittable-thread-local</artifactId>
     <version>2.11.0</version>
     <scope>compile</scope>
  </dependency>
  ```

  代码调整如下：

  ```java
  private static void fun2() throws Exception {
      TransmittableThreadLocal<Integer> threadLocal = new TransmittableThreadLocal<>();
      threadLocal.set(6);
      System.out.println("父线程获取数据：" + threadLocal.get());
  
      ExecutorService ttlExecutorService = TtlExecutors.getTtlExecutorService(Executors.newFixedThreadPool(1));
  
      threadLocal.set(6);
      ttlExecutorService.submit(() -> {
          System.out.println("第一次从线程池中获取数据：" + threadLocal.get());
      });
  
      threadLocal.set(7);
      ttlExecutorService.submit(() -> {
          System.out.println("第二次从线程池中获取数据：" + threadLocal.get());
      });
  
  }
  ```

  执行结果：

  ```properties
  父线程获取数据：6
  第一次从线程池中获取数据：6
  第二次从线程池中获取数据：7
  ```

  我们看到，使用了TransmittableThreadLocal之后，第二次从线程中也能正确获取最新的数据7了。

  nice。

  如果你仔细观察这个例子，你可能会发现，代码中除了使用`TransmittableThreadLocal`类之外，还使用了`TtlExecutors.getTtlExecutorService`方法，去创建`ExecutorService`对象。

  这是非常重要的地方，如果没有这一步，`TransmittableThreadLocal`在线程池中共享数据将不会起作用。

  创建`ExecutorService`对象，底层的submit方法会`TtlRunnable`或`TtlCallable`对象。

  以TtlRunnable类为例，它实现了`Runnable`接口，同时还实现了它的run方法：

  ```java
  public void run() {
      Map<TransmittableThreadLocal<?>, Object> copied = (Map)this.copiedRef.get();
      if (copied != null && (!this.releaseTtlValueReferenceAfterRun || this.copiedRef.compareAndSet(copied, (Object)null))) {
          Map backup = TransmittableThreadLocal.backupAndSetToCopied(copied);
  
          try {
              this.runnable.run();
          } finally {
              TransmittableThreadLocal.restoreBackup(backup);
          }
      } else {
          throw new IllegalStateException("TTL value reference is released after run!");
      }
  }
  ```

  这段代码的主要逻辑如下：

  1. 把当时的ThreadLocal做个备份，然后将父类的ThreadLocal拷贝过来。
  2. 执行真正的run方法，可以获取到父类最新的ThreadLocal数据。
  3. 从备份的数据中，恢复当时的ThreadLocal数据。

## 一个线程OOM了，其他线程是否还能运行？

> 还能运行。虽然说堆是线程共享的区域，一个线程堆抛出OOM异常，你可能会觉得其他线程也会抛出OOM异常。但其实不然，当一个线程抛出OOM异常后，它所占据的内存会全部释放掉，从而不会影响其他线程的运行。 另外如果主线程异常了，子线程还能运行吗？这个问题也是可以运行的。线程不像进程，一个进程之间的线程之间是没有父子之分的，都是平级关系。即线程都是一样的，退出了一个不会影响另外一个。

## 线程池

> - 参数
>
> ```java
> corePoolSize：核心线程的数量
> maximumPoolSize：线程池中最大的线程数量
> keepAliveTime：线程池中非核心线程空闲的存活时间
> TimeUnit：线程空闲存活时间的时间单位
> workQueue：存放任务的阻塞队列
> threadFactory：用于创建核心线程的线程工厂，可以给创建的线程自定义名字、设置守护线程等，方便查日志
> handler：线程池的饱和策略（拒绝策略），有四种类型。
> ```
>
> - 流程
>
> ```
> 往核心线程池提交->核心满了往阻塞队列扔->阻塞队列满了扩线程->max满了拒绝策略
> ```
>
> 详细的执行流程参考ThreadPoolExecutor源码
>
> - 四种拒绝策略
>
> ```
> AbortPolicy：抛出一个异常，默认的拒绝策略
> DiscardPolicy：直接丢弃任务
> DiscardOldestPolicy：丢弃队列里最老的任务，将当前这个任务继续提交给线程池。
> CallerRunsPolicy：交给线程池调用所在的线程进行处理。
> ```
>
> - 工作队列
>
> 阻塞队列区别于其他类型的队列的最主要的特点就是“阻塞”这两个字，所以下面重点介绍阻塞功能：**阻塞功能使得生产者和消费者两端的能力得以平衡，当有任何一端速度过快时，阻塞队列便会把过快的速度给降下来**。实现阻塞最重要的两个方法是 take 方法和 put 方法。
>
> ```
> ArrayBlockingQueue：有界队列，是一个用数组实现的有界阻塞队列，按FIFO排序。
> LinkedBlockingQueue：按FIFO排序任务，容量可以设置，不设置的话将是一个无边界的阻塞队列
> PriorityBlockingQueue:支持优先级排序的无界阻塞队列
> DelayBlockingQueue:使用优先级队列实现的延迟无界阻塞队列，到时才能获取
> SynchronousQueue:不存储元素的阻塞队列，也即单个元素的队列，只存一个元素
> LinkedTransferQueue:由链表结构组成的无界阻塞队列
> LinkedBlockingDeque:由链表结构组成的双向阻塞队列
> ```
>
> - 常用的线程池
>
> ```
> newFixedThreadPool(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(),threadFactory):
> 固定数目线程的线程池，内部使用LinkedBlockingQueue
> 场景：适用于处理CPU密集型的任务，确保CPU在长期工作线程使用的情况下，尽可能少的分配线程。
> 
> newCachedThreadPool(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,new SynchronousQueue<Runnable>(),threadFactory)
> 可缓存线程的线程池，内部使用SynchronousBlockingQueue
> 使用场景：用于并发量大执行大量短期的小任务。
> 
> newSingleThreadExecutor(1, 1,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>(),hreadFactory)
> 单线程的线程池，内部使用LinkedBlockingQueue）
> 使用场景：适用于串行执行任务的情景，一个任务接一个任务的执行
> 
> newScheduledThreadPool(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS, new DelayedWorkQueue());
> 定时及周期性执行的线程池，内部使用DelayQueue
> 使用场景：周期性的执行任务的场景，做一些简单的定时调度。
> 
> newWorkStealingPool窃取线程池
> newSingleThreadScheduledExecutor单线程定时任务线程池
> 
> 其实newFixedThreadPool、newCachedThreadPool、newSingleThreadExecutor 和 newWorkStealingPool方法创建和使用线程池的方法是一样的。这四个方法创建线程池返回值是ExecutorService，通过它的execute方法执行线程。
> 
> newScheduledThreadPool 和 newSingleThreadScheduledExecutor 方法创建和使用线程池的方法也是一样的, 这两个方法创建的线程池返回的值是ScheduledExecutorService，通过它的schedule方法执行线程，可以设置时间
> ```
>
> - 线程池状态
>
> ```
> RUNNING
> 该状态的线程池会接收新任务，并处理阻塞队列中的任务
> 调用shutdown()方法可以切换到SHUTDOWN状态
> 调用shutdownNow()方法可以切换到STOP状态
> 
> SHUTDOWN
> 该状态的线程池不会接收新任务，但会处理阻塞队列中的任务
> 队列为空，并且线程池中执行的任务也为空，进入TIDYING状态
> 
> STOP
> 该状态的线程池不会接收新任务，也不会处理阻塞队列中的任务，而且会中断正在执行中的任务
> 线程池中执行的任务一旦变为空，进入TIDYING状态
> 
> TIDYING
> 该状态表明所有的任务已经运行终止，记录的任务数量为0
> terminated()执行完毕，进入TERMINATED状态
> 
> TERMINATED
> 该状态表明线程池彻底终止或死亡
> ```
>
> - execute和submit的区别
>
> ```
> execute 用于提交不需要返回值的任务
> submit()方法用于提交需要返回值的任务。线程池会返回一个future类型的对象，通过这个 future对象可以判断任务是否执行成功，并且可以通过future的get()方法来获取返回值
> ```
>
> - 线程池关闭
>
> ```
> 可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池。它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。
> 
> shutdown() 将线程池状态置为shutdown,并不会立即停止：
> 停止接收外部submit的任务
> 内部正在跑的任务和队列里等待的任务，会执行完
> 等到第二步完成后，才真正停止
> 
> shutdownNow() 将线程池状态置为stop。一般会立即停止，事实上不一定：
> 和shutdown()一样，先停止接收外部提交的任务
> 忽略队列里等待的任务
> 尝试将正在跑的任务interrupt中断
> 返回未执行的任务列表
> ```
>
> - 线程池异常
>
> ```
> 1. runnnable异常，也就是业务异常
> Runable的异常不会导致线程池停止运行，其他的线程正常运行，执行Runable发生错误的线程将会被销毁会重新建一个线程
> 
> 2. 提交任务到任务队列已满异常
> 线程池使用默认的拒绝策略的时候，当线程池提交任务到任务队列已满线程池会直接抛出错误，进而影响到主线程的后续的运行如果没有在主线程中进行错误处理。提交任务到任务队列已满异常影响的范围和方式由拒绝策略决定
> 
> 3.线程池导致某些异常会导致线程池直接退出可能同时导致住线程或者主应用发生问题或者退出。
> ```
>
> - 如何实现线程池参数的动态修改
>
> ```
> 线程池的几个参数如核心线程数、最大线程数、非核心线程保有时间、拒绝策略、线程工厂等都设置了set参数。
> 因此可以通过配置中心、自己扩展ThreadPoolExecutor等方式来修改
> ```
>
> - 自己实现一个线程池
>
> ```java
> public class MyThreadPoolExecutor implements Executor {
> 
>     //记录线程池中线程数量
>     private final AtomicInteger ctl = new AtomicInteger(0);
> 
>     //核心线程数
>     private volatile int corePoolSize;
>     //最大线程数
>     private volatile int maximumPoolSize;
> 
>     //阻塞队列
>     private final BlockingQueue<Runnable> workQueue;
> 
>     public MyThreadPoolExecutor(int corePoolSize, int maximumPoolSize, BlockingQueue<Runnable> workQueue) {
>         this.corePoolSize = corePoolSize;
>         this.maximumPoolSize = maximumPoolSize;
>         this.workQueue = workQueue;
>     }
> 
>     /**
>      * 执行
>      *
>      * @param command
>      */
>     @Override
>     public void execute(Runnable command) {
>         //工作线程数
>         int c = ctl.get();
>         //小于核心线程数
>         if (c < corePoolSize) {
>             //添加任务失败
>             if (!addWorker(command)) {
>                 //执行拒绝策略
>                 reject();
>             }
>             return;
>         }
>         //任务队列添加任务
>         if (!workQueue.offer(command)) {
>             //任务队列满，尝试启动线程添加任务
>             if (!addWorker(command)) {
>                 reject();
>             }
>         }
>     }
> 
>     /**
>      * 饱和拒绝
>      */
>     private void reject() {
>         //直接抛出异常
>         throw new RuntimeException("Can not execute!ctl.count："
>                 + ctl.get() + "workQueue size：" + workQueue.size());
>     }
> 
>     /**
>      * 添加任务
>      *
>      * @param firstTask
>      * @return
>      */
>     private boolean addWorker(Runnable firstTask) {
>         if (ctl.get() >= maximumPoolSize) return false;
>         Worker worker = new Worker(firstTask);
>         //启动线程
>         worker.thread.start();
>         ctl.incrementAndGet();
>         return true;
>     }
> 
>     /**
>      * 线程池工作线程包装类
>      */
>     private final class Worker implements Runnable {
>         final Thread thread;
>         Runnable firstTask;
> 
>         public Worker(Runnable firstTask) {
>             this.thread = new Thread(this);
>             this.firstTask = firstTask;
>         }
> 
>         @Override
>         public void run() {
>             Runnable task = firstTask;
>             try {
>                 //执行任务
>                 while (task != null || (task = getTask()) != null) {
>                     task.run();
>                     //线程池已满，跳出循环
>                     if (ctl.get() > maximumPoolSize) {
>                         break;
>                     }
>                     task = null;
>                 }
>             } finally {
>                 //工作线程数增加
>                 ctl.decrementAndGet();
>             }
>         }
> 
>         /**
>          * 从队列中获取任务
>          *
>          * @return
>          */
>         private Runnable getTask() {
>             for (; ; ) {
>                 try {
>                     System.out.println("workQueue size:" + workQueue.size());
>                     return workQueue.take();
>                 } catch (InterruptedException e) {
>                     e.printStackTrace();
>                 }
>             }
>         }
>     }
> 
>     //测试
>     public static void main(String[] args) {
>         MyThreadPoolExecutor myThreadPoolExecutor = new MyThreadPoolExecutor(2, 2,
>                 new ArrayBlockingQueue<Runnable>(10));
>         for (int i = 0; i < 10; i++) {
>             int taskNum = i;
>             myThreadPoolExecutor.execute(() -> {
>                 try {
>                     Thread.sleep(1500);
>                 } catch (InterruptedException e) {
>                     e.printStackTrace();
>                 }
>                 System.out.println("任务编号：" + taskNum);
>             });
>         }
>     }
> }
> ```
>
> - 断电应该怎么处理
>
> ```
> 我们可以对正在处理和阻塞队列的任务做事务管理或者对阻塞队列中的任务持久化处理，并且当断电或者系统崩溃，操作无法继续下去的时候，可以通过回溯日志的方式来撤销正在处理的已经执行成功的操作。然后重新执行整个阻塞队列。
> 
> 也就是：阻塞队列持久化；正在处理任务事务控制；断电之后正在处理任务的回滚，通过日志恢复该次操作；服务器重启后阻塞队列中的数据再加载。
> ```

## 最佳线程数

- 经验值

  IO密集型配置线程数经验值是：2N，其中N代表CPU核数。

  CPU密集型配置线程数经验值是：N + 1，其中N代表CPU核数。

- 算法

  最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目

- 实际

  使用动态线程池，根据压测与监控随时调整

# synchronized & Reetrantlock

## synchronized

### 原理

> 对象构成
>
> - 对象头
>   - 对象标记（Mark Word，8字节）:用于存储对象自身运行时的数据，如哈希码(Hash Code)，GC分代年龄，锁状态标志，偏向线程ID、偏向时间戳等信息，这些信息与对象无关，它会根据对象的状态复用自己的存储空间。它是**实现轻量级锁和偏向锁的关键**。
>   - 类型指针（Class Pointer，8字节）:对象会指向它的类的元数据的指针，虚拟机通过这个指针确定这个对象是哪个类的实例
>   - Array length，如果对象是一个数组，还必须记录数组长度的数据
> - 实例数据
>   - 存放类的属性信息，包括父类的属性信息
> - 对齐填充（保证8字节的倍数）

- 同步代码块原理

  过monitor对象来完成的，`Monitorenter`和`Monitorexit`指令，会让对象在执行，使其锁计数器加1或者减1。每一个对象在同一时间只与一个monitor(锁)相关联，而一个monitor在同一时间只能被一个线程获得，任意线程对Object的访问，首先要获得Object的监视器，如果获取失败，该线程就进入同步状态，线程状态变为BLOCKED，当Object的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器。

  正常情况下，一个Monitorenter会有两个Monitorexit，一个是正常退出，一个是异常退出

- 同步方法原理

  多了一个标志位**ACC_SYNCHRONIZED(静态方法还会加一个ACC_STATIC来区分加对象锁还是类锁)**，作用就是一旦执行到这个方法时，就会先判断是否有标志位，如果有这个标志位，就会先尝试获取monitor，获取成功才能执行方法，方法执行完成后再释放monitor。**在方法执行期间，其他线程都无法获取同一个monitor**。归根结底还是对monitor对象的争夺，只是同步方法是一种隐式的方式来实现。

- 可重入原理：加锁次数计数器

- 为什么任何一个对象都可以成为一个锁？

  每一个对象天生带着一个对象监视器，每一个被锁住的对象都会和Monitor关联起来，ObjectMonitor里边owner会记录哪个线程持有、waitset记录处于wait状态的线程队列、entrylist记录处于等待锁block状态的线程队列，recursions记录锁的重入次数，count记录该线程获取锁的次数

### 锁优化

- 锁粗化
- 锁消除
- 轻量级锁
- 偏向锁
- 适应性自旋锁

### 锁膨胀

- 锁膨胀方向： 无锁 → 偏向锁 → 轻量级锁 → 重量级锁 (此过程是不可逆的)

- java5以前只有synchronized，这个是操作系统级别的重量操作，因为涉及到用户态和内核态之间的切换。java的线程是映射到操作系统的原生线程之上的，如果要阻塞或者唤醒一个线程需要操作系统介入，需要切换用户态与核心态，这种切换会消耗大量的系统资源，因为用户态与核心态都有各自的专用的内存空间、寄存器。

  java早期版本中，监视器锁是依赖于底层的操作系统的Mutex Lock（系统互斥）来实现的，挂起线程和恢复线程都需要转入内核态去完成，阻塞或者唤醒一个java线程需要操作系统切换CPU状态来完成，如果同步啊代码中的内容过于简单，这种切换有时候可能比用户代码执行的时间还长。

  ```xml
  <!--查看对象头工具-->
  <dependency>  
      <groupId>org.openjdk.jol</groupId>  
      <artifactId>jol-core</artifactId>  
      <version>0.9</version>  
  </dependency>
  ```

### 自旋锁

- 不陷入阻塞，而是空转CPU

### 自适应自旋锁

- 自旋时间不固定，而是由前一次在同一个锁上的自旋 时间及锁的拥有者的状态来决定的

### 锁消除

> 锁消除的主要判定依据来源于逃逸分析的数据支持。意思就是：JVM会判断再一段程序中的同步明显不会逃逸出去从而被其他线程访问到，那JVM就把它们当作栈上数据对待，认为这些数据时线程独有的，不需要加同步。此时就会进行锁消除。
>
> 当然在实际开发中，我们很清楚的知道哪些是线程独有的，不需要加同步锁，但是在Java API中有很多方法都是加了同步的，那么此时JVM会判断这段代码是否需要加锁。如果数据并不会逃逸，则会进行锁消除。比如如下操作：在操作String类型数据时，由于String是一个不可变类，对字符串的连接操作总是通过生成的新的String对象来进行的。因此Javac编译器会对String连接做自动优化。在JDK 1.5之前会使用StringBuffer对象的连续append()操作，在JDK 1.5及以后的版本中，会转化为StringBuidler对象的连续append()操作。
>
> ```java
> public static String test03(String s1, String s2, String s3) {
> String s = s1 + s2 + s3;
> return s;
> }
> ```

### 锁粗化

> 原则上，我们都知道在加同步锁时，尽可能的将同步块的作用范围限制到尽量小的范围(只在共享数据的实际作用域中才进行同步，这样是为了使得需要同步的操作数量尽可能变小。在存在锁同步竞争中，也可以使得等待锁的线程尽早的拿到锁)。
>
> 大部分上述情况是完美正确的，但是如果存在连串的一系列操作都对同一个对象反复加锁和解锁，甚至加锁操作时出现在循环体中的，那即使没有线程竞争，频繁的进行互斥同步操作也会导致不必要的性能操作。
>
> ```java
> public static String test04(String s1, String s2, String s3) {
> StringBuffer sb = new StringBuffer();
> sb.append(s1);
> sb.append(s2);
> sb.append(s3);
> return sb.toString();
> }
> ```
>
> 在上述的连续append()操作中就属于这类情况。JVM会检测到这样一连串的操作都是对同一个对象加锁，那么JVM会将加锁同步的范围扩展(粗化)到整个一系列操作的 外部，使整个一连串的append()操作只需要加锁一次就可以了。

### 可重入锁

> 进入同一线程的内部可以再次获取锁
>
> 递归方法、内部调用
>
> 可以在一定程度上避免死锁
>
> 可重入锁原理是在监视器中记录获取锁的次数

## Reetrantlock

## 区别与联系

- 相同点

  >1. 都是用来协调多线程中的共享对象、变量的访问
  >2. 都是可重入锁，即同一线程可多次获得同一锁
  >3. 都保证了可见性和互斥性

- 不同点

  |                | synchronized         | Reetrantlock                                                 |
  | -------------- | -------------------- | ------------------------------------------------------------ |
  | 底层实现       | JVM提供              | AQS中的特殊队列数据结构                                      |
  | 释放           | 自动释放             | lock和unlock配合try和finally                                 |
  | 是否可中断     | 只有发生异常时可中断 | 可通过trylock(long timeout,TimeUnit unit)设置超时时间或lockInterruptibly()放到代码块中，调用interrupt方法进行中断 |
  | 是否公平锁     | 否                   | 可通过构造函数传入boolean进行选择，默认false非公平锁         |
  | 是否可绑定条件 | 不能                 | 可通过绑定Condition结合await()和notifyAll()方法来唤醒线程    |
  | 锁的对象       | 锁对象               | 锁线程                                                       |

  

# 并发包

## CountDownLatch

主线程遇到CountDownLatch阻塞在那，要等待CountDownLatch里的所有线程都执行完毕，主线程才能继续执行。需要注意的是CountDownLatch创建的线程数和每个线程里countDown的总次数需要和初始化。[参考链接](https://blog.csdn.net/qq812908087/article/details/81112188)

1. 主线程等待子线程结束

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class test1 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(10);

        for (int i=0; i<10; i++) {//i<10等3秒多一点，i<9等10秒多一点
            new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println(Thread.currentThread().getName() + " 运行");
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        latch.countDown();
                    }
                }
            }).start();
        }

        System.out.println("等待子线程运行结束");
        latch.await(10, TimeUnit.SECONDS);//如果等10秒时候还没返回，那就自己结束
        System.out.println("子线程运行结束");
    }
}
```

2. 子线程等待主线程处理完毕开始处理，子线程处理完毕后，主线程输出

```java
import java.util.concurrent.CountDownLatch;

public class test2 {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(1);
        CountDownLatch await = new CountDownLatch(5);

        for (int i=0; i< 5; i++) {
            new Thread(()->{
                try {
                    countDownLatch.await();//主线程计数器上等待
                    System.out.println("子线程" +Thread.currentThread().getName()+ "处理自己事情");
                    Thread.sleep(1000);
                    await.countDown();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }

        System.out.println("主线程处理自己事情");
        Thread.sleep(3000);
        countDownLatch.countDown();//主线程计数器降为0,开始执行子线程
        System.out.println("主线程处理结束");
        await.await();//子线程计数器上等待
        System.out.println("子线程处理完毕啦");
    }
}
```

- CyclicBarrier

  是循环栅栏，意思是多个线程相互阻塞，只有多个线程都达到了栅栏时候，才能同时执行后续的逻辑。 

- Semaphore

  信号量，就是用来控制访问有限资源的线程数量，线程要访问资源首先要获得许可证，一个许可证对应一个资源，如果资源不足了，线程就要等待，如果其他线程释放了一个资源（许可证），那么信号量就通知等待的一个线程，分配给它一个许可证。

- Exchanger

  理解其意思就行，就是线程间的数据交换。是怎么交换呢？实际上是共用一个Exchange的多个线程，在全部都到达栅栏的时候才可以进行数据的交换，否则的话先到达的线程只能等待其他的线程达到栅栏。那怎么才算到达栅栏呢，就是线程内部调用exchanger.exchange方法。然后这个方法的返回值就是其他线程交换的数据，参数就是当前线程想要交还给其他线程的数据。

# 并发编程实战

## 第一章

```java
public class test1 {
    private int value;
    public void addValue() {
        value++;
    }

    public static void main(String[] args) throws InterruptedException {
        var test1=new test1();
        final var countDownLatch=new CountDownLatch(10);
        for(int i=0;i<10;i++){
            new Thread(()->{
                for(int j=0;j<10000;j++) {
                    test1.addValue();
                }
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
        System.out.println("最后结果:"+test1.value);
    }
}
```

最后结果:小于10000

```java
public class test1 {
    private int value;
    public synchronized void addValue() {
        value++;
    }

    public static void main(String[] args) throws InterruptedException {
        var test1=new test1();
        final var countDownLatch=new CountDownLatch(10);
        for(int i=0;i<10;i++){
            new Thread(()->{
                for(int j=0;j<10000;j++) {
                    test1.addValue();
                }
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
        System.out.println("最后结果:"+test1.value);
    }
}
```

最后结果:100000

## 第二章

- “无状态对象一定是线程安全的。”

```java
public class StatelessFactorizer {
    public void service(int x){
        Integer i=x;
        Integer[] factors=factor(i);
        System.out.println(x+":"+factors[0]+","+factors[1]);
    }

    public Integer[] factor(int x){
        return new Integer[]{x/2,x%2};
    }

    public static void main(String[] args) throws InterruptedException {
        final var countDownLatch=new CountDownLatch(10);
        StatelessFactorizer sf=new StatelessFactorizer();
        for(int i=0;i<10;i++){
            new Thread(()->{
                for(int j=100;j<200;j++) {
                   sf.service(j);
                }
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
    }
}
```

- 希望加一个统计函数调用次数的统计量，变的线程不安全，由数据竞争导致

```java
public class StatelessFactorizer {
    private long count=0;
    public long getCount(){
        return count;
    }
    public void service(int x){
        Integer i=x;
        Integer[] factors=factor(i);
        count++;
        //有这一句会使得count极为接近真实值
        System.out.println(x+":"+factors[0]+","+factors[1]);
    }

    public Integer[] factor(int x){
        return new Integer[]{x/2,x%2};
    }

    public static void main(String[] args) throws InterruptedException {
        final var countDownLatch=new CountDownLatch(10);
        StatelessFactorizer sf=new StatelessFactorizer();
        for(int i=0;i<10;i++){
            new Thread(()->{
                for(int j=0;j<10000;j++) {
                   sf.service(j);
                }
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
        System.out.println("调用次数:"+sf.count);
    }
}
```

- 通过引入一个原子变量来确保线程安全

```java
public class CountingFactorizer {
    private final AtomicLong count=new AtomicLong(0);

    public void service(int x){
        Integer i=x;
        Integer[] factors=factor(i);
        count.incrementAndGet();
        //System.out.println(x+":"+factors[0]+","+factors[1]);
    }

    public Integer[] factor(int x){
        return new Integer[]{x/2,x%2};
    }

    public static void main(String[] args) throws InterruptedException {
        final var countDownLatch=new CountDownLatch(10);
        final var cf=new CountingFactorizer();
        for(int i=0;i<10;i++){
            new Thread(()->{
                for(int j=0;j<10000;j++) {
                    cf.service(j);
                }
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
        System.out.println("调用次数:"+cf.count.get());
    }
}
```

- 延迟初始化中竞态条件导致的线程不安去

```java
public class LazyInitRace {
    private Object instance=null;
    public Object getInstance() throws InterruptedException {
        Thread.sleep(200);
        if(instance==null) {
            instance = new Object();
        }
        return instance;
    }

    public static void main(String[] args) throws InterruptedException {
        final var countDownLatch=new CountDownLatch(10);
        LazyInitRace lazyInitRace=new LazyInitRace();
        for(int i=0;i<50;i++){
            new Thread(()->{
                try {
                    System.out.println(lazyInitRace.getInstance()==lazyInitRace.getInstance());
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
    }
}
```

- 对这个服务做缓存，如果已经上次计算过，就直接获取，尽管对set方法的每次调用都是原子的，但仍然无法同时更新lastNumber和lastFactors

```java
public class UnsafeCachingFactorizer {
    private final AtomicReference<Integer> lastNumber = new AtomicReference<>();
    private final AtomicReference<Integer[]> lastFactors = new AtomicReference<>();

    public void service(int x) {
        Integer i = x;
        if (lastNumber.equals(x)) {
            System.out.println(x + ":" + lastFactors.get()[0] + lastFactors.get()[1]);
        } else {
            Integer[] factors = factor(i);
            lastNumber.set(x);
            lastFactors.set(factors);
            System.out.println(x + ":" + factors[0] + "," + factors[1]);
        }
    }

    public Integer[] factor(int x) {
        return new Integer[]{x / 2, x % 2};
    }
}
```

- 解决这个问题的办法就是对方法上锁

```java
public class UnsafeCachingFactorizer {
    private final Integer lastNumber;
    private final Integer[] lastFactors;

    public synchronized void service(int x) {
        Integer i = x;
        if (lastNumber.equals(x)) {
            System.out.println(x + ":" + lastFactors.get()[0] + lastFactors.get()[1]);
        } else {
            Integer[] factors = factor(i);
            lastNumber.set(x);
            lastFactors.set(factors);
            System.out.println(x + ":" + factors[0] + "," + factors[1]);
        }
    }

    public Integer[] factor(int x) {
        return new Integer[]{x / 2, x % 2};
    }
}
```

- 锁同步,下面的代码实现了锁的重入，如果不能重入，那么代码会死锁，但是代码实际未发生死锁，但是要注意的是当重写父类中的同步方法，如果想要达到同步的效果重写方法也必须是同步化的

```java
public class Widget extends SuperWidget{
    @Override
    public synchronized void doSomething(){
        System.out.println("hello");
        super.doSomething();
    }

    public static void main(String[] args) {
        new Widget().doSomething();
    }
}

class SuperWidget{
    public synchronized void doSomething(){
        System.out.println("hello supper");
    }
}
```

- 线程调用了一个对象的同步方法，那么它也可以调用对象的另外一个同步方法

```java
public class InstanceSychronized {
    public synchronized void fun1() {
        System.out.println("我是一号同步方法");
        this.fun2();//调用二号同步方法
    }

    //同步方法2
    public synchronized void fun2() {
        System.out.println("我是二号同步方法");
        this.fun3();//调用三号同步方法
    }

    //同步方法3
    public synchronized void fun3() {
        System.out.println("我是三号同步方法");
    }

    public static void main(String[] args) {
        var is = new InstanceSychronized();
        new Thread(() -> {
            is.fun1();
        }).start();
    }
}
```

- 活跃性与性能

```java
public class CachedFactorizer {
    private Integer lastNumber;
    private Integer[] lastFactors;
    private long hits;
    private long cacheHints;
    public synchronized long getHits(){
        return hits;
    }
    public synchronized double getCacheHitRatio(){
        return (double)cacheHints/(double)hits;
    }
    public void service(int x){
        Integer i=x;
        Integer[] factors=null;
      //负责保护判断是否只需返回缓存结果的“先检查后执行”操作序列
        synchronized (this){
            ++hits;
            if(i.equals(lastNumber)){
                ++cacheHints;
                factors=lastFactors.clone();
            }
        }
        if(factors==null){
            factors=factor(i);//耗时操作，尽量不要同步
          //负责确保对缓存的数值和因数分解结果进行同步更新
            synchronized (this){
                lastNumber=i;
                lastFactors=factors.clone();
            }
        }
    }

    private Integer[] factor(Integer x) {
        return new Integer[]{x/2,x%2};
    }
}
```

# 其它

一个JVM 启动之后，自己会启动一些线程

## 必备五线程

- Attach Listener ：线程是负责接收到外部的命令，而对该命令进行执行并且把结果返回给发送者。通常我们会用一些命令去要求jvm给我们一些反馈信息，如：java -version、jmap、jstack等等。如果该线程在jvm启动的时候没有初始化，那么，则会在用户第一次执行jvm命令时，得到启动。
- signal dispather： 前面我们提到第一个Attach Listener线程的职责是接收外部jvm命令，当命令接收成功后，会交给signal dispather线程去进行分发到各个不同的模块处理命令，并且返回处理结果。signal dispather线程也是在第一次接收外部jvm命令时，进行初始化工作。
- Finalizer：  用来执行所有用户Finalizer 方法的线程。
- Reference Handler ：它主要用于处理引用对象本身（软引用、弱引用、虚引用）的垃圾回收问题。
- Common-Cleaner：
- Monitor Ctrl-Break：监控ctrl-c
- Main：主线程

## RMI线程

- ***RMI TCP Connection(2)-127.0.0.1***
- ***RMI Scheduler(0)***
- ***RMI TCP Connection(1)-127.0.0.1***
- ***RMI TCP Accept-0***

一个Tomcat启动以后，后台一些常用线程的作用

- 名字里带有 Catalina-utility的是Tomcat中的工具线程，主要是干杂活，比如在后台定期检查Session是否过期、定期检查Web应用是否更新（热部署热加载）、检查异步Servlet的连接是否过期等等。
- 名字里带有Acceptor的线程负责接收浏览器的连接请求。
- 名字里带有Poller的线程，其实内部是个Selector，负责侦测IO事件。
- 名字里带有Catalina-exec的是工作线程，负责处理请求。

# 并发编程常见业务场景

## 定时任务

使用一个线程执行定时任务

```java
public class SheduledTask {
    public static void init() {
        new Thread(() -> {
            while (true) {
                try {
                    System.out.println("下载文件");
                    Thread.sleep(1000 * 3);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
    public static void main(String[] args) {
        SheduledTask.init();
    }
}
//优点：这种定时任务非常简单，学习成本低，容易入手，对于那些简单的周期性任务，是个不错的选择。
//缺点：不支持指定某个时间点执行任务，不支持延迟执行等操作，功能过于单一，无法应对一些较为复杂的场景。
```

## 监听器

使用一个线程监听数据库数据并同步，而且使用开关控制

```java
@Service
public CanalService {
    private volatile boolean running = false;//注意这个值
    private Thread thread;

    @Autowired
    private CanalConnector canalConnector;

    public void handle() {
        //连接canal
        while(running) {
           //业务处理
        }
    }

    public void start() {
       thread = new Thread(this::handle, "name");
       running = true;
       thread.start();
    }

    public void stop() {
       if(!running) {
          return;
       }
       running = false;
    }
}
```

```java
public class CanalConfig {
    @Autowired
    private CanalService canalService;

    @ApolloConfigChangeListener
    public void change(ConfigChangeEvent event) {
        String value = event.getChange("test.canal.enable").getNewValue();
        if(BooleanUtils.toBoolean(value)) {
            canalService.start();
        } else {
            canalService.stop();
        }
    }
}
```

## 收集日志

阻塞队列平衡日志生产与消费速度，使用单线程接收登录日志

```java
@Component
public class LoginLogQueue {
    private static final int QUEUE_MAX_SIZE    = 1000;

    private BlockingQueueblockingQueue queue = new LinkedBlockingQueue<>(QUEUE_MAX_SIZE);

    //生成消息
    public boolean push(LoginLog loginLog) {
        return this.queue.add(loginLog);
    } 

    //消费消息
    public LoginLog poll() {
        LoginLog loginLog = null;
        try {
            loginLog = this.queue.take();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return result;
    }
}
```

```java
@Service
public class LoginSerivce {

    @Autowired
    private LoginLogQueue loginLogQueue;

    public int login(UserInfo userInfo) {
        //业务处理
        LoginLog loginLog = convert(userInfo);
        loginLogQueue.push(loginLog);
    }  
}
```

```java
Service
public class LoginInfoConsumer {
    @Autowired
    private LoginLogQueue queue;

    @PostConstruct
    public voit init {
       new Thread(() -> {
          while (true) {
              LoginLog loginLog = queue.take();
              //写入数据库
          }
        }).start();
    }
}
```

## excel导入

导入excel以后流程很长

```java
supplierList.parallelStream().forEach(x -> importSupplier(x));
```

## 并行化接口

多个接口调用而又没有互相依赖

```java
public UserInfo getUserInfo(Long id) throws InterruptedException, ExecutionException {
    final UserInfo userInfo = new UserInfo();
    CompletableFuture userFuture = CompletableFuture.supplyAsync(() -> {
        getRemoteUserAndFill(id, userInfo);
        return Boolean.TRUE;
    }, executor);

    CompletableFuture bonusFuture = CompletableFuture.supplyAsync(() -> {
        getRemoteBonusAndFill(id, userInfo);
        return Boolean.TRUE;
    }, executor);

    CompletableFuture growthFuture = CompletableFuture.supplyAsync(() -> {
        getRemoteGrowthAndFill(id, userInfo);
        return Boolean.TRUE;
    }, executor);
    CompletableFuture.allOf(userFuture, bonusFuture, growthFuture).join();

    userFuture.get();
    bonusFuture.get();
    growthFuture.get();
    return userInfo;
}
```

## 获取用户上下文

```java
public class RequestHolder {

    private static final ThreadLocal<SysUser> userHolder = new ThreadLocal<SysUser>();


    public static void setUser(SysUser sysUser) {
        userHolder.set(sysUser);
    }

    public static SysUser getUser() {
        return userHolder.get();
    }

    public static void remove() {
        userHolder.remove();
    }
}
```

```java
// 1、拦截器获取用户信息
// 2、记录到ThreadLocal中
@Component
public class AuthenticationHandlerInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        log.debug("进入拦截器,URL:{}", request.getServletPath());

        // 加入用户全局信息
        RequestHolder.setUser(userInfo);
        return true;
    }
    ....
    ....
}

// 3、使用时通过get()方法获取值
@RestController
@RequestMapping
public class Controller {
   .....
    @GetMapping("/test")
    public String test() {
        // 从ThreadLocal获取数据
        RequestHolder.getUserId();
        return "访问成功";
    }
   .....
}
```

## 传递参数

MDC中传递参数

> 在new一个Thread的时候会调用Thread的init方法，该方法中如果parent线程的inheritableThreadLocals不是null的话，就会用createInheritedMap方法，用parent的inheritableThreadLocals中的元素构造一个新的ThreadLocalMap。
>
> 注意：该操作只在线程初始化的时候进行，所以在该线程初始化之后，parent线程对parent线程自己的inheritableThreadLocals变量的操作不会影响到当前线程的inheritableThreadLocals了，因为已经不是同一个map了。
> MDC就是利用这个InheritableThreadLocal 把父线程的context带到子线程中，把上下文传递到子线程中通过日志输出，把一次完整的请求串联起来。

## 模拟高并发

```java
public static void concurrenceTest() {
    /**
     * 模拟高并发情况代码
     */
    final AtomicInteger atomicInteger = new AtomicInteger(0);
    final CountDownLatch countDownLatch = new CountDownLatch(1000); // 相当于计数器，当所有都准备好了，再一起执行，模仿多并发，保证并发量
    final CountDownLatch countDownLatch2 = new CountDownLatch(1000); // 保证所有线程执行完了再打印atomicInteger的值
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    try {
        for (int i = 0; i < 1000; i++) {
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    try {
                        countDownLatch.await(); //一直阻塞当前线程，直到计时器的值为0,保证同时并发
                    } catch (InterruptedException e) {
                        log.error(e.getMessage(),e);
                    }
                    //每个线程增加1000次，每次加1
                    for (int j = 0; j < 1000; j++) {
                        atomicInteger.incrementAndGet();
                    }
                    countDownLatch2.countDown();
                }
            });
            countDownLatch.countDown();
        }

        countDownLatch2.await();// 保证所有线程执行完
        executorService.shutdown();
    } catch (Exception e){
        log.error(e.getMessage(),e);
    }
}
```

## 处理MQ消息

消息积压时

```java
@Service
public class MyConsumerService {
    @Autowired
    private Executor messageExecutor;

    @KafkaListener(id="test",topics={"topic-test"})
    public void listen(String message){
        System.out.println("收到消息：" + message);
        messageExecutor.submit(new MyWork(message);
    }
}

 public class MyWork implements Runnable {
    private String message;

    public MyWork(String message) {
       this.message = message;
    }

    @Override
    public void run() {
        System.out.println(message);
    }
}                              
```

## 统计数量

在多线程的场景中，有时候需要统计数量，比如：用多线程导入供应商数据时，统计导入成功的供应商数有多少。如果这时候用count++统计次数，最终的结果可能会不准。因为count++并非原子操作，如果多个线程同时执行该操作，则统计的次数，可能会出现异常。为了解决这个问题，就需要使用`concurent`的`atomic`包下面的类，比如：`AtomicInteger`、`AtomicLong`等

```java
@Servcie
public class ImportSupplierService {
  private static AtomicInteger count = new AtomicInteger(0);

  public int importSupplier(List<SupplierInfo> supplierList) {
       if(CollectionUtils.isEmpty(supplierList)) {
           return 0;
       }

       supplierList.parallelStream().forEach(x -> {
           try {
             importSupplier(x);
             count.addAndGet(1);
           } catch(Exception e) {
              log.error(e.getMessage(),e);
           }
       );

      return count.get();
  }    
}
```

## 延迟定时任务

如果用户下单后，超过30分钟还未完成支付，则系统自动将该订单取消。

```java
public class ScheduleExecutorTest {

    public static void main(String[] args) {
        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);
        scheduledExecutorService.scheduleAtFixedRate(() -> {
            System.out.println("doSomething");
        },1000,1000, TimeUnit.MILLISECONDS);
    }
}
```

# 并发编程常见的问题

- SimpleDateFormat线程不安全

- 单例模式双重检查锁问题

- volatile的原子性问题

- 死锁问题

  > ```java
  > public class Account {
  > public Integer id;
  > public String name;
  > public Integer balance=100;
  > public AtomicInteger atomicInteger=new AtomicInteger(100);
  > 
  > public Account(int id, String name) {
  > this.id=id;
  > this.name=name;
  > }
  > 
  > public void transfer(Account target, int amount) throws InterruptedException{
  > synchronized (this){//锁住转出账户
  > System.out.println(Thread.currentThread().getName()+"begin");
  > Thread.sleep(2000);
  > synchronized (target){//锁住转入账户
  >     this.balance-=amount;
  >     target.balance+=amount;
  >     System.out.println(Thread.currentThread().getName()+"end");
  > }
  > }
  > }
  > 
  > public static void main(String[] args) {
  > Account zhangshan=new Account(1, "zhangshan");
  > Account lisi=new Account(2, "lisi");
  > new Thread(()->{
  > try {
  >     zhangshan.transfer(lisi,50);
  > } catch (InterruptedException e) {
  >     e.printStackTrace();
  > }
  > }, "t1").start();
  > new Thread(()->{
  > try {
  >     lisi.transfer(zhangshan,50);
  > } catch (InterruptedException e) {
  >     e.printStackTrace();
  > }
  > }, "t2").start();
  > }
  > }
  > 
  > t2begin
  > t1begin
  > ```
  >
  > **jps** 找到进程号
  >
  > **jstack 进程ID**  找到信息 
  >
  > 或者使用**jconsole->线程->检测死锁**
  >
  > **为什么会产生死锁？**
  >
  > 互斥、占有且等待、不可抢占、循环等待
  >
  > **怎么破坏条件解决死锁？**
  >
  > 1. 不使用互斥锁，使用原子操作（破坏互斥）
  >
  >    ```java
  >    public void transfer(Account target, int amount) throws InterruptedException{
  >         synchronized (this){//锁住转出账户
  >                 this.atomicBalance.addAndGet(-amount);
  >                 target.atomicBalance.addAndGet(amount);
  >         }
  >     }
  >    //ThreadLocal CAS？
  >    ```
  >
  > 2. 一次性申请（破坏占有且等待）
  >
  >    ```java
  >    将两个资源放在一个list一次申请
  >    ```
  >
  > 3. 设置超时（破坏不可抢占）
  >
  >    ```java
  >    lock.trylock
  >    ```
  >
  > 4. 排序（破坏循环等待）
  >
  >    ```java
  >    将两个账号id排序再按顺序申请
  >    ```
  >
  > 银行家算法保证避免死锁：**分配资源之前，判断系统是否安全，如果安全才会进行资源分配。**

- HashMap的问题

- 使用默认线程池的问题

  > - `newFixedThreadPool`：允许请求的队列长度是Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。
  >
  > - `newSingleThreadExecutor`：允许请求的队列长度是Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。
  >
  > - `newCachedThreadPool`：允许创建的线程数是Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。
  >
  >   顺便说一下，如果是一些低并发场景，使用`Executors`类创建线程池也未尝不可，也不能完全一棍子打死。在这些低并发场景下，很难出现OOM问题，所以我们需要根据实际业务场景选择。

- @Async注解的陷阱

  > 开发spring的大神们，为了简化这类异步操作，已经帮我们把异步功能封装好了。spring中提供了`@Async`注解，我们可以通过它即可开启异步功能，使用起来非常方便。
  >
  > 用@Async注解开启的异步功能，会调用`AsyncExecutionAspectSupport`类的`doSubmit`方法。
  >
  > 使用@Async注解开启的异步功能，默认情况下，每次都会创建一个新线程。
  >
  > 如果在高并发的场景下，可能会产生大量的线程，从而导致OOM问题。

- 自旋锁浪费cpu资源

  > 如果在高并发的情况下，compareAndSwapInt会很大概率失败，因此导致了此处cpu不断的自旋，这样会严重浪费cpu资源。
  >
  > 那么，如果解决这个问题呢？
  >
  > 答：使用`LockSupport`类的`parkNanos`方法。
  >
  > 具体代码如下：
  >
  > ```java
  > private boolean compareAndSwapInt2(Object var1, long var2, int var4, int var5) {
  >  if(this.compareAndSwapInt(var1,var2,var4, var5)) {
  >       return true;
  >   } else {
  >       LockSupport.parkNanos(10);
  >       return false;
  >   }
  > }
  > ```

- ThreadLocal用完没清空

  ```java
  private static final ThreadLocal<Integer> currentUser = ThreadLocal.withInitial(() -> null);
  
  @GetMapping("wrong")
  public Map wrong(@RequestParam("userId") Integer userId) {
      //设置用户信息之前先查询一次ThreadLocal中的用户信息
      String before  = Thread.currentThread().getName() + ":" + currentUser.get();
      //设置用户信息到ThreadLocal
      currentUser.set(userId);
      //设置用户信息之后再查询一次ThreadLocal中的用户信息
      String after  = Thread.currentThread().getName() + ":" + currentUser.get();
      //汇总输出两次查询结果
      Map result = new HashMap();
      result.put("before", before);
      result.put("after", after);
      return result;
  }
  
  //按理说，在设置用户信息之前第一次获取的值始终应该是 null，但我们要意识到，程序运行在 Tomcat 中，执行程序的线程是 Tomcat 的工作线程，而 Tomcat 的工作线程是基于线程池的。顾名思义，线程池会重用固定的几个线程，一旦线程重用，那么很可能首次从 ThreadLocal 获取的值是之前其他用户的请求遗留的值。这时，ThreadLocal 中的用户信息就是其他用户的信息。为了更快地重现这个问题，我在配置文件中设置一下 Tomcat 的参数，把工作线程池最大线程数设置为 1，这样始终是同一个线程在处理请求：
  server.tomcat.max-threads=1
  //修正这段代码的方案是，在代码的 finally 代码块中，显式清除 ThreadLocal 中的数据。这样一来，新的请求过来即使使用了之前的线程也不会获取到错误的用户信息了  
  ```

- 不要以为使用了并发工具就可以解决一切线程安全问题，期望通过把线程不安全的类替换为线程安全的类来一键解决问题。比如，认为使用了 ConcurrentHashMap 就可以解决线程安全问题，没对复合逻辑加锁导致业务逻辑错误。如果你希望在一整段业务逻辑中，对容器的操作都保持整体一致性的话，需要加锁处理。

- 没有充分了解并发工具的特性，还是按照老方式使用新工具导致无法发挥其性能。比如，使用了 ConcurrentHashMap，但没有充分利用其提供的基于 CAS 安全的方法，还是使用锁的方式来实现逻辑。比如computeIfAbsent、putIfAbsent等方法

- 没有了解清楚工具的适用场景，在不合适的场景下使用了错误的工具导致性能更差。比如，没有理解 CopyOnWriteArrayList 的适用场景，把它用在了读写均衡或者大量写操作的场景下，导致性能问题。对于这种场景，你可以考虑是用普通的 List。

# 锁

| 序号 | 锁名称   | 应用                                                         |
| :--- | :------- | :----------------------------------------------------------- |
| 1    | 乐观锁   | CAS                                                          |
| 2    | 悲观锁   | synchronized、vector、hashtable                              |
| 3    | 自旋锁   | CAS                                                          |
| 4    | 可重入锁 | synchronized、Reentrantlock、Lock                            |
| 5    | 读写锁   | ReentrantReadWriteLock，CopyOnWriteArrayList、CopyOnWriteArraySet |
| 6    | 公平锁   | Reentrantlock(true)                                          |
| 7    | 非公平锁 | synchronized、reentrantlock(false)                           |
| 8    | 共享锁   | ReentrantReadWriteLock中读锁                                 |
| 9    | 独占锁   | synchronized、vector、hashtable、ReentrantReadWriteLock中写锁 |
| 10   | 重量级锁 | synchronized                                                 |
| 11   | 轻量级锁 | 锁优化技术                                                   |
| 12   | 偏向锁   | 锁优化技术                                                   |
| 13   | 分段锁   | concurrentHashMap                                            |
| 14   | 互斥锁   | synchronized                                                 |
| 15   | 同步锁   | synchronized                                                 |
| 16   | 死锁     | 相互请求对方的资源                                           |
| 17   | 锁粗化   | 锁优化技术                                                   |
| 18   | 锁消除   | 锁优化技术                                                   |

# 面试题

https://mp.weixin.qq.com/s/asVIrMWDJkBPojdRRZnKzQ

# 线程代码

## 使用奇偶线程交替打印奇偶数

```java
//使用监视器交替打印数据
public class TwoPrint {
    private static volatile int i = 1;
    public static void main(String[] args) {
        final Object obj = new Object();

        Runnable runnable = () -> {
            synchronized (obj) {
                for (; i < 10; ) {
                    System.out.println(Thread.currentThread().getName() + " " + (i++));
                    try {
                        obj.notifyAll();
                        obj.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                obj.notifyAll();//保证两个线程都能退出
            }
        };

        Thread t1 = new Thread(runnable, "线程1 ");
        Thread t2 = new Thread(runnable, "线程2 ");
        t2.start();
        t1.start();
    }
}

//使用ReentrantLock+Condition交替打印
public class TwoPrint {
    private static volatile int i = 1;
    public static void main(String[] args) {
        Lock lock=new ReentrantLock();
        Condition condition = lock.newCondition();

        Runnable runnable=()-> {
            lock.lock();
            try {
                for (; i < 10; ) {
                    System.out.println(Thread.currentThread().getName() + " " + (i++));
                    condition.signalAll();
                    try {
                        condition.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                condition.signalAll();//保证正常结束
            }
            finally {
                lock.unlock();
            }
        };

        Thread t1 = new Thread(runnable, "线程1 ");
        Thread t2 = new Thread(runnable, "线程2 ");
        t2.start();
        t1.start();
    }
}
```

# JUC及其源码分析

[思否多线程源码分析](https://segmentfault.com/a/1190000015558984)

## start分析

> new Thread().start();
>
> 执行的是Thread中的public synchronized void start()
>
> 调用private native void start0();
>
> 调用thread.cpp中的Thread::start对应方法

## 管程

> JVM同步是基于进入和退出监视器对象（Monitor,管程对象）来实现的，每个对象实例都会有一个Monitor对象。
>
> Monitor对象会和Java对象一同创建并销毁，底层由c++来实现
>
> - 如果一个java对象被某个线程锁住，则该java对象的Mark Word字段中LocakWord指向monitor的起始地址
> - Monitor的Owner字段会存放拥有相关联对象锁的线程的id

## 守护线程

> 垃圾收集线程等是守护线程，用户线程全部执行完以后守护线程会自动结束
>
> 设置守护线程要在start方法之前设置

## 线程中断协商机制

> - 一个线程不应该由其他线程来强制中断或停止,而是应该由线程自己自行停止,所以,Thread.stop、Thread.suspend、Thread. resume都已经被废弃了。在Java中没有办法立即停止一条线程,然而停止线程却显得尤为重要,如取消一个耗时操作。因此,Java提供了一种用于停止线程的机制 — 中断
>
> - 中断只是一种协作机制,Java没有给中断增加任何语法,中断的过程完全需要程序员自己实现。若要中断一个线程,你需要手动调用该线程的interrupt方法,该方法也仅仅是将线程对象的中断标识设为true
>
> - 每个线程对象中都有一个标识,用于标识线程是否被中断;该标识位为true表示中断,为false表示未中断;通过调用线程对象的interrupt方法将线程的标识位设为true;可以在别的线程中调用,也可以在自己的线程中调用
>
> - 如何中断一个线程？
>
>   - 通过volatile变量进行
>   - 通过AtomicBoolean
>   - 通过Thread类自带的中断API方法实现
>
>   ```java
>   public class InterruptDemo{
>      static volatile boolean isStop = false;
>      static AtomicBoolean atomicBoolean = new AtomicBoolean(false);
>   
>      /**通过Thread类自带的中断API方法实现*/
>      public static void m3(){
>      Thread t1 = new Thread(() -> {
>          while (true) {
>              if (Thread.currentThread().isInterrupted()) {
>                  System.out.println("-----isInterrupted() = true,程序结束。");
>                  break;
>              }
>              System.out.println("------hello Interrupt");
>          }
>      }, "t1");
>      t1.start();
>   
>      try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
>   
>      new Thread(() -> {
>          t1.interrupt();//修改t1线程的中断标志位为true
>      },"t2").start();
>   }
>   
>      /**
>       * 通过AtomicBoolean
>       */
>      public static void m2(){
>          new Thread(() -> {
>              while(true)
>              {
>                  if(atomicBoolean.get())
>                  {
>                      System.out.println("-----atomicBoolean.get() = true,程序结束。");
>                      break;
>                  }
>                  System.out.println("------hello atomicBoolean");
>              }
>          },"t1").start();
>   
>          //暂停几秒钟线程
>          try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
>   
>          new Thread(() -> {
>              atomicBoolean.set(true);
>          },"t2").start();
>      }
>   
>      /**
>       * 通过一个volatile变量实现
>       */
>      public static void m1(){
>          new Thread(() -> {
>              while(true)
>              {
>                  if(isStop)
>                  {
>                      System.out.println("-----isStop = true,程序结束。");
>                      break;
>                  }
>                  System.out.println("------hello isStop");
>              }
>          },"t1").start();
>   
>          //暂停几秒钟线程
>          try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
>   
>          new Thread(() -> {
>              isStop = true;
>          },"t2").start();
>      }
>   }
>   ```
>
> - interrupt方法只是设置中断标志位为true,线程未必会立刻中断
>
> - 中断不活动的线程不会有任何影响，标志位不会变
>
> - 打断一个wait、join、sleep线程，中断状态将被清除并且抛出一个异常，要中断就在异常中再interrupt一次
>
>   >此时处理InterruptedException异常有以下几种方法
>   >
>   >1.不要生吞此异常；
>   >
>   >2.如果可以处理此异常，完成清理工作之后退出；
>   >
>   >3.不处理此异常，不继续执行任务，重新抛出：**Thread.currentThread().interrupt()**
>   >
>   >4.不处理此异常，继续执行任务，捕捉到异常之后恢复中断标记（交由后续程序检查中断）。
>
> - interrupted静态方法，返回当前线程中断状态并清除线程的中断状态
>
> - isInterrupted实例方法，与interrupted静态方法调用的是同一个native方法，静态方法清理了中断标志，实例方法不清理

## LocalSupport

> Object: wait notify
>
> ```java
> 锁X=new Object();
> new Thread(()->synchronized(锁X){wait}).start();
> new Thread(()->synchronized(锁X){notify}).start();
> // wait、notify必须在synchronized同步代码块中，否则将抛出IllegalMonitorStateException
> // notify必须在wait方法之后，否则调用wait的会一直等待,无法唤醒
> ```
>
> JUC Codition: await signal
>
> ```java
> Lock lock=new ReentrantLock();
> Condition condition=lock.newCondition();
> new Thread(()->{lock();condition.await();unlock()}).start();
> new Thread(()->{lock();condition.signal();unlock()}).start();
> // await、signal必须在lock中，否则将抛出IllegalMonitorStateException
> //signal必须在await方法之后，否则调用await的会一直等待,无法唤醒
> ```
>
> LockSupport: park unpark
>
> ```java
> new Thread(()->{LockSupport.park()}).start();
> new Thread(()->{LockSupport.unpark()}).start();
> //不必有锁
> //不必按顺序
> //Locaksupport方法都是静态方法
> //通行证最多发放一次，多次发放不累积
> ```

## Java内存模型

> - CPU->寄存器->CPU缓存->内存
>
> - JVM规范中试图定义一中Java内存模型（JMM），来屏蔽掉各种硬件和操作系统的访问差异，以实现让java在各种平台下都能达到一致的内存访问效果。
>
> - 通过JMM来实现线程和主内存之间的抽象关系
>
> - 三大特性：可见性、原子性、有序性

## volatile内存语义

> 写一个volatile变量时，线程内存的值会立即刷新到主内存中
>
> 读一个volatile变量时，会把线程内存设置为无效重新去主内存读
>
> 总结：读时直接读主内存，写时立即刷新到主内存

## 内存屏障

> 其实就是一种JVM指令，java内存模型的重排序规则会要求Java编译器在生成JVM指令时插入特定的内存屏障指令，通过volatile实现内存模型的可见性和有序性。
>
> 内存屏障之前的的所有写操作都要回写到主内存
>
> 内存屏障之后的所有读操作都能获取到内存屏障之前的所有写操作的最新结果
>
> 总结：对一个volatile变量的写先行发生于任意后续对这个变量的读
>
> 读屏障：读指令前插入读屏障，工作内存或cpu高速缓存当中内容立刻失效
>
> 写屏障：写指令之后插入写屏障，强制把缓存区的数据刷回到内存
>
> | 屏障类型   | 示例                     | 说明                               |
> | ---------- | ------------------------ | ---------------------------------- |
> | loadload   | load1;loadload;load2     | 保证load1的读取操作在load2之前进行 |
> | storestore | store1;storestore;store2 | store2写开始时store1已经存入到主存 |
> | loadstore  | load1;loadstore;store2   | store2写时load的读操作已结束       |
> | storeload  | store1;storeload;load2   | load2读操作时store1写操作已结束    |
>
> 对于重排序的处理：
>
> 对于编译器的重排序，JMM会根据重排序规则，禁止特定类型的编译器重排序
>
> 对于处理器的重排序，Java编译器在生成指令序列的适当位置，插入内存屏障指令来禁止特定类型的的处理器排序
>
> 实际应用：
>
> 1. 单一变量赋值，复合赋值不可以（i++）
>
> 2. 状态标志
>
> 3. 开销较低的读写锁策略
>
>    ```java
>    volatile int value;
>    public int getValue(){
>      return value;//
>    }
>    public synchronized int inrease(){
>      value++
>    }
>    ```
>
> 4. DCL双端锁的发布
>
>    - 单例实现，避免重排序，二次判空时用来避免对象分配的重排序
>
> - 底层的volatile是怎么跟操作系统勾搭上的？
>
>   反编译以后会字节码会加入ACC_VOLATILE标志，字节码在生成机器码的时候发现是volatile会在相应的位置插入内存屏障

## CAS

> CAS是一条CPU的原子指令（cmpxchg指令），不会造成所谓的数据不一致的问题，Unsafe提供的CAS方法（比如compareAndSwapXXX）底层实现即为CPU指令cmpxchg
>
> 执行cmpxchg指令的时候，会判断当前系统是否为多核系统，如果是就给总线加锁，只有一个线程会对总线加锁成功，加锁成功之后会执行cas操作，也就是说cas的原子性实际上是cpu实现独占的。比起synchronized重量级锁，这里的排他时间要短很多，所以在多线程情况下性能会比较好。
>
> ```java
> getAndIncrement-> unsafe.getAndAddInt(this,valueoff,1) ->
> do {
>         v = getIntVolatile(o, offset);
> } while (!weakCompareAndSetInt(o, offset, v, v + delta));
> ```
>
> - JDK提供的CAS机制，在汇编层级会禁止变量两侧的指令优化，然后使用cmpxchg指令比较并更新变量值
>
> ```java
> class User{
>   String name;
>   Integer age;
> }
> AtomicReference<User> atomicReference=new AtomicReference<>();
> User user1=new User("张三",1);
> User user2=new User("李四",1);
> boolean b = atomicReference.compareAndSet(user1, user2);
> ```
>
> - CAS手写自旋锁
>
> ```java
> public class test1 {
>    AtomicReference<Thread> atomicReference=new AtomicReference<>();
>    public void lock(){
>        System.out.println(Thread.currentThread().getName()+" come in");
>        Thread thread = Thread.currentThread();
>        while(!atomicReference.compareAndSet(null, thread)){
> 
>        }
>    }
> 
>    public void unlock(){
>        Thread thread = Thread.currentThread();
>        atomicReference.compareAndSet(thread,null);
>        System.out.println(Thread.currentThread().getName()+" come out");
>    }
> 
>    public static void main(String[] args) throws IllegalAccessException, InterruptedException {
>        test1 test1 = new test1();
>        new Thread(()->{
>            test1.lock();
>            try {
>                TimeUnit.SECONDS.sleep(5);
>            } catch (InterruptedException e) {
>                e.printStackTrace();
>            }
>            test1.unlock();
>        },"A").start();
> 
>        TimeUnit.MILLISECONDS.sleep(500);
> 
>        new Thread(()->{
>            test1.lock();
>            test1.unlock();
>        },"B").start();
>    }
> }
> ```
>
> - 缺点
>
>   循环时间长带来的CPU空转，ABA问题
>
> ```java
> //使用版本号解决ABA问题
> public class AtomicStampedDemo {
>    public Integer id;
>    public String name;
> 
>    public AtomicStampedDemo(Integer id, String name) {
>        this.id = id;
>        this.name = name;
>    }
> 
>    public static void main(String[] args) {
>        AtomicStampedDemo a = new AtomicStampedDemo(1,"A");
>        AtomicStampedReference<AtomicStampedDemo> stampedReference=new AtomicStampedReference<>(a,1);
>        System.out.println(stampedReference.getReference()+" "+stampedReference.getStamp());
>        AtomicStampedDemo b = new AtomicStampedDemo(2, "B");
>        boolean x=stampedReference.compareAndSet(a,b,stampedReference.getStamp(),stampedReference.getStamp()+1);
>        System.out.println(x+" "+stampedReference.getReference()+" "+stampedReference.getStamp());
>    }
> }
> ```

## unSafe

> 是CAS系统的核心类，Java方法无法直接访问底层系统，需要通过本地方法来访问，unSafe相当于一个后门，基于该类可以直接操作特定内存数据。其内部方法可以像C的指针一样直接操作内存。其中的所有方法都是native的。
>
> - 变量valueOffset表示该变量在内存中的偏移地址

## 原子类

> 基本（Integer、Boolean、Long）、数组（Integer、Long、reference）、引用（reference、stampedReference、MarkableRference）、对象的属性修改（IntegerFiledUpdater、LongFiledUpdater、ReferenceFieldUpdater）、增强类（1.8以后DoubleAdder，LongAdder，Longaccumulator，DoubleAccumulator，吞吐量高代价是空间消耗更高）
>
> - stampedReference用来解决多次问题，markablereference解决一次性问题，可以理解为stampedReference的戳为true或false
>
> - FiledUpdater，只能更新public volatile修饰的字段
>
>   ```java
>   //没有原子类的时候采用如下方式更新字段
>   class count{
>     int money=0;
>     String other;
>     synchronized add{
>        money++;//这里要锁整个对象,other也不能操作
>     }
>   }
>   //有了原子类的时候
>   class count{
>     public volatile int money=0;
>     String other;
>    //只有一个线程会执行初始化
>     AtomicIntegerFieldUpdater<Account> fieldUpdater=AtomicIntegerFieldUpdater.newUpdater(Account.class,"money");
>      public void add(Account account){
>          fieldUpdater.getAndIncrement(account);
>      }
>   
>   }
>   //类似于做手术全麻与局麻的区别
>   ```
>
> - 热点数据计数，点赞数加加统计，不要求实时精确(50个现线程，每个线程100w次，总点赞数出来)
>
>   ```java
>   class Click {
>      int num = 0;
>   
>      public synchronized void clickBySynchronized() {
>          num++;
>      }
>   
>      AtomicLong atomicLong = new AtomicLong(0);
>   
>      public void clickByAtomicLong() {
>          atomicLong.getAndIncrement();
>      }
>   
>      LongAdder longAdder = new LongAdder();
>   
>      public void clickByLongAdder() {
>          longAdder.increment();
>      }
>   
>      LongAccumulator longAccumulator = new LongAccumulator((x, y) -> x + y, 0);
>   
>      public void clickByLongAccumulator() {
>          longAccumulator.accumulate(1);
>      }
>   }
>   
>   public class ClickLike {
>      public static final int _1w = 10000;
>      public static final int threadNumber = 50;
>   
>      public static void main(String[] args) throws InterruptedException {
>          Click click=new Click();
>          long startTime;
>          long endTime;
>          CountDownLatch countDownLatch1=new CountDownLatch(threadNumber);
>          CountDownLatch countDownLatch2=new CountDownLatch(threadNumber);
>          CountDownLatch countDownLatch3=new CountDownLatch(threadNumber);
>          CountDownLatch countDownLatch4=new CountDownLatch(threadNumber);
>          startTime=System.currentTimeMillis();
>          for(int i=1;i<=threadNumber;i++){
>              new Thread(()->{
>                  try {
>                      for(int j=1;j<=100*_1w;j++){
>                          click.clickBySynchronized();
>                      }
>                  }finally {
>                      countDownLatch1.countDown();
>                  }
>              },String.valueOf(i)).start();
>          }
>          countDownLatch1.await();
>          endTime=System.currentTimeMillis();
>          System.out.println("costTime:"+(endTime-startTime)+"毫秒,clickby sychronized:"+click.num);
>   
>          startTime=System.currentTimeMillis();
>          for(int i=1;i<=threadNumber;i++){
>              new Thread(()->{
>                  try {
>                      for(int j=1;j<=100*_1w;j++){
>                          click.clickByAtomicLong();
>                      }
>                  }finally {
>                      countDownLatch2.countDown();
>                  }
>              },String.valueOf(i)).start();
>          }
>          countDownLatch2.await();
>          endTime=System.currentTimeMillis();
>          System.out.println("costTime:"+(endTime-startTime)+"毫秒,clickby AtomicLong:"+click.atomicLong.get());
>   
>          startTime=System.currentTimeMillis();
>          for(int i=1;i<=threadNumber;i++){
>              new Thread(()->{
>                  try {
>                      for(int j=1;j<=100*_1w;j++){
>                          click.clickByLongAdder();
>                      }
>                  }finally {
>                      countDownLatch3.countDown();
>                  }
>              },String.valueOf(i)).start();
>          }
>          countDownLatch3.await();
>          endTime=System.currentTimeMillis();
>          System.out.println("costTime:"+(endTime-startTime)+"毫秒,clickby LongAdder:"+click.longAdder.sum());
>   
>          startTime=System.currentTimeMillis();
>          for(int i=1;i<=threadNumber;i++){
>              new Thread(()->{
>                  try {
>                      for(int j=1;j<=100*_1w;j++){
>                          click.clickByLongAccumulator();
>                      }
>                  }finally {
>                      countDownLatch4.countDown();
>                  }
>              },String.valueOf(i)).start();
>          }
>          countDownLatch4.await();
>          endTime=System.currentTimeMillis();
>          System.out.println("costTime:"+(endTime-startTime)+"毫秒,clickby LongAccumulator:"+click.longAccumulator.get());
>      }
>   }
>   //结果,可以看到性能有量级的差异
>   无锁：costTime:65毫秒,28090093
>   costTime:2674毫秒,clickby sychronized:50000000
>   costTime:1013毫秒,clickby AtomicLong:50000000
>   costTime:89毫秒,clickby LongAdder:50000000
>   costTime:84毫秒,clickby LongAccumulator:50000000
>   ```
>
> - LongAdder是Stripe64的子类
>
>   ```java
>   int base//单线程时在base上进行CAS
>   Cell[数组] //线程多时在各个槽上分段进行CAS 
>   //求和时，将base值加上所有Cell数组的和，以空间换时间的思想
>   ```
>
>   ![images](https://img-blog.csdnimg.cn/2021040618450399.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)
>
>   ![images](https://img-blog.csdnimg.cn/20210406185208783.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)
>
>   ![images](https://img-blog.csdnimg.cn/20210616223445769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RaODQ1MTk1NDg1,size_16,color_FFFFFF,t_70)
>
>   - 为啥高并发下sum的值不精确？
>
>     sum执行时,并没有限制对base和cells的更新(一句要命的话)。所以LongAdder不是强一致性,它是最终一致性的
>     首先,最终返回的sum局部变量,初始被赋值为base,而最终返回时,很可能base已经被更新了,而此时局部变量sum不会更新,造成不一致
>     其次,这里对cell的读取也无法保证是最后一次写入的值。所以,sum方法在没有并发的情况下,可以获得正确的结果

## AQS

## StampedLock

# 池化技术

TCP 是面向连接的基于字节流的协议

>- 面向连接，意味着连接需要先创建再使用，创建连接的三次握手有一定开销
>- 基于字节流，意味着字节是发送数据的最小单元，TCP 协议本身无法区分哪几个字节是完整的消息体，也无法感知是否有多个客户端在使用同一个 TCP 连接，TCP 只是一个读写数据的管道。

如果客户端 SDK 没有使用连接池，而直接是 TCP 连接，那么就需要考虑每次建立 TCP 连接的开销，并且因为 TCP 基于字节流，在多线程的情况下对同一连接进行复用，可能会产生线程安全问题。

先看一下涉及 TCP 连接的客户端 SDK，对外提供 API 的三种方式

>- 连接池和连接分离的 API：有一个 XXXPool 类负责连接池实现，先从其获得连接 XXXConnection，然后用获得的连接进行服务端请求，完成后使用者需要归还连接。通常，XXXPool 是线程安全的，可以并发获取和归还连接，而 XXXConnection 是非线程安全的。
>
>- 内部带有连接池的 API：对外提供一个 XXXClient 类，通过这个类可以直接进行服务端请求；这个类内部维护了连接池，SDK 使用者无需考虑连接的获取和归还问题。一般而言，XXXClient 是线程安全的。
>- 非连接池的 API：一般命名为 XXXConnection，以区分其是基于连接池还是单连接的，而不建议命名为 XXXClient 或直接是 XXX。直接连接方式的 API 基于单一连接，每次使用都需要创建和断开连接，性能一般，且通常不是线程安全的。

虽然上面提到了 SDK 一般的命名习惯，但不排除有一些客户端特立独行，因此在使用三方 SDK 时，一定要先查看官方文档了解其最佳实践，

- Jedis

```java
Jedis jedis = new Jedis("127.0.0.1", 6379);
new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        String result = jedis.get("a");
        if (!result.equals("1")) {
            log.warn("Expect a to be 1 but found {}", result);
            return;
        }
    }
}).start();
new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        String result = jedis.get("b");
        if (!result.equals("2")) {
            log.warn("Expect b to be 2 but found {}", result);
            return;
        }
    }
}).start();
TimeUnit.SECONDS.sleep(5);
//这段代码有时会有奇怪的现象发生，有的是读取 Key 为 b 的 Value 读取到了 1，有的是流非正常结束，还有的是连接关闭异常

//Jedis 继承了 BinaryJedis，BinaryJedis 中保存了单个 Client 的实例，Client 最终继承了 Connection，Connection 中保存了单个 Socket 的实例，和 Socket 对应的两个读写流。因此，一个 Jedis 对应一个 Socket 连接。

//BinaryClient 封装了各种 Redis 命令，其最终会调用基类 Connection 的方法，使用 Protocol 类发送命令。看一下 Protocol 类的 sendCommand 方法的源码，可以发现其发送命令时是直接操作 RedisOutputStream 写入字节。我们在多线程环境下复用 Jedis 对象，其实就是在复用 RedisOutputStream。如果多个线程在执行操作，那么既无法确保整条命令以一个原子操作写入 Socket，也无法确保写入后、读取前没有其他数据写到远端

//看到这里我们也可以理解了，为啥多线程情况下使用 Jedis 对象操作 Redis 会出现各种奇怪的问题。比如，写操作互相干扰，多条命令相互穿插的话，必然不是合法的 Redis 命令，那么 Redis 会关闭客户端连接，导致连接断开；又比如，线程 1 和 2 先后写入了 get a 和 get b 操作的请求，Redis 也返回了值 1 和 2，但是线程 2 先读取了数据 1 就会出现数据错乱的问题。修复方式是，使用 Jedis 提供的另一个线程安全的类 JedisPool 来获得 Jedis 的实例。JedisPool 可以声明为 static 在多个线程之间共享，扮演连接池的角色。使用时，按需使用 try-with-resources 模式从 JedisPool 获得和归还 Jedis 实例。
```

修复代码的方式如下：

```java
private static JedisPool jedisPool = new JedisPool("127.0.0.1", 6379);

new Thread(() -> {
    try (Jedis jedis = jedisPool.getResource()) {
        for (int i = 0; i < 1000; i++) {
            String result = jedis.get("a");
            if (!result.equals("1")) {
                log.warn("Expect a to be 1 but found {}", result);
                return;
            }
        }
    }
}).start();
new Thread(() -> {
    try (Jedis jedis = jedisPool.getResource()) {
        for (int i = 0; i < 1000; i++) {
            String result = jedis.get("b");
            if (!result.equals("2")) {
                log.warn("Expect b to be 2 but found {}", result);
                return;
            }
        }
    }
}).start();
```

- 连接池一定要复用

```java
@GetMapping("wrong1")
public String wrong1() {
    CloseableHttpClient client = HttpClients.custom()
            .setConnectionManager(new PoolingHttpClientConnectionManager())
            .evictIdleConnections(60, TimeUnit.SECONDS).build();
    try (CloseableHttpResponse response = client.execute(new HttpGet("http://127.0.0.1:45678/httpclientnotreuse/test"))) {
        return EntityUtils.toString(response.getEntity());
    } catch (Exception ex) {
        ex.printStackTrace();
    }
    return null;
}
//CloseableHttpClient 属于第二种模式，即内部带有连接池的 API，其背后是连接池，最佳实践一定是复用。复用方式很简单，你可以把 CloseableHttpClient 声明为 static，只创建一次，并且在 JVM 关闭之前通过 addShutdownHook 钩子关闭连接池，在使用的时候直接使用 CloseableHttpClient 即可，无需每次都创建。
```

改进这段代码的方式：

```java
private static CloseableHttpClient httpClient = null;
static {
    //当然，也可以把CloseableHttpClient定义为Bean，然后在@PreDestroy标记的方法内close这个HttpClient
    httpClient = HttpClients.custom().setMaxConnPerRoute(1).setMaxConnTotal(1).evictIdleConnections(60, TimeUnit.SECONDS).build();
    Runtime.getRuntime().addShutdownHook(new Thread(() -> {
        try {
            httpClient.close();
        } catch (IOException ignored) {
        }
    }));
}

@GetMapping("right")
public String right() {
    try (CloseableHttpResponse response = httpClient.execute(new HttpGet("http://127.0.0.1:45678/httpclientnotreuse/test"))) {
        return EntityUtils.toString(response.getEntity());
    } catch (Exception ex) {
        ex.printStackTrace();
    }
    return null;
}
```

