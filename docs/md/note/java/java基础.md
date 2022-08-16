

本文大部分内容参考了https://www.pdai.tech/md/java/basic/java-basic-lan-basic.html

# 数据类型

## 包装类型

8个基本类型

>boolean/1  byte/8  char/16   short/16   int/32   float/32  long/64  double/64

基本类型都有对应的包装类型，可以通过装箱拆箱完成

> ```java
> Integer x = 2;     // 装箱通过valueOf(int)
> int y = x;         // 拆箱通过intValue
> ```

为什么要装箱？

- 装箱后有更多方法可以调用
- 更加面向对象，java不纯是面向对象语言，真正面向对象没有基础数据类型
- 泛型中基本类型是不可以做泛型参数的，比如List<Integer>这种

为什么要拆箱？

- 基础数据类型这些频繁使用，处理完整对象需要很多CPU指令、内存

为什么要装箱拆箱？

- 一句话，保证通用性提高性能

什么是缓存池？

- new Integer(123) 每次都新建一个对象，Integer.valueOf(123) 会使用缓存池的对象

缓存池缓存在哪儿？

- Integer对象里有一个静态内部类IntegerCache，这个类里边存了Integer[]数组，数组里存的Integer对象，总共存了-128~127的对象，然后判断数字在这个范围就直接返回

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
```

其余数据类型的缓存类型有吗？

>boolean values true and false
>
>all byte values
>
>short values between -128 and 127
>
>int values between -128 and 127
>
>long values between -128 and 127
>
>char in the range \u0000 to \u007F

为什么不缓存Float、Double？

- Float、Double这没法缓存，小数太多了，没法区分常用不常用

## String

String类为什么要设计成final的？

- 可以缓存hash值，String 的 hash 值经常被使用，不可变的特性可以使得 hash 值也不可变

  ```java
  Test test = new Test(123);
  Map<Test,String> map1 = new HashMap<>();
  map1.put(test,"123");
  test = new Test(123);
  System.out.print(map1.get(test));    //试图取出对应的 value,结果是null
  
  String key = "key";
  HashMap<String,Integer> map = new HashMap<>();
  map.put(key,123);
  key = "key";
  System.out.println(map.get("key"));  //试图取出对应的 value,结果是123
  ```

  > 在使用 String 类型的对象做 key 时我们可以只根据传入的字符串内容就能获得对应存在 map 中的 value 值，而非 String 类型的对象在获得对应的 value 时需要的条件太过苛刻，首先要保证散列码相同，并且经过 equals() 方法判断为 true 时才可以获得对应的 value,这一般需要自己重写。
  >
  > 其次，即使是Test重写了hashCode和equals 方法，也不如String高效，最关键的是String中缓存有个hash变量，它可以缓存hashCode，避免重复计算hashCode，而Test则没有这样的效果。
  >
  > 最后，String的hash值是惰性初始化，创建时并没有直接计算出hashCode。初次计算后进行了缓存，以后查找起来更快了。

- String Pool 的需要，如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool

- 安全性，网络连接参数等

- 线程安全

String类怎么保证是fianl的？

```java
private final char value[];//8之前
private final byte[] value;//8以后

char类型的数据在jvm中占用了两个字节的空间，使用的是UTF-16
编码仅仅优化为byte[]是不够的，关键是提供了ISO-8859-1/Latin-1编码可能（Latin-1就是ISO-8859-1）。

Latin-1编码是用单个字节来表示字符，比两个字节的utf-16节省了一半空间。
所以String类中多了一个编码标志位coder，用来表示使用的是utf-16编码，还是Latin-1编码。
  
String name="jack";使用LATIN1编码，占用4个字节就够了，而原来的char[]，就得占用8个字节。
String name="小明";没得办法，和char[]表示String没什么区别，即使现在是byte[]来表示String，还是得乖乖用UTF16编码，和优化之前一样，没节省空间（LATIN1编码集支持的字符有限，其中就不支持中文字符，因此才保留了UTF16兜底）
```

### String, StringBuffer and StringBuilder

**1. 可变性**

- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 线程安全**

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步

### String.intern()

使用 String.intern() 可以保证相同内容的字符串变量引用同一的内存对象。s1 和 s2 采用 new String() 的方式新建了两个不同对象，而 s3 是通过 s1.intern() 方法取得一个对象引用。intern() 首先把 s1 引用的对象放到 String Pool(字符串常量池)中，然后返回这个对象引用。因此 s3 和 s1 引用的是同一个字符串常量池的对象。

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
System.out.println(s1.intern() == s3);  // true
```

字符串常量池放在哪里？

- JDK1.7之前，运行时常量池（字符串常量池也在里边）是存放在方法区，此时方法区的实现是永久带。
- JDK1.7字符串常量池被单独从方法区移到堆中，运行时常量池剩下的还在永久带（方法区）
- JDK1.8，永久带更名为元空间（方法区的新的实现），但字符串常量池池还在堆中，运行时常量池在元空间（方法区）。

```java
/**
* 不改变s引用情况下改变s的值
* */
public class mianshi1 {
    public static void main(String[] args) throws NoSuchFieldException, IllegalAccessException {
        String s= "abc";
        Field value = s.getClass().getDeclaredField("value");
        value.setAccessible(true);
        value.set(s, new byte[]{'a','b','c','d'});
        System.out.println(s);
    }
}

public class mianshi2 {
    public static void main(String[] args) {
        String s1=new String("abc");//堆。创建了2个对象，编译阶段在常量池创建常量对象，运行阶段在堆中创建String对象
        String s2="abc";//常量池。创建了一个对象
        String s3="abc";
        String s4=s2.intern();
        System.out.println(s1==s2);
        System.out.println(s2==s3);
        System.out.println(s2==s4);
        String s5="123"+"456";//1个常量对象，jvm编译以后的合并优化
        String s6="123"+new String("456");//4个，2个常量池，1个string，还有1个合并以后得
    }
}

/*
* -128~127缓存，所谓缓存其实是一个内部类，在装箱过程中调用valueof时返回缓存的数组对象
* */
public class mianshi3 {
    public static void main(String[] args) {
        Integer i1=127;
        Integer i2=127;
        System.out.println(i1==i2);
        Integer i3=128;
        Integer i4=128;
        System.out.println(i3==i4);
        Integer i5=-127;
        Integer i6=-127;
        System.out.println(i5==i6);
        Integer i7=-128;
        Integer i8=-128;
        System.out.println(i7==i8);
    }
}
```



# 参数传递

在将一个参数传入一个方法时，本质上是将对象的地址以值的方式传递到形参。可以理解为传递了引用的复制，当改变形参这个对象的属性时是可以改变实参的属性的，因为指向了堆中的一个对象，当改变形参的指向时是不会改变实参的指向的。

# 类型转换

```java
short s1 = 1;
s1 += 1;//可行，相当于s1 = (short) (s1 + 1);
s1 = s1 + 1;//不行，1是整形求和时得出的是整形，不能自动向下转型
float f = 1.1;//不行，1.1是double，不能向下转型
```

# switch

从 Java 7 开始，可以在 switch 条件判断语句中使用 String 对象。

# 继承

为什么要访问控制？

- java的四个关键字：public（全部类可访问）、protected（比public少了其他包）、default（比protected少了子类）、private（比default少了包）包不可访问。
- 设计良好的模块会隐藏所有的实现细节，把它的 API 与它的实现清晰地隔离开来。模块之间只通过它们的 API 进行通信，一个模块不需要知道其他模块的内部工作情况，这个概念被称为信息隐藏或封装。因此访问权限应当尽可能地使每个类或者成员不被外界访问。
- 如果子类的方法重写了父类的方法，那么子类中该方法的访问级别不允许低于父类的访问级别。这是为了确保可以使用父类实例的地方都可以使用子类实例，也就是确保满足里氏替换原则。
- 访问控制存在的原因：
  - a、让客户端程序员无法触及他们不应该触及的部分 ；
  -  b、允许库设计者可以改变类内部的工作方式而不用担心会影响到客户端程序员

# 反射

# SPI



# 抽象类与接口

## 抽象类

- 抽象类和抽象方法都使用 abstract 关键字进行声明。抽象类一般会包含抽象方法，抽象方法一定位于抽象类中。
- 抽象类和普通类最大的区别是，抽象类不能被实例化，需要继承抽象类才能实例化其子类。

## 接口

- 接口是抽象类的延伸，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。
- 从 Java 8 开始，接口也可以拥有默认的方法实现，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类。
- 接口的成员(字段 + 方法)默认都是 public 的，并且不允许定义为 private 或者 protected。
- 接口的字段默认都是 static 和 final 的。

抽象类与接口的区别

- 从设计层面上看，抽象类提供了一种 IS-A 关系，那么就必须满足里式替换原则，即子类对象必须能够替换掉所有父类对象。而接口更像是一种 LIKE-A 关系，它只是提供一种方法实现契约，并不要求接口和实现接口的类具有 IS-A 关系。
- 接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是 public 的，而抽象类的成员可以有多种访问权限。

什么时候用接口？

- 需要让不相关的类都实现一个方法以及多重继承使用接口

什么时候用抽象类？

- 需要在几个相关的类中共享代码

```java
public interface test1 {
    public static final int a = 0;
    int b=1;
    //写不写public都一样，不写默认就是public，为啥不是private？private别人都不能访问，没啥用
    //为啥不是protected？假设AB不同包，A是protected，B能访问A的与B是A的子类互为前提，死循环
    private int test() {//为啥有private方法，又不能被重写？纯粹为了代码美观，可以让别的方法使用,为default服务
        return 0;
    }

    public default int test2() {
        //为啥要有default方法？为了避免每个实现接口的方法都要反复重写,default可重写也可不重写
        //可以调用abstract/private/static/private static方法。
        test4();
        test();
        return 0;
    }

    public abstract int test3();//不写默认就是public abstract

    static void test4() {//为啥有static方法？纯粹为了提高利用率，美观,只能调用static/private static
        System.out.println("dewdw");
    }

    private static void test6() {//为啥有？纯粹为了提高利用率，美观，为static方法服务
        System.out.println("private static method");
    }
}

```



# Object方法

## hashcode与equal

自己实现一个equal方法

- 检查是否为同一个对象的引用，如果是直接返回 true；
- 检查是否是同一个类型，如果不是，直接返回 false；
- 将 Object 对象进行转型；
- 判断每个关键域是否相等。

```java
public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        EqualExample that = (EqualExample) o;

        if (x != that.x) return false;
        if (y != that.y) return false;
        return z == that.z;
    
```

等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价。

为什么重写equal方法一定要重写hashcode方法？

```java
EqualExample e1 = new EqualExample(1, 1, 1);
EqualExample e2 = new EqualExample(1, 1, 1);
System.out.println(e1.equals(e2)); // true
HashSet<EqualExample> set = new HashSet<>();
set.add(e1);
set.add(e2);
System.out.println(set.size());   // 2
```

# java8升java11重要特性

[原文](https://www.pdai.tech/md/java/java8up/java9-11.html#java-8-%E5%8D%87java-11-%E9%87%8D%E8%A6%81%E7%89%B9%E6%80%A7%E5%BF%85%E8%AF%BB)

## 语言新特性

- JDK9允许在接口中使用私有方法

  ```java
  //主要用于接口中互相调用
  public interface SayHi {
      private String buildMessage() {
          return "Hello";
      }
      void sayHi(final String message);
      default void sayHi() {
          sayHi(buildMessage());
      }
  }
  ```

- JDK10局部变量类型推断

- JDK11用于Lambda参数的局部变量语法

## 新工具和库更新

- JDK9新增了各种集合的of方法（list,set,map）创建不可变集合,Stream 中增加了新的方法 ofNullable、dropWhile、takeWhile 和 iterate，Collectors 中增加了新的方法 filtering 和 flatMapping，Optional 类中新增了 ifPresentOrElse、or 和 stream 等方法

- JDK9进程API

- JDK9变量句柄

  ```java
  //变量句柄是一个变量或一组变量的引用，包括静态域，非静态域，数组元素和堆外数据结构中的组成部分等。变量句柄的含义类似于已有的方法句柄。
  public class HandleTarget {
      public int count = 1;
  }
  public class VarHandleTest {
      private HandleTarget handleTarget = new HandleTarget();
      private VarHandle varHandle;
      @Before
      public void setUp() throws Exception {
          this.handleTarget = new HandleTarget();
          this.varHandle = MethodHandles
              .lookup()
              .findVarHandle(HandleTarget.class, "count", int.class);
      }
      @Test
      public void testGet() throws Exception {
          assertEquals(1, this.varHandle.get(this.handleTarget));
          assertEquals(1, this.varHandle.getVolatile(this.handleTarget));
          assertEquals(1, this.varHandle.getOpaque(this.handleTarget));
          assertEquals(1, this.varHandle.getAcquire(this.handleTarget));
      }
  }
  ```

- JDK9-IO流新特性

  类 java.io.InputStream 中增加了新的方法来读取和复制 InputStream 中包含的数据。

  - readAllBytes：读取 InputStream 中的所有剩余字节。
  - readNBytes： 从 InputStream 中读取指定数量的字节到数组中。
  - transferTo：读取 InputStream 中的全部字节并写入到指定的 OutputStream 中 。

- JDK9改进应用安全性能

- JDK10根证书认证

- JDK11标准HTTP Client升级，完全支持异步非阻塞（可以使用Spring提供的WebClient）

  ```java
      HttpClient client = HttpClient.newHttpClient();
      HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("http://openjdk.java.net/"))
            .build();
      client.sendAsync(request, BodyHandlers.ofString())
            .thenApply(HttpResponse::body)
            .thenAccept(System.out::println)
            .join();
  ```

- JDK11简化启动单个源代码文件的方法

  ```java
  java HelloWorld.java就可以直接运行文件
  ```

- JDK11 - 支持 TLS 1.3 协议

## JVM优化

- JDK9-统一JVM日志

  移除Java8中废弃的垃圾回收器配置组合，同时把G1设为默认的垃圾回收器，CMS被声明位废弃

- JDK10-统一GC接口

- JDK10-并行全垃圾回收器G1

- JDK11 - Epsilon：低开销垃圾回收器

- JDK11- 低开销的 Heap Profiling

- JDK11 - 可伸缩低延迟垃圾收集器(ZGC)

  根据 SPECjbb 2015 的基准测试，128G 的大堆下最大停顿时间才 1.68ms，远低于 10ms，和 G1 算法相比，改进非常明显

# Java 11 升Java 17 重要特性必读

- Spring6和SpringBoot3只支持JDK17

## 语言新特性

- JDK14-Switch 表达式

  ```java
  //省去break
  int dayOfWeek = switch (day) {
      case MONDAY, FRIDAY, SUNDAY -> 6;
      case TUESDAY                -> 7;
      case THURSDAY, SATURDAY     -> 8;
  case WEDNESDAY              -> 9;
      default              -> 0;
  
  };
  ```

- JDK15-文本块

  ```java
  public static void main(String[] args) {
      String query = """
             SELECT * from USER \
             WHERE `id` = 1 \
             ORDER BY `id`, `name`;\
             """;
      System.out.println(query);
  }
  ```

- JDK16-instanceof 模式匹配

  ```java
  if (person instanceof Student student) {
      student.say();
     // other student operations
  } else if (person instanceof Teacher teacher) {
      teacher.say();
      // other teacher operations
  }
  ```

- JDK16-Records类型

- JDK17-密封的类和接口，限制类只能被谁继承

## 新工具和库更新

- JDK13 - Socket API 重构
- JDK14 - 改进 NullPointerExceptions 提示信息
- JDK15 - 隐藏类 Hidden Classes
- JDK15 - DatagramSocket API重构
- JDK16 - 对基于值的类发出警告
- JDK17 - 增强的伪随机数生成器

## JVM优化

- JDK13 - 增强 ZGC 释放未使用内存
- DK14 - G1 的 NUMA 可识别内存分配
- JDK14 - 删除 CMS 垃圾回收器
- JDK14 - 弃用 ParallelScavenge 和 SerialOld GC 的组合使用
- JDK15 - 禁用偏向锁定
- JDK15 - 低暂停时间垃圾收集器
- JDK16 - ZGC 并发线程处理
- JDK16 - 弹性元空间

# 常见问题

<font color='red'> try中的语句执行错误后边的还会执行吗？</font>

```java
public static void main(String[] args) {
        //System.out.println(test.num);
        try {
            int e=1/0;
            System.out.println("hello1");
        }catch (Exception e){

        }
        System.out.println("hello2");
    }

输出：hello2
```
