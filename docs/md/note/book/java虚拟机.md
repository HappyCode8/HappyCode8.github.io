# 类加载

加载、链接、初始化

加载阶段：引导类加载器、扩展类加载器、系统类加载器（也叫应用类加载器）、自定义类加载器

链接阶段：验证、准备、解析

初始化阶段：初始化

## 加载（loading）

1、全限定名获取二进制字节流（本地、网络、压缩包）

2、将字节流转化为方法区的运行时数据结构

3、内存中生成java.lang.Class对象，作为方法区这个类的各种数据的访问入口

## 链接（link）

链接分为验证、准备、解析三个阶段

### 验证（verify）

文件格式验证、元数据验证、字节码验证、符号引用验证

### 准备（prepare）

类变量:被 static 修饰的变量

类成员变量:其他所有类型的变量

在准备阶段，JVM 只会为「类变量」分配内存，而不会为「类成员变量」分配内存，例如下面的代码只会为factor属性分配内存，而不会为website属性分配内存。

```java
public static int factor = 3;
public String website = "www.cnblogs.com/chanshuyi";
```

在准备阶段，JVM 会为类变量分配内存，并为其初始化。但是这里的初始化指的是为变量赋予 Java 语言中该数据类型的零值，而不是用户代码里初始化的值。例如下面的代码在准备阶段之后，sector 的值将是 0，而不是 3。

```java
public static int sector = 3;
```

但如果一个变量是常量（被 static final 修饰）的话，那么在准备阶段，属性便会被赋予用户希望的值。例如下面的代码在准备阶段之后，number 的值将是 3，而不是 0。

```java
public static final int number = 3;
```

两个语句的区别是一个有 final 关键字修饰，另外一个没有。而 final 关键字在 Java 中代表不可改变的意思，意思就是说 number 的值一旦赋值就不会在改变了。既然一旦赋值就不会再改变，那么就必须一开始就给其赋予用户想要的值，因此被 final 修饰的类变量在准备阶段就会被赋予想要的值。而没有被 final 修饰的类变量，其可能在初始化阶段或者运行阶段发生变化，所以就没有必要在准备阶段对它赋予用户想要的值。

### 解析（resolve）

## 初始化

「类成员变量」的内存分配需要等到初始化阶段才开始。

```java
public class test {
    private static int num=1;
    static {
        num=2;
        number=20;
        System.out.println(num);
        System.out.println(number);//此句非法，非法的前向引用
    }
    private static int number=10;//prepare赋值为0，初始化覆盖20-》10
    public static void main(String[] args) {
        System.out.println(test.num);
    }
}
```

对应的字节码文件

```
 0 iconst_1
 1 putstatic #3 <testVM/test.num>
 4 iconst_2
 5 putstatic #3 <testVM/test.num>
 8 bipush 20
10 putstatic #5 <testVM/test.number>
13 bipush 10
15 putstatic #5 <testVM/test.number>
18 return
```

初始化阶段就是执行类的构造器函数<clinit>()的过程，它与类的构造器不同，它是类变量的赋值动作与静态代码块中的语句合并而来的(如果没有这样的操作就不会生成)。

image-20201128231643849

<init>对应类的构造器，是对象初始化方法，它是成员变量的赋值语句、普通代码块的语句、构造函数合并而来的，它在实例化对象的时候执行

![image](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FVXpLQ2F5bS0zWkdxNXBzR1JjOFRGNEJOQ3BnS1ZhVXRRVFNVQjhmQTd4MXV3.png)

<font color='red' size=5>如果一个类具有父类，那么在执行子类的时候clinit前，必须先执行父类的clinit，换句话说，初始化子类时必须要先初始化父类。但是对于静态字段，只有直接定义这个字段的类才会被初始化（执行静态代码块）。因此通过其子类来引用父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。</font>

<font color='red' size=5>同理，在执行子类的init前，必须先执行父类的init。</font>

[面试题加强理解](https://www.cnblogs.com/chanshuyi/p/the_java_class_load_mechamism.html)

<font color='red'>虚拟机保证一个类的clinit方法在多线程下被同步加锁</font>，下面的代码只会输出一次“初始化当前类”，当两个线程同时执行clinit的时候，其中抢到执行权的线程会一直卡住。

```java
public class DeadThreadTest {
    public static void main(String[] args) {
        Runnable r = () -> {
            System.out.println(Thread.currentThread().getName() + "开始");
            DeadThread dead = new DeadThread();
            System.out.println(Thread.currentThread().getName() + "结束");
        };

        Thread t1 = new Thread(r, "线程1");
        Thread t2 = new Thread(r, "线程2");

        t1.start();
        t2.start();
    }
}

class DeadThread {
    static {
        if (true) {
            System.out.println(Thread.currentThread().getName() + "初始化当前类");
            while (true) {

            }
        }
    }
}
```

```
线程2开始
线程1开始
线程2初始化当前类
```

## 类加载器

```java
public static void main(String[] args) {
    System.out.println(ClassLoader.getSystemClassLoader());
    System.out.println(ClassLoader.getSystemClassLoader().getParent());
    System.out.println(ClassLoaderTest.class.getClassLoader());
    System.out.println(String.class.getClassLoader());
    /*
    jdk.internal.loader.ClassLoaders$AppClassLoader@512ddf17 系统类加载器
    jdk.internal.loader.ClassLoaders$PlatformClassLoader@604ed9f0 扩展类加载器
    jdk.internal.loader.ClassLoaders$AppClassLoader@512ddf17 系统类加载器
    null 核心类库使用引导类加载器
    * */
}
```

### 引导类加载器（Bootstrap）

使用c/c++实现，嵌套在JVM内部，加载核心类库，没有父加载器，不继承java.lang.ClassLoader，加载扩展类和应用类加载器，并指定为他们的父类加载器，处于安全考虑，Bootstrap只加载java、javax、sun等开头的类

### 扩展类加载器（PlatformClassLoader）

java语言编写，派生于ClassLoader类，从jdk的安装目录的jre/lib/ext子目录下加载类库，1.8之前叫ExtClassLoader

### 系统类加载器（AppClassLoader）

java语言编写，java应用类都由他加载，加载环境变量classpath或系统属性java.class.path指定路径下的类库

类加载器在java11中的变化[模块化系统](https://dy.163.com/article/FN4MNKQ705372AEM.html)

### 自定义类加载器

为什么？

- 隔离加载类（同一路径下的同名类的仲裁）
- 修改类加载方式
- 扩展加载源（从数据库加载代码）
- 防止源码泄露（加密源码）

怎么办？

- 继承ClassLoader
- 建议把加载逻辑写在findClass中
- 如果没有太过于复杂的需求，可以直接继承URLClassLoader

## 双亲委派机制

如果一个类加载器收到加载请求，他不会自己先去加载，而是委托给父类加载器去加载，并且一直向上委托，直到到达Bootstrap加载器，如果父类加载不成功，才会让子类加载。

优点

- 避免类的重复加载
- 安全（沙箱安全机制）

## 其它

是否是一个类？必须全限定类名相同，加载的ClassLoader是同一个。

主动使用，会触发类的初始化，被动使用不会，以下为主动使用：

- 创建类的实例
- 访问某个类或接口的静态变量，或者对该静态变量赋值
- 调用类的静态方法
- 反射（比如：Class.forName(“com.atguigu.Test”)）
- 初始化一个类的子类
- Java虚拟机启动时被标明为启动类的类（主类）

# 运行时数据区

![image-20210216124124419](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FVkpnTW5vVWpFeEtySWdjSVNUNnRlTUJLeHpaNVl3NWZhaDJyTEgwRVZfaDVR.png)

## 程序计数器（PC寄存器）

- java中的程序计数器并非物理上的寄存器，而是对物理寄存器的一种抽象模拟。

## 虚拟机栈

- 一个线程一个栈，生命周期与线程一致。

- 一个栈帧对应一个方法

- 访问速度仅次于程序计数器，方法执行伴随着进栈出栈，不存在垃圾回收问题

- 虚拟机栈如果采用固定大小值，当超过最大栈容量时，会抛出StackOverflowError异常，如果采用动态扩展，当扩展时无法申请到足够内存时会抛出OutOfMemoryError异常。
  
  ```java
  static void main(String[] args){//StackOverflowError
        main(args);
  }
  ```

- 使用-Xss size来设置栈内存大小，linux,mac等都是1024KB，-Xss 1m， -Xss 1024k ，-Xss 1048756都是设置1m栈

- 不同线程所包含的栈帧是不允许相互引用的，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧。java方法有两种函数返回方式，一种是正常返回，使用return指令，另一种是抛出异常，不管使用哪种方式，都会导致栈帧被弹出。

### 栈帧内部结构

- 局部变量表
  
  - 数字数组，主要用于存储方法参数和定义在方法体内的局部变量，这些数据类型包括基本数据类型、对象引用以及返回地址等
  
  - 不存在数据安全问题
  
  - 所需容量大小是在编译期确定下来的，并保存在方法的Code属性的max  local variables数据项中，在方法运行期间是不会改变局部变量表的大小的。
    
    ![image-20210216154719672](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FVC1KOTd5NVRzVkprN3hmWS1abV9GSUIzcjVmWU9SUzRfd0pDTEt6N3o5SFpn.png)
  
  - 随着栈帧销毁，局部变量表也会随之销毁
  
  - 在局部变量表里，32位以内的类型只占用一个slot(byte,short,char,bool转int存)，64占用两个（long,double）
  
  - 局部变量表可以重复利用
    
    ![image-20210216164801367](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FV3VQSFBhT2NWbEhpVGRsRGRJdnBsZ0J4Wkw5RUdRU21ka3Myam96X1lkQmJR.png)
  
  - 如果当前帧是由构造方法或者实例方法创建的，那么该对象引用this将会存放在index为0的slot处，其余的参数按照参数表顺序继续排列。
    
    ![image-20210216162223742](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FV2hBbjVNZkJqZEp0dHRqakxLZVZGY0JZRlhaanJOZHMzX0t0NDhULVZLemxR.png)
    
    ```java
    public class PCRegister {
        private int count=1;
        public static void main(String[] args) {
           int x=1;
           PCRegister pcRegister=new PCRegister();
           count++;//此处报错，this变量不存在于当前方法的局部变量表中
        }
    
        private void method1() {
            count++;
        }
    }
    ```
  
  - 成员变量包括类变量、实例变量，方法中会包含局部变量
    
    - 类变量在链接的prepare阶段赋默认值，初始化阶段赋显式赋值
    - 实例变量随着对象创建，会在堆空间分配实例变量空间，并进行默认赋值
    - 局部变量必须在使用前显式赋值
  
  - 局部变量表中的变量是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收

- 操作数栈
  
  - 每一个独立的栈帧中，除了包含局部变量表，还包括一个操作数栈，在方法执行过程中，根据字节码指令，往栈中写入数据或者提取数据，主要用于保存中间结果同时作为计算过程中变量临时的存储空间。
  - 操作数栈所需的最大栈深度在编译器句定义好了，保存在方法的code属性中，为max stack的值。
  - 操作数栈并非采用索引的方式访问数据，而是只能通过标准的入栈出栈来访问数据，32位占一个栈深度，long,double等占两个栈深度。
  - Push,load,add等用到操作数栈

- 栈顶缓存技术
  
  - 将栈顶元素全部缓存在物理CPU的寄存器中，以此降低对内存的读写次数，提升执行引擎的执行效率。

- 动态链接（指向运行时常量池的方法引用）
  
  - 每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用。包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接。
  - 当java源文件被编译到字节码文件时，所有的变量和方法的引用都作为符号引用保存在class文件的常量池里。比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示，那么动态链接的作用是为了将符号引用转换为调用方法的直接引用。

- 方法的调用
  
  - 静态链接、动态链接
  
  - 早期绑定、晚期绑定
    
    ```java
    interface A{}
    abstract class B{}
    class C extends B implements A{
        method C(){
          super()//早期绑定
        }
        method C(){
          this()//早期绑定
        }
    }
    class D extends B implements A{}
    class test{
      method x(B b){}//晚期绑定
      method y(A a){}//晚期绑定
    }
    ```
  
  - 非虚方法：如果方法在编译期就确定了具体的调用版本，这个版本在运行时是不可变的，这样的方法称为非虚方法，静态方法、私有方法、final方法、实例构造器、父类方法都是非虚方法。
    
    ```
    invokestatic 调用静态方法，解析阶段确定唯一方法版本
    invokespecial 调用<init>方法、私有以及父类方法，解析阶段确定唯一方法版本
    invokevirtual 调用所有虚方法
    invokeinteface 调用接口方法
    
    动态调用指令
    invokedynamic 动态解析出需要调用的方法，然后执行
    
    前四条指令固化在虚拟机内部，方法调用不可人为干预，而invokedynamic指令则支持用户确定方法版本，其中invokestatic、invokespecial指令调用的方法称为非虚方法，其余的称为虚方法（final修饰的除外）
    ```
  
  - 动态类型语言和静态类型语言两者的区别就在于**对类型的检查是在编译期还是在运行期**，满足前者就是静态类型语言，反之是动态类型语言。说的再直白一点就是，静态类型语言是判断变量自身的类型信息；动态类型语言是判断变量值的类型信息，变量没有类型信息，变量值才有类型信息，这是动态语言的一个重要特征。
    
    ```
    Java：String info = "mogu blog"; (Java是静态类型语言的，会先编译就进行类型检查) 
    JS：var name = "shkstart";  var name = 10; （运行时才进行检查）      
    ```

- 方法重写的本质
  
  1. 找到操作数栈顶的第一个元素所执行的对象的实际类型，记作C。
  
  2. 如果在类型C中找到与常量中的描述符合简单名称都相符的方法，则进行访问权限校验。
     
     - 如果通过则返回这个方法的直接引用，查找过程结束
     - 如果不通过，则返回java.lang.IllegalAccessError 异常
  
  3. 否则，按照继承关系从下往上依次对C的各个父类进行第2步的搜索和验证过程。
  
  4. 如果始终没有找到合适的方法，则抛出java.lang.AbstractMethodError异常。
     
     以上过程称为动态分派。

- 虚方法表
  
  - 在面向对象的编程中，会很频繁的使用到**动态分派**，如果在每次动态分派的过程中都要重新在类的方法元数据中搜索合适的目标的话就可能影响到执行效率。因此，为了提高性能，**JVM采用在类的方法区建立一个虚方法表（virtual method table）来实现**，非虚方法不会出现在表中。使用索引表来代替查找。【上面动态分派的过程，我们可以看到如果子类找不到，还要从下往上找其父类，非常耗时】
  - 每个类中都有一个虚方法表，表中存放着各个方法的实际入口。
  - 虚方法表是什么时候被创建的呢？虚方法表会在类加载的链接阶段被创建并开始初始化，类的变量初始值准备完成之后，JVM会把该类的虚方法表也初始化完毕。如果类中重写了方法，那么调用的时候，就会直接在该类的虚方法表中查找
  
  ![image-20210217161604668](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FVjhxb25EVmlWSkpxZm11WFFuaV9BRUJybnY3bGYzakxUakJNQ1V3N3ljNWJB.png)
  
  1、比如说son在调用toString的时候，Son没有重写过，Son的父类Father也没有重写过，那就直接调用Object类的toString。那么就直接在虚方法表里指明toString直接指向Object类。
  
  2、下次Son对象再调用toString就直接去找Object，不用先找Son-->再找Father-->最后才到Object的这样的一个过程。  

- 方法返回地址
  
  当一个方法开始执行后，只有两种方式可以退出这个方法，
  
  **正常退出：**
  
  1. 执行引擎遇到任意一个方法返回的字节码指令（return），会有返回值传递给上层的方法调用者，简称**正常完成出口**；
  2. 一个方法在正常调用完成之后，究竟需要使用哪一个返回指令，还需要根据方法返回值的实际数据类型而定。
  3. 在字节码指令中，返回指令包含：
     - ireturn：当返回值是boolean，byte，char，short和int类型时使用
     - lreturn：Long类型
     - freturn：Float类型
     - dreturn：Double类型
     - areturn：引用类型
     - return：返回值类型为void的方法、实例初始化方法、类和接口的初始化方法
  
  **异常退出：**
  
  1. 在方法执行过程中遇到异常（Exception），并且这个异常没有在方法内进行处理，也就是只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，简称**异常完成出口**。
  2. 方法执行过程中，抛出异常时的异常处理，存储在一个异常处理表，方便在发生异常的时候找到处理异常的代码
  
  ![img](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FWUZReUZpV0FQWkFrWDVodzdFbXU2RUJIZm9HTG53ckhpeC1zaTlOS0NGUWJR.png)
  
  异常处理表：
  
  - 反编译字节码文件，可得到 Exception table
  - from ：字节码指令起始地址
  - to ：字节码指令结束地址
  - target ：出现异常跳转至地址为 11 的指令执行
  - type ：捕获异常的类型
  
  ![img](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FV1N1azNTd29zaEJsWllqcnp4T0dZY0JrU3hnekozQm1rRzlTbnJnVDlXQmxR.png)

- 面试
  
  - 举例栈溢出的情况？
    
    ​    SOF（StackOverflowError），栈大小分为固定的，和动态变化。如果是固定的就可能出现StackOverflowError。如果是动态变化的，内存不足时就可能出现OOM。
  
  - 调整栈大小，就能保证不出现溢出么
    
    不能保证不溢出，只能保证SOF出现的几率小。
  
  - 分配的栈内存越大越好么？
    
    不是，一定时间内降低了OOM概率，但是会挤占其它的线程空间，因为整个虚拟机的内存空间是有限的。
  
  - 垃圾回收是否涉及到虚拟机栈？
    
    不会。
  
  | 位置                         | 是否有Error | 是否存在GC |
  | -------------------------- | -------- | ------ |
  | PC计数器                      | 无        | 不存在    |
  | 虚拟机栈                       | 有，SOF    | 不存在    |
  | 本地方法栈(在HotSpot的实现中和虚拟机栈一样) |          |        |
  | 堆                          | 有，OOM    | 存在     |
  | 方法区                        | 有        | 存在     |
  
  - 方法中定义的局部变量是否线程安全？
  1. 如果只有一个线程才可以操作此数据，则必是线程安全的。
  2. 如果有多个线程操作此数据，则此数据是共享数据。如果不考虑同步机制的话，会存在线程安全问题。
  
  **具体问题具体分析：**
  
  - 如果对象是在内部产生，并在内部消亡，没有返回到外部，那么它就是线程安全的，反之则是线程不安全的。
  
  ```java
  /**
   * 面试题：
   * 方法中定义的局部变量是否线程安全？具体情况具体分析
   *
   *   何为线程安全？
   *      如果只有一个线程才可以操作此数据，则必是线程安全的。
   *      如果有多个线程操作此数据，则此数据是共享数据。如果不考虑同步机制的话，会存在线程安全问题。
   */
  public class StringBuilderTest {
  
      int num = 10;
  
      //s1的声明方式是线程安全的（只在方法内部用了）
      public static void method1(){
          //StringBuilder:线程不安全
          StringBuilder s1 = new StringBuilder();
          s1.append("a");
          s1.append("b");
          //...
      }
      //sBuilder的操作过程：是线程不安全的（作为参数传进来，可能被其它线程操作）
      public static void method2(StringBuilder sBuilder){
          sBuilder.append("a");
          sBuilder.append("b");
          //...
      }
      //s1的操作：是线程不安全的（有返回值，可能被其它线程操作）
      public static StringBuilder method3(){
          StringBuilder s1 = new StringBuilder();
          s1.append("a");
          s1.append("b");
          return s1;
      }
      //s1的操作：是线程安全的（s1自己消亡了，最后返回的智商s1.toString的一个新对象）
      public static String method4(){
          StringBuilder s1 = new StringBuilder();
          s1.append("a");
          s1.append("b");
          return s1.toString();
      }
  
      public static void main(String[] args) {
          StringBuilder s = new StringBuilder();
  
          new Thread(() -> {
              s.append("a");
              s.append("b");
          }).start();
  
          method2(s);
  
      }
  }
  ```

## 本地方法栈

主要与本地方法接口打交道，基本与虚拟机栈实现一样。

## 堆

- 堆的核心概述
  - 进程唯一，线程共享
  - 

# 本地方法接口

- native修饰的方法，主要是用来与C,C++交互，比如启动线程这种与操作系统进行映射的方法，与abstract不可以共用，与static、权限修饰符刻意共用，也可以抛出异常，没有java的实现。
  
  native方法存在的主要原因：

- 与java环境外的交互，这是本地方法存在的主要原因

- 实现jre与底层操作系统的交互，jvm有一部分本身就是用C写的，如果要使用一些Java语言本身没有提供封装的操作系统的特性时，也需要使用本地方法

- sun的解释器是用C实现的，可以像普通C一样与外部交互

- 本地方法栈，虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法的调用，本地方法栈也是线程私有的，也允许被实现成固定的或者是可动态扩展的的内存大小，在内存溢出方面也是相同的，会有Stack Overflow与outofMemory异常，它的具体方法是本地方法栈中登记native方法，在执行引擎执行时加载本地方法库

- 并不是所有的JVM都支持本地方法，虚拟机规范没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等。

- 在Hotspot JVM中，直接将本地方法栈和虚拟机栈合二为一。

# 执行引擎

## 类加载举例

```java
class Parent {
    // 静态变量
    public static String p_StaticField = "父类--静态变量";
    // 变量
    public String p_Field = "父类--变量";
    // 静态初始化块
    static {
        System.out.println(p_StaticField);
        System.out.println("父类--静态初始化块");
    }
    // 初始化块
    {
        System.out.println(p_Field);
        System.out.println("父类--初始化块");
    }
    // 构造器
    public Parent() {
        System.out.println("父类--构造器");
    }
}
public class SubClass extends Parent {
    // 静态变量
    public static String s_StaticField = "子类--静态变量";
    // 变量
    public String s_Field = "子类--变量";
    // 静态初始化块
    static {
        System.out.println(s_StaticField);
        System.out.println("子类--静态初始化块");
    }
    // 初始化块
    {
        System.out.println(s_Field);
        System.out.println("子类--初始化块");
    }
    // 构造器
    public SubClass() {
        System.out.println("子类--构造器");
    }
    // 程序入口
    public static void main(String[] args) {
        new SubClass();
    }
}
/**
 * 父类--静态变量
 * 父类--静态初始化块
 * 子类--静态变量
 * 子类--静态初始化块
 * 父类--变量
 * 父类--初始化块
 * 父类--构造器
 * 子类--变量
 * 子类--初始化块
 * 子类--构造器
 * */
```

## 

```java
//声明时new操作是在构造函数里new先执行
class T{
        public T(String str){System.out.println("Constrant T ()" + str);}
}

public class Test1 {
    String str1;
    String str2;
    T t = new T(str1);

    public Test1(){
        str1 = "str1";
        str2 = "str2";
        System.out.println("Constrant Test1()");
    }


    public static void main(String[] args){
        new Test1();
    }
}
//输出
Constrant T ()null
Constrant Test1()
```
