

# Scala

[菜鸟教程](https://www.runoob.com/scala/scala-pattern-matching.html)

1. Class, Case Class（样例类）, Object（伴生对象）, Case Object,Trait的区别

   - class与object的区别

     >scala 中没有 static 关键字，所以 对于一个class来说，所有的方法和成员变量在实例被 new 出来之前都是无法访问的
     >object 中所有成员变量和方法默认都是static的, 所以可以直接访问main方法，Scala 中使用单例模式时，除了定义的类之外，还要定义一个同名的 object 对象，它和类的区别是，object对象不能带参数。

   - class与case class的区别

     >适合用于不可变的数据。它是一种特殊的类，能够被优化以用于模式匹配。
     >
     >case class类编译成class文件之后会自动生成apply方法，这个方法负责对象的创建。

   - Trait

     >相当于java的接口

2. 至简原则

   - 普通函数的简化原则

     一般的函数包括函数名、参数名、参数类型、函数返回值类型、函数体

     ```scala
         //(1)return可以省略，scala会使用函数体的最后一行代码作为返回值
         def f1(s:String):String = {
           s+" cls"
         }
     
         //(2)如果函数体只有一行代码，可以省略花括号
         def f2(s:String):String = s+" cls"
     
         //(3)返回值类型如果能够推断出来，那么可以省略(:一起省略)
         def f3(s:String) = s+"cls"
     
         var s = f3("dh ai ")
         println(s)
     
         //(4)如果有return，那么不能省略返回值类型，必须指定
         def f4(s:String) :String = {
           return s+"cls"
         }
     
         //(5)如果有函数声明unit（等同于void）,那么及时函数体中使用return关键字也不起作用
         def f5(s:String):Unit = {
           return s+" cls"
         }
         val as = f5("dh ")
         println(as)
     
         //(6)如果期望是无返回值类型，则可省略=
         def f6(s:String):String={
             return s+" cls"
         }
     
         //(7)如果函数无参，但是声明了参数列表，那么调用时小括号可加可不加
         def f7():String={
           "cls"
         }
         val as1 = f7()
         val as2 = f7
         println(as1 +" "+ as2)
     
         //(8)如果函数没有参数列表，那么小括号可以省略，调用时小括号必须省略
         def f8 = "cls"
         println(f8)
     
         //(9)如果不关心名称，只关心函数名(def)可以省略
         def f9 = (x:String)=>{println("wusong")}
         def f10(f:String=>Unit) = {
           f("")
         }
         f10(f9)
         println(f10((x:String)=>{println("wusong")}))
     ```

   - 匿名函数

     **(x:Int)=>{函数体}**

     ```scala
     		//定义一个函数:参数包含数据和逻辑函数
         def operation(arr:Array[Int],op:Int=>Int)={
           for (elem <- arr) yield op(elem)
         }
     
         //定义逻辑函数
         def op(ele:Int):Int = {
           ele+1
         }
     
         //（3）标准函数调用
         val arr = operation(Array(1,2,3,4),op)
         println(arr.mkString(","))
     
         //(4)采用匿名函数
         val arr1 = operation(Array(1,2,3,4),(ele:Int)=>{
           ele+1
         })
         println(arr1.mkString(","))
     
         //(4.1)参数类型可以省略，会根据形参进行自动推导
         var arr2 = operation(Array(1,2,3,4),(ele)=>{
           ele+1
         })
         println(arr2.mkString(","))
     
         //(4.2)类型省略之后，发现只有一个参数，则圆括号可以省略；
         //其他情况:没有参数和参数超过1的永远不能省略圆括号
         val arr3 = operation(Array(1,2,3,4),ele =>{
           ele+1
         })
         println(arr3.mkString(","))
     
         //(4.3)匿名函数如果只有一行，则大括号也可以省略
         val arr4 = operation(Array(1,2,3,4),ele => ele+1)
         println(arr4.mkString(","))
     
         //(4.4)如果参数只出现一次，则参数可以省略切后面参数可以用_代替
         val arr5 = operation(Array(1,2,3,4),_+1)
         println(arr5.mkString(" "))
     
     
     //实际例子
     def calculator(a: Int, b: Int, op: (Int, Int) => Int): Int = {
      op(a, b)
      }
      // （1）标准版
      println(calculator(2, 3, (x: Int, y: Int) => {x + y}))
      // （2）如果只有一行，则大括号也可以省略
      println(calculator(2, 3, (x: Int, y: Int) => x + y))
      // （3）参数的类型可以省略，会根据形参进行自动的推导;
      println(calculator(2, 3, (x , y) => x + y))
      // （4）如果参数只出现一次，则参数省略且后面参数可以用_代替
      println(calculator(2, 3, _ + _))
     ```

     

# Spark



# 安装与启动

docker pull singularities/spark

```yml
version: "2"
 
services:
  master:
    image: singularities/spark
    command: start-spark master
    hostname: master
    ports:
      - "6066:6066"
      - "7070:7070"
      - "8080:8080"
      - "50070:50070"
  worker:
    image: singularities/spark
    command: start-spark worker master
    environment:
      SPARK_WORKER_CORES: 1
      SPARK_WORKER_MEMORY: 2g
    links:
      - master
    volumes:
      # 设置本地目录和镜像目录的映射关系（格式：本地机器目录:镜像中对应路径）
      - /Users/wyj/Documents/spark/data:/input_files
```

