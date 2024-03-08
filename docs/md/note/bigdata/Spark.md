# Scala

[菜鸟教程](https://www.runoob.com/scala/scala-pattern-matching.html)

1. Class, Case Class（样例类）, Object（伴生对象）, Case Object,Trait的区别
   
   - class与object的区别
     
     > scala 中没有 static 关键字，所以 对于一个class来说，所有的方法和成员变量在实例被 new 出来之前都是无法访问的
     > object 中所有成员变量和方法默认都是static的, 所以可以直接访问main方法，Scala 中使用单例模式时，除了定义的类之外，还要定义一个同名的 object 对象，它和类的区别是，object对象不能带参数。
   
   - class与case class的区别
     
     > 适合用于不可变的数据。它是一种特殊的类，能够被优化以用于模式匹配。
     > 
     > case class类编译成class文件之后会自动生成apply方法，这个方法负责对象的创建。
   
   - Trait
     
     > 相当于java的接口

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
     ```

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

## 实例

- 去重
  
  ```
  data1:
  2012-3-1 a
  2012-3-2 b
  
  data2:
  2012-3-1 a
  2012-3-2 a
  ```
  
  思路：将每一行作为键，按照键值去重
  
  ```java
  import org.apache.spark.SparkConf;
  import org.apache.spark.api.java.JavaPairRDD;
  import org.apache.spark.api.java.JavaRDD;
  import org.apache.spark.api.java.JavaSparkContext;
  import org.apache.spark.api.java.function.Function2;
  import org.apache.spark.api.java.function.PairFunction;
  import org.apache.spark.api.java.function.VoidFunction;
  import scala.Tuple2;
  
  public class DeduplicatedApplication {
      public static void main(String[] args) {
          //1、构建sparkconf,设置配置信息
          /** Local模式就是运行在一台计算机上的模式，通常就是用于在本机上练手和测试。它可以通过以下集中方式设置Master
           local: 所有计算都运行在一个线程当中，没有任何并行计算，通常我们在本机执行一些测试代码，或者练手，就用这种模式;
           local[K]: 指定使用几个线程来运行计算，比如local[4]就是运行4个Worker线程。通常我们的Cpu有几个Core，就指定几个线程，最大化利用Cpu的计算能力;
           local[*]: 这种模式直接帮你按照Cpu最多Cores来设置线程数了。*/
          SparkConf conf = new SparkConf().setAppName("DeduplicatedApplication").setMaster("local[*]");
          ;
          //2、构建java版的sparkContext
          JavaSparkContext javaSparkContext = new JavaSparkContext(conf);
          //3、读取数据文件
          JavaRDD<String> dataRDD1 = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/data1.txt");
          JavaRDD<String> dataRDD2 = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/data1.txt");
          //4、合并数据集
          JavaRDD<String> dataRDD = dataRDD1.union(dataRDD2);
          //5、行为key, ""为value, 生成<行、"">键值对
          JavaPairRDD<String, String> map = dataRDD.mapToPair(
                  //三个参数分别表示输入是一个String，输出是<String, String>
                  (PairFunction<String, String, String>) s -> new Tuple2<>(s, "")
          );
          //6、通过合并键去重
          JavaPairRDD<String, String> result = map.reduceByKey(
                  //前两个参数分别表示输入的KV都是String，第三个参数表示输出的是String
                  (Function2<String, String, String>) (str1, str2) -> str2
          );
          //7、输出
          result.foreach(
                  //两个参数分别表示输入的KV
                  (VoidFunction<Tuple2<String, String>>) it -> System.out.println(it._1)
          );
          javaSparkContext.stop();
      }
  }
  ```

- 排序
  
  ```
  2
  32
  654
  32
  15
  756
  65223
  ```
  
  思路：
  
  将数字作为key直接按ley排序
  
  ```java
  import org.apache.spark.SparkConf;
  import org.apache.spark.api.java.JavaPairRDD;
  import org.apache.spark.api.java.JavaRDD;
  import org.apache.spark.api.java.JavaSparkContext;
  import org.apache.spark.api.java.function.PairFunction;
  import scala.Tuple2;
  
  public class DataRankApplication {
      public static void main(String[] args) {
          //1、构建sparkconf,设置配置信息
          SparkConf conf = new SparkConf().setAppName("DataRankApplication").setMaster("local[*]");
          ;
          //2、构建java版的sparkContext
          JavaSparkContext javaSparkContext = new JavaSparkContext(conf);
          //3、读取数据文件
          JavaRDD<String> dataRDD1 = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/file1.txt");
          JavaRDD<String> dataRDD2 = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/file2.txt");
          JavaRDD<String> dataRDD3 = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/file3.txt");
          //4、合并数据集
          JavaRDD<String> dataRDD = dataRDD1.union(dataRDD2).union(dataRDD3);
          //5、生成<值、" ">键值对
          JavaPairRDD<Integer, String> map = dataRDD.mapToPair(
                  //输入是String,输出是<Integer, String>
                  (PairFunction<String, Integer, String>) s -> new Tuple2<>(Integer.parseInt(s), "")
          );
          //6、进行排序
          JavaPairRDD<Integer, String> result = map.sortByKey();
  
          //7、输出,这里要进行收集才会排序
          result.collect().iterator().forEachRemaining(
                  it-> System.out.println(it._1)
          );
          javaSparkContext.stop();
          /** 如果要输出排名，可以使用下面的输出方式
           * Object[] array = result.collect().toArray();
            for (int i = 0; i < array.length; i++) {
                Tuple2<Integer,String> tuple2 = (Tuple2<Integer,String>) array[i];
                System.out.println(i+1+" "+ tuple2._1);
            }
           * */
      }
  }
  ```

- 求均值
  
  ```
  chinese:
  张三 78
  李四 89
  王五 96
  赵六 67
  english:
  张三 88
  李四 99
  王五 66
  赵六 77
  ```
  
  思路：
  
  将名字作为key,对value求和同时计数然后对value相除
  
  ```java
  import org.apache.spark.SparkConf;
  import org.apache.spark.api.java.JavaPairRDD;
  import org.apache.spark.api.java.JavaRDD;
  import org.apache.spark.api.java.JavaSparkContext;
  import org.apache.spark.api.java.function.PairFunction;
  import scala.Tuple2;
  
  public class AverageApplication {
      public static void main(String args[]) {
          //1、构建sparkconf,设置配置信息
          SparkConf conf = new SparkConf().setAppName("Average Application").setMaster("local[2]");;
          //2、构建java版的sparkContext
          JavaSparkContext sc = new JavaSparkContext(conf);
          //3、读取数据文件
          JavaRDD<String> dataRDD1 = sc.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/math.txt");
          JavaRDD<String> dataRDD2 = sc.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/chinese.txt");
          JavaRDD<String> dataRDD3 = sc.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/english.txt");
          //4、合并数据集
          JavaRDD<String> lines = dataRDD1.union(dataRDD2).union(dataRDD3);
          //5、每一行转化为键值对,然后转化为<名字，成绩，1>
          JavaPairRDD<String, Integer> result = lines
                  .mapToPair((PairFunction<String, String, Integer>) s -> {
                      String[] line = s.split(" ");
                      return new Tuple2<>(line[0], Integer.parseInt(line[1]));
                  })
                  //<名字, <成绩, 1>>
                  .mapValues(x -> new Tuple2<>(x, 1))
                  //<名字，<成绩和，课程数>>
                  .reduceByKey((x, y) -> new Tuple2<>(x._1 + y._1, x._2 + y._2))
                  //<名字，平均成绩>
                  .mapToPair(x -> new Tuple2<>(x._1, x._2._1 / x._2._2));
  
          System.out.println(result.collect());
      }
  }
  ```

- 联表查询
  
  ```
  child parent
  Tom Lucy
  Tom Jack
  Jone Lucy
  Jone Jack
  Lucy Mary
  Lucy Ben
  Jack Alice
  Jack Jesse
  Terry Alice
  Terry Jesse
  Philip Terry
  Philip Alma
  Mark Terry
  Mark Alma
  ```
  
  给定的是儿子与父母的关系，根据此表输出祖父母与孙子的关系
  
  ```java
  import java.util.ArrayList;
  import java.util.Arrays;
  import java.util.List;
  
  public class SingleRelationApplication {
      public static void main(String[] args) {
          //1、构建sparkconf,设置配置信息
          SparkConf conf = new SparkConf().setAppName("SingleRelation Application").setMaster("local[*]");
          ;
          //2、构建java版的sparkContext
          JavaSparkContext javaSparkContext = new JavaSparkContext(conf);
          //3、读取数据文件
          JavaRDD<String> dataRDD = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/data/relation.txt");
          //去掉第一行
          List list = Arrays.asList(dataRDD.collect().toArray());
          dataRDD = javaSparkContext.parallelize(list.subList(1, list.size()));
          //4、把每一行映射成为键值对，<x y> <y z>,输出为<x 0y><y 1x> <y 0z><z 1y>
          //<x y><y z> -> <x 0y><y 0z>
          JavaPairRDD<String, String> map1 = dataRDD.mapToPair(
                  (PairFunction<String, String, String>) s -> {
                      String[] line = s.split(" ");
                      return new Tuple2<>(line[0], "0" + line[1]);//0表示为父亲
                  }
          );
          //<x y><y z> -> <y 1x><z 1y>
          JavaPairRDD<String, String> map2 = dataRDD.mapToPair(
                  (PairFunction<String, String, String>) s -> {
                      String[] line = s.split(" ");
                      return new Tuple2<>(line[1], "1" + line[0]);//1表示为孩子
                  }
          );
          //5、合并
          JavaPairRDD<String, String> map = map1.union(map2);
          //6、根据键建立祖孙联系<子, 0父 1孙>
          JavaPairRDD<String, String> result = map.reduceByKey(
                  (Function2<String, String, String>) (str1, str2) -> str1 + "#" + str2
          );
  
          //7、输出<祖,孙>
          result.collect().iterator().forEachRemaining(it -> {
              String[] split = it._2.split("#");
              ArrayList<String> grandParents = new ArrayList<>();
              ArrayList<String> grandSons = new ArrayList<>();
              Arrays.stream(split).forEach(v -> {
                  if (v.startsWith("0")) {
                      grandParents.add(v.substring(1));
                  } else {
                      grandSons.add(v.substring(1));
                  }
              });
              if (grandParents.size() > 0 && grandSons.size() > 0) {
                  for (var grandParent : grandParents) {
                      for (var grandSon : grandSons) {
                          System.out.println(grandParent + " " + grandSon);
                      }
                  }
              }
          });
      }
  }
  ```

## 简单项目

- 数据格式
  
  > ```
  > 2019-07-17_95_26070e87-1ad7-49a3-8fb3-cc741facaddf_37_2019-07-17 00:00:02_手机_-1_-1_null_null_null_null_3
  > ```
  > 
  > - 每一行数据表示用户的一次行为，这个行为只能是4种行为的一种（搜索，点击，下单，支付）
  > 
  > - 如果搜索关键字为null,表示数据不是搜索数据
  > 
  > - 如果点击的品类ID和产品ID为-1，表示数据不是点击数据
  > 
  > - 针对于下单行为，一次可以下单多个商品，所以品类ID和产品ID可以是多个，id之
  >   
  >   间采用逗号分隔，如果本次不是下单行为，则数据采用 null 表示
  > 
  > - 支付行为和下单行为类似
  > 
  > ```java
  > class UserVisitAction{
  >    public String date;//用户点击行为的日期
  >    public Long user_id;//用户的ID
  >    public String session_id;//Session的ID
  >    public Long page_id;//某个页面的ID
  >    public String action_time;//动作的时间点
  >    public String search_keyword;//用户搜索的关键词
  >    public Long click_category_id;//某一个商品品类的ID
  >    public Long click_product_id;//某一个商品的ID
  >    public String order_category_ids;//一次订单中所有品类的ID集合
  >    public String order_product_ids;//一次订单中所有商品的ID集合
  >    public String pay_category_ids;//一次支付中所有品类的ID集合
  >    public String pay_product_ids;//一次支付中所有商品的ID集合
  >    public Long city_id;//城市id
  > }
  > ```

- 需求1：Top10热门品类
  
  > 需求说明：品类是指产品的分类，大型电商网站品类分多级，咱们的项目中品类只有一级，不同的
  > 
  > 公司可能对热门的定义不一样。我们按照每个品类的点击、下单、支付的量来统计热门品类。
  > 
  > 综合排名 = 点击数20%+下单数30%+支付数50% 
  > 
  > 本项目需求优化为:先按照点击数排名，靠前的就排名高;如果点击数相同，再比较下单数;下单数再相同，就比较支付数。
  > 
  > - 实现方案1：分别统计每个品类点击的次数，下单的次数和支付的次数
  >   
  >   ```java
  >   import org.apache.spark.SparkConf;
  >   import org.apache.spark.api.java.JavaPairRDD;
  >   import org.apache.spark.api.java.JavaRDD;
  >   import org.apache.spark.api.java.JavaSparkContext;
  >   import org.apache.spark.api.java.function.FlatMapFunction;
  >   import org.apache.spark.api.java.function.Function2;
  >   import org.apache.spark.api.java.function.PairFunction;
  >   import org.apache.spark.api.java.function.VoidFunction;
  >   import scala.Serializable;
  >   import scala.Tuple2;
  >   import scala.Tuple3;
  >   
  >   import java.util.*;
  >   
  >   //top10热门品类
  >   public class HotCategoryTop10Analysis {
  >      public static void main(String[] args) {
  >          // Top10热门品类
  >          SparkConf sparConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis");
  >          JavaSparkContext sparkContext = new JavaSparkContext(sparConf);
  >   
  >          // 1. 读取原始日志数据
  >          JavaRDD<String> actionRDD = sparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/user_visit_action.txt");
  >   
  >          // 2. 统计品类的点击数量：（品类ID，点击数量）
  >          JavaPairRDD<String, Integer> clickCountRDD = actionRDD.filter(it -> !"-1".equals(it.split("_")[6]))//过滤
  >                  //参数依次是入参、出参<k,v>
  >                  .mapToPair((PairFunction<String, String, Integer>) it -> new Tuple2<>(it.split("_")[6], 1))//将第六项提取为<商品,1>
  >                  .reduceByKey((Function2<Integer, Integer, Integer>) Integer::sum);
  >          clickCountRDD.foreach((VoidFunction<Tuple2<String, Integer>>) it -> System.out.println(it._1 + ":" + it._2));
  >   
  >          // 3. 统计品类的下单数量：（品类ID，下单数量）, 先切成【(1,1)，(2,1)，(3,1)】
  >          JavaRDD<String> orderActionRDD = actionRDD.filter(it -> !"null".equals(it.split("_")[8]));
  >          JavaPairRDD<String, Integer> orderCountRDD = orderActionRDD
  >                  .flatMap((FlatMapFunction<String, String>) it -> Arrays.asList(it.split("_")[8].split(",")).iterator())
  >                  .mapToPair((PairFunction<String, String, Integer>) it -> new Tuple2<>(it, 1))
  >                  .reduceByKey((Function2<Integer, Integer, Integer>) Integer::sum);
  >          orderCountRDD.foreach((VoidFunction<Tuple2<String, Integer>>) it -> System.out.println(it._1 + ":" + it._2));
  >   
  >          // 4. 统计品类的支付数量：（品类ID，支付数量）, 先切成【(1,1)，(2,1)，(3,1)
  >          JavaRDD<String> payActionRDD = actionRDD.filter(it -> !"null".equals(it.split("_")[10]));
  >          JavaPairRDD<String, Integer> payCountRDD = payActionRDD
  >                  .flatMap((FlatMapFunction<String, String>) it -> Arrays.asList(it.split("_")[10].split(",")).iterator())
  >                  .mapToPair((PairFunction<String, String, Integer>) it -> new Tuple2<>(it, 1))
  >                  .reduceByKey((Function2<Integer, Integer, Integer>) Integer::sum);
  >          payCountRDD.foreach((VoidFunction<Tuple2<String, Integer>>) it -> System.out.println(it._1 + ":" + it._2));
  >          // 5. 将品类进行排序，并且取前10名，cogroup = connect + group
  >          //    点击数量排序，下单数量排序，支付数量排序
  >          //    元组排序：先比较第一个，再比较第二个，再比较第三个，依此类推
  >          //    ( 品类ID, ( 点击数量, 下单数量, 支付数量 ) )
  >          JavaPairRDD<String, Tuple3<Iterable<Integer>, Iterable<Integer>, Iterable<Integer>>> cogroup = clickCountRDD.cogroup(orderCountRDD, payCountRDD);
  >          JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> analysisRDD = cogroup.mapValues(it -> {
  >              var clickCnt = 0;
  >              Iterator<Integer> clickIter = it._1().iterator();
  >              if (clickIter.hasNext()) {
  >                  clickCnt = clickIter.next();
  >              }
  >              var orderCnt = 0;
  >              Iterator<Integer> orderIter = it._2().iterator();
  >              if (orderIter.hasNext()) {
  >                  orderCnt = orderIter.next();
  >              }
  >              var payCnt = 0;
  >              Iterator<Integer> payIter = it._3().iterator();
  >              if (payIter.hasNext()) {
  >                  payCnt = payIter.next();
  >              }
  >              return new Tuple3<>(clickCnt, orderCnt, payCnt);
  >          });
  >   
  >          List<Tuple2<Tuple3<Integer, Integer, Integer>, String>> result = analysisRDD
  >                  .mapToPair(s -> new Tuple2<>(s._2, s._1))
  >                  .sortByKey(new SparkTupleComparator())
  >                  .take(10);
  >   
  >          // 6. 将结果采集到控制台打印出来
  >          result.iterator().forEachRemaining(it -> {
  >                      System.out.println(it._2 + " " + it._1);
  >                  }
  >          );
  >   
  >          sparkContext.stop();
  >      }
  >   }
  >   
  >   class SparkTupleComparator implements Comparator<Tuple3<Integer, Integer, Integer>>, Serializable {
  >      @Override
  >      public int compare(Tuple3<Integer, Integer, Integer> o1, Tuple3<Integer, Integer, Integer> o2) {
  >          if (!Objects.equals(o1._1(), o2._1())) {
  >              return o2._1() - o1._1();
  >          } else if (!Objects.equals(o1._2(), o2._2())) {
  >              return o2._2() - o1._2();
  >          }
  >          return o2._3() - o1._3();
  >      }
  >   }
  >   ```
  >   
  >   一个优化的方案
  >   
  >   ```java
  >   import org.apache.spark.SparkConf;
  >   import org.apache.spark.api.java.JavaPairRDD;
  >   import org.apache.spark.api.java.JavaRDD;
  >   import org.apache.spark.api.java.JavaSparkContext;
  >   import org.apache.spark.api.java.function.FlatMapFunction;
  >   import org.apache.spark.api.java.function.Function2;
  >   import org.apache.spark.api.java.function.PairFunction;
  >   import org.apache.spark.api.java.function.VoidFunction;
  >   import scala.Tuple2;
  >   import scala.Tuple3;
  >   
  >   import java.util.Arrays;
  >   import java.util.List;
  >   
  >   public class HotCategoryTop10Analysis1 {
  >      public static void main(String[] args) {
  >           /*Top10热门品类
  >           Q : actionRDD重复使用
  >           Q : cogroup性能可能较低*/
  >          SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis");
  >          JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
  >          // 1. 读取原始日志数据
  >          JavaRDD<String> actionRDD = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/user_visit_action.txt");
  >          actionRDD.cache();
  >          // 2. 统计品类的点击数量：（品类ID，点击数量）
  >          JavaPairRDD<String, Integer> clickCountRDD = actionRDD.filter(it -> !"-1".equals(it.split("_")[6]))//过滤
  >                  //参数依次是入参、出参<k,v>
  >                  .mapToPair((PairFunction<String, String, Integer>) it -> new Tuple2<>(it.split("_")[6], 1))//将第六项提取为<商品,1>
  >                  .reduceByKey((Function2<Integer, Integer, Integer>) Integer::sum);
  >          clickCountRDD.foreach((VoidFunction<Tuple2<String, Integer>>) it -> System.out.println(it._1 + ":" + it._2));
  >          // 3. 统计品类的下单数量：（品类ID，下单数量）, 先切成【(1,1)，(2,1)，(3,1)】
  >          JavaRDD<String> orderActionRDD = actionRDD.filter(it -> !"null".equals(it.split("_")[8]));
  >          JavaPairRDD<String, Integer> orderCountRDD = orderActionRDD
  >                  .flatMap((FlatMapFunction<String, String>) it -> Arrays.asList(it.split("_")[8].split(",")).iterator())
  >                  .mapToPair((PairFunction<String, String, Integer>) it -> new Tuple2<>(it, 1))
  >                  .reduceByKey((Function2<Integer, Integer, Integer>) Integer::sum);
  >          orderCountRDD.foreach((VoidFunction<Tuple2<String, Integer>>) it -> System.out.println(it._1 + ":" + it._2));
  >          // 4. 统计品类的支付数量：（品类ID，支付数量）, 先切成【(1,1)，(2,1)，(3,1)
  >          JavaRDD<String> payActionRDD = actionRDD.filter(it -> !"null".equals(it.split("_")[10]));
  >          JavaPairRDD<String, Integer> payCountRDD = payActionRDD
  >                  .flatMap((FlatMapFunction<String, String>) it -> Arrays.asList(it.split("_")[10].split(",")).iterator())
  >                  .mapToPair((PairFunction<String, String, Integer>) it -> new Tuple2<>(it, 1))
  >                  .reduceByKey((Function2<Integer, Integer, Integer>) Integer::sum);
  >          payCountRDD.foreach((VoidFunction<Tuple2<String, Integer>>) it -> System.out.println(it._1 + ":" + it._2));
  >          // (品类ID, 点击数量) => (品类ID, (点击数量, 0, 0))
  >          // (品类ID, 下单数量) => (品类ID, (0, 下单数量, 0))=> (品类ID, (点击数量, 下单数量, 0))
  >          // (品类ID, 支付数量) => (品类ID, (0, 0, 支付数量)) => (品类ID, (点击数量, 下单数量, 支付数量))
  >          // (品类ID, ( 点击数量, 下单数量, 支付数量))
  >          JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> rdd1 = clickCountRDD.mapValues(it -> new Tuple3(it, 0, 0));
  >          JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> rdd2 = orderCountRDD.mapValues(it -> new Tuple3(0, it, 0));
  >          JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> rdd3 = payCountRDD.mapValues(it -> new Tuple3(0, 0, it));
  >          JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> unionRdd = rdd1.union(rdd2).union(rdd3);
  >          JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> analysisRDD = unionRdd.reduceByKey((t1, t2) -> new Tuple3<>(t1._1()+t2._1(),t1._2()+t2._2(),t1._3()+t2._3()));
  >          List<Tuple2<Tuple3<Integer, Integer, Integer>, String>> result = analysisRDD
  >                  .mapToPair(s -> new Tuple2<>(s._2, s._1))
  >                  .sortByKey(new SparkTupleComparator())
  >                  .take(10);
  >          // 5. 将结果采集到控制台打印出来
  >          result.iterator().forEachRemaining(it -> {
  >                      System.out.println(it._2 + " " + it._1);
  >                  }
  >          );
  >   
  >          javaSparkContext.stop();
  >      }
  >   }
  >   ```
  > 
  > - 实现方案2：一次性统计每个品类点击的次数，下单的次数和支付的次数
  >   
  >   ```java
  >   import org.apache.spark.SparkConf;
  >   import org.apache.spark.api.java.JavaPairRDD;
  >   import org.apache.spark.api.java.JavaRDD;
  >   import org.apache.spark.api.java.JavaSparkContext;
  >   import org.apache.spark.api.java.function.FlatMapFunction;
  >   import org.apache.spark.api.java.function.PairFunction;
  >   import scala.Tuple2;
  >   import scala.Tuple3;
  >   
  >   import java.util.Arrays;
  >   import java.util.List;
  >   import java.util.stream.Stream;
  >   
  >   public class HotCategoryTop10Analysis2 {
  >      public static void main(String[] args) {
  >          // Q : 存在大量的shuffle操作（reduceByKey）
  >          // reduceByKey 聚合算子，spark会提供优化，缓存
  >          SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis");
  >          JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
  >   
  >          JavaRDD<String> actionRDD = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/user_visit_action.txt");
  >          JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> flatRDD = actionRDD
  >                  .flatMap(
  >                          (FlatMapFunction<String, Tuple2<String, Tuple3<Integer, Integer, Integer>>>) s -> {
  >                              String[] data = s.split("_");
  >                              if (!"-1".equals(data[6])) {
  >                                  return Stream.of(data[6]).map(it -> new Tuple2<>(it, new Tuple3<>(1, 0, 0))).iterator();
  >                              }
  >                              if (!"null".equals(data[8])) {
  >                                  return Arrays.stream(data[8].split(",")).map(it -> new Tuple2<>(it, new Tuple3<>(0, 1, 0))).iterator();
  >                              }
  >                              if (!"null".equals(data[10])) {
  >                                  return Arrays.stream(data[10].split(",")).map(it -> new Tuple2<>(it, new Tuple3<>(0, 0, 1))).iterator();
  >                              }
  >                              //此处比较trick，scala可以返回空列表，java暂时没找到好的方法
  >                              return Stream.of(new String[]{"0"}).map(it -> new Tuple2<>(it, new Tuple3<>(0, 0, 0))).iterator();
  >                          }).mapToPair((PairFunction<Tuple2<String, Tuple3<Integer, Integer, Integer>>, String, Tuple3<Integer, Integer, Integer>>) it -> new Tuple2<>(it._1, it._2));
  >          JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> analysisRDD = flatRDD.reduceByKey((t1, t2) -> new Tuple3<>(t1._1() + t2._1(), t1._2() + t2._2(), t1._3() + t2._3()));
  >          List<Tuple2<Tuple3<Integer, Integer, Integer>, String>> result = analysisRDD.mapToPair(s -> new Tuple2<>(s._2, s._1))
  >                  .sortByKey(new SparkTupleComparator())
  >                  .take(10);
  >          result.iterator().forEachRemaining(it -> {
  >                      System.out.println(it._2 + " " + it._1);
  >                  }
  >          );
  >          javaSparkContext.stop();
  >      }
  >   }
  >   ```
  > 
  > - 实现方案3：使用累加器的方式聚合数据
  >   
  >   ```java
  >   import lombok.AllArgsConstructor;
  >   import org.apache.spark.SparkConf;
  >   import org.apache.spark.api.java.JavaRDD;
  >   import org.apache.spark.api.java.JavaSparkContext;
  >   import org.apache.spark.util.AccumulatorV2;
  >   import scala.Tuple2;
  >   
  >   import java.io.Serializable;
  >   import java.util.ArrayList;
  >   import java.util.HashMap;
  >   import java.util.List;
  >   import java.util.Map;
  >   import java.util.stream.Collectors;
  >   
  >   public class HotCategoryTop10Analysis3 {
  >      public static void main(String[] args) {
  >          //前边的方法都有shuffle操作，性能可能较低
  >          SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis");
  >          JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
  >          JavaRDD<String> actionRDD = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/user_visit_action.txt");
  >          HotCategoryAccumulator acc = new HotCategoryAccumulator();
  >          javaSparkContext.sc().register(acc, "hotCategory");
  >          actionRDD.foreach(it -> {
  >                      String[] datas = it.split("_");
  >                      if (!"-1".equals(datas[6])) {
  >                          acc.add(new Tuple2<>(datas[6], "click"));
  >                      } else if (!"null".equals(datas[8])) {
  >                          String[] ids = datas[8].split(",");
  >                          for (String id : ids) {
  >                              acc.add(new Tuple2<>(id, "order"));
  >                          }
  >                      } else if (!"null".equals(datas[10])) {
  >                          String[] ids = datas[10].split(",");
  >                          for (String id : ids) {
  >                              acc.add(new Tuple2<>(id, "pay"));
  >                          }
  >                      }
  >                  }
  >          );
  >   
  >          List<HotCategory> result = new ArrayList<>(acc.value().values()).stream().sorted((o1, o2) -> {
  >              if (!o1.clickCnt.equals(o2.clickCnt)) {
  >                  return o2.clickCnt - o1.clickCnt;
  >              } else if (!o1.orderCnt.equals(o2.orderCnt)) {
  >                  return o2.orderCnt - o1.orderCnt;
  >              }
  >              return o2.payCnt - o1.payCnt;
  >          }).limit(10).collect(Collectors.toList());
  >          // 6. 将结果采集到控制台打印出来
  >          result.iterator().forEachRemaining(it-> System.out.println(it.cid+" "+it.clickCnt+" "+it.orderCnt+" "+it.payCnt));
  >   
  >          //resultRDD.foreach(println)
  >          javaSparkContext.stop();
  >      }
  >   }
  >   
  >   @AllArgsConstructor
  >   class HotCategory implements Serializable {
  >      public String cid;
  >      public Integer clickCnt;
  >      public Integer orderCnt;
  >      public Integer payCnt;
  >   }
  >   
  >   /**
  >   * 累加器用来把 Executor 端变量信息聚合到 Driver 端。在 Driver 程序中定义的变量，在
  >   * Executor 端的每个 Task 都会得到这个变量的一份新的副本，每个 task 更新这些副本的值后，
  >   * 传回 Driver 端进行 merge
  >   * */
  >   class HotCategoryAccumulator extends AccumulatorV2<Tuple2<String, String>, Map<String, HotCategory>> {
  >      Map<String, HotCategory> hcMap = new HashMap<>();
  >   
  >      @Override
  >      public boolean isZero() {
  >          return hcMap.isEmpty();
  >      }
  >   
  >      @Override
  >      public AccumulatorV2<Tuple2<String, String>, Map<String, HotCategory>> copy() {
  >          return new HotCategoryAccumulator();
  >      }
  >   
  >      @Override
  >      public void reset() {
  >          hcMap.clear();
  >      }
  >   
  >      @Override
  >      public void add(Tuple2<String, String> v) {
  >          final var cid = v._1;
  >          final var actionType = v._2;
  >          final var category = hcMap.getOrDefault(cid, new HotCategory(cid, 0, 0, 0));
  >          if (actionType == "click") {
  >              category.clickCnt += 1;
  >          } else if (actionType == "order") {
  >              category.orderCnt += 1;
  >          } else if (actionType == "pay") {
  >              category.payCnt += 1;
  >          }
  >          hcMap.put(cid, category);
  >      }
  >   
  >      @Override
  >      public void merge(AccumulatorV2<Tuple2<String, String>, Map<String, HotCategory>> other) {
  >          final var map1 = this.hcMap;
  >          final var map2 = other.value();
  >          map2.forEach((cid, hc) -> {
  >              HotCategory category = map1.getOrDefault(cid, new HotCategory(cid, 0, 0, 0));
  >              category.clickCnt += hc.clickCnt;
  >              category.orderCnt += hc.orderCnt;
  >              category.payCnt += hc.payCnt;
  >              map1.put(cid, category);
  >          });
  >      }
  >   
  >      @Override
  >      public Map<String, HotCategory> value() {
  >          return hcMap;
  >      }
  >   }
  >   ```

- 需求2：**Top10** 热门品类中每个品类的 **Top10** 活跃 **Session** 统计
  
  > ```java
  > import org.apache.spark.SparkConf;
  > import org.apache.spark.api.java.JavaPairRDD;
  > import org.apache.spark.api.java.JavaRDD;
  > import org.apache.spark.api.java.JavaSparkContext;
  > import org.apache.spark.api.java.function.FlatMapFunction;
  > import org.apache.spark.api.java.function.PairFunction;
  > import scala.Tuple2;
  > import scala.Tuple3;
  > 
  > import java.util.ArrayList;
  > import java.util.Arrays;
  > import java.util.List;
  > import java.util.stream.Collectors;
  > import java.util.stream.Stream;
  > 
  > public class HotCategoryTop10SessionAnalysis {
  >    public static void main(String[] args) {
  >        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis");
  >        JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
  >        JavaRDD<String> actionRDD = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/user_visit_action.txt");
  >        List<String> top10Ids = top10Category(actionRDD);
  >        //过滤保留前10品类的ID
  >        JavaPairRDD<String, Iterable<Tuple2<String, Integer>>> groupRDD = actionRDD.filter(it -> !"-1".equals(it.split("_")[6]) && top10Ids.contains(it.split("_")[6]))
  >                //切成<<品类，sessionID>,1>
  >                .mapToPair((PairFunction<String, Tuple2<String, String>, Integer>) it -> new Tuple2<>(new Tuple2<>(it.split("_")[6], it.split("_")[2]), 1))
  >                //计算<<品类，sessionID>,求和>
  >                .reduceByKey(Integer::sum)
  >                //计算<品类，<sessionId,求和>>
  >                .mapToPair(it -> new Tuple2(it._1()._1, new Tuple2<>(it._1()._2, it._2)))
  >                //按照key聚合
  >                .groupByKey();
  >        JavaPairRDD<String, List<Tuple2<String, Integer>>> resultRDD = groupRDD.mapValues(it -> {
  >            ArrayList<Tuple2<String, Integer>> sessions = new ArrayList<>();
  >            it.iterator().forEachRemaining(sessions::add);
  >            return sessions.stream().sorted((o1, o2) -> o2._2()- o1._2()).limit(10).collect(Collectors.toList());
  >        });
  >        resultRDD.collect().forEach(it -> System.out.println(it._1 + ":" + it._2));
  >    }
  > 
  >    private static List<String> top10Category(JavaRDD<String> actionRDD) {
  >        JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> flatRDD = actionRDD
  >                .flatMap(
  >                        (FlatMapFunction<String, Tuple2<String, Tuple3<Integer, Integer, Integer>>>) s -> {
  >                            String[] data = s.split("_");
  >                            if (!"-1".equals(data[6])) {
  >                                return Stream.of(data[6]).map(it -> new Tuple2<>(it, new Tuple3<>(1, 0, 0))).iterator();
  >                            }
  >                            if (!"null".equals(data[8])) {
  >                                return Arrays.stream(data[8].split(",")).map(it -> new Tuple2<>(it, new Tuple3<>(0, 1, 0))).iterator();
  >                            }
  >                            if (!"null".equals(data[10])) {
  >                                return Arrays.stream(data[10].split(",")).map(it -> new Tuple2<>(it, new Tuple3<>(0, 0, 1))).iterator();
  >                            }
  >                            //此处比较trick，scala可以返回空列表，java暂时没找到好的方法
  >                            return Stream.of(new String[]{"0"}).map(it -> new Tuple2<>(it, new Tuple3<>(0, 0, 0))).iterator();
  >                        }).mapToPair((PairFunction<Tuple2<String, Tuple3<Integer, Integer, Integer>>, String, Tuple3<Integer, Integer, Integer>>) it -> new Tuple2<>(it._1, it._2));
  >        JavaPairRDD<String, Tuple3<Integer, Integer, Integer>> analysisRDD = flatRDD.reduceByKey((t1, t2) -> new Tuple3<>(t1._1() + t2._1(), t1._2() + t2._2(), t1._3() + t2._3()));
  >        return analysisRDD.mapToPair(s -> new Tuple2<>(s._2, s._1))
  >                .sortByKey(new SparkTupleComparator())
  >                .take(10)
  >                .stream()
  >                .map(it -> it._2)
  >                .collect(Collectors.toList());
  >    }
  > }
  > ```

- 需求3：页面单跳转换率统计
  
  > 页面单跳转化率:
  > 计算页面单跳转化率，什么是页面单跳转换率，比如一个用户在一次 Session 过程中访问的页面路径 3,5,7,9,10,21，那么页面 3 跳到页面 5 叫一次单跳，7-9 也叫一次单跳， 那么单跳转化率就是要统计页面点击的概率。比如:计算 3-5 的单跳转化率，先获取符合条件的 Session 对于页面 3 的访问次数(PV) 为 A，然后获取符合条件的 Session 中访问了页面 3 又紧接着访问了页面 5 的次数为 B， 那么 B/A 就是 3-5 的页面单跳转化率。
  > 
  > ```java
  > import lombok.AllArgsConstructor;
  > import org.apache.spark.SparkConf;
  > import org.apache.spark.api.java.JavaPairRDD;
  > import org.apache.spark.api.java.JavaRDD;
  > import org.apache.spark.api.java.JavaSparkContext;
  > import org.apache.spark.api.java.function.FlatMapFunction;
  > import org.apache.spark.api.java.function.PairFunction;
  > import scala.Tuple2;
  > 
  > import java.io.Serializable;
  > import java.util.ArrayList;
  > import java.util.Comparator;
  > import java.util.List;
  > import java.util.Map;
  > import java.util.stream.Collectors;
  > 
  > //计算页面单跳转化率
  > public class PageflowAnalysis {
  >    public static void main(String[] args) {
  >        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("HotCategoryTop10Analysis");
  >        JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
  >        // 1. 读取原始日志数据
  >        JavaRDD<String> actionRDD = javaSparkContext.textFile("/Users/wyj/Desktop/learn/myProject/testSpark/src/main/resources/user_visit_action.txt");
  >        JavaRDD<UserVisitAction> actionDataRDD = actionRDD.map(it -> {
  >            String[] datas = it.split("_");
  >            return new UserVisitAction(
  >                    datas[0],
  >                    Long.parseLong(datas[1]),
  >                    datas[2],
  >                    Long.parseLong(datas[3]),
  >                    datas[4],
  >                    datas[5],
  >                    Long.parseLong(datas[6]),
  >                    Long.parseLong(datas[7]),
  >                    datas[8],
  >                    datas[9],
  >                    datas[10],
  >                    datas[11],
  >                    Long.parseLong(datas[12]));
  >        });
  >        actionDataRDD.cache();
  >        // 1-2,2-3,3-4,4-5,5-6,6-7
  >        List<Long> ids = List.of(1L, 2L, 3L, 4L, 5L, 6L, 7L);
  >        List<Tuple2<Long, Long>> okFlowIds = convertZipList(ids);
  >        //计算分母,就是对每个页面的访问次数的统计<页面id,访问次数>
  >        Map<Long, Long> pageidToCountMap = actionDataRDD.filter(it -> ids.contains(it.page_id))
  >                .mapToPair(it -> new Tuple2<>(it.page_id, 1L))
  >                .reduceByKey(Long::sum)
  >                .collectAsMap();
  > 
  >        //计算分子
  >        // 根据session进行分组
  >        JavaPairRDD<String, Iterable<UserVisitAction>> sessionRDD = actionDataRDD.groupBy(it -> it.session_id);
  >        //<session, <页面跳转，1>>
  >        JavaPairRDD<String, List<Tuple2<Tuple2<Long, Long>, Integer>>> mvRDD = sessionRDD.mapValues(it -> {
  >            ArrayList<UserVisitAction> sessions = new ArrayList<>();
  >            it.iterator().forEachRemaining(sessions::add);
  >            //将每个用户的访问按时间排序以后收集它的页面跳转关系
  >            List<Long> flowIds = sessions.stream()
  >                    .sorted(Comparator.comparing(o -> o.action_time))
  >                    .map(x -> x.page_id)
  >                    .collect(Collectors.toList());
  >            return convertZipList(flowIds).stream()
  >                    .filter(okFlowIds::contains)
  >                    .map(x -> new Tuple2<>(x, 1))
  >                    .collect(Collectors.toList());
  >        });
  >        JavaPairRDD<Tuple2<Long, Long>, Integer> dataRDD = mvRDD.map(it -> it._2)
  >                .flatMap((FlatMapFunction<List<Tuple2<Tuple2<Long, Long>, Integer>>, Tuple2<Tuple2<Long, Long>, Integer>>) it -> it.iterator())
  >                .mapToPair((PairFunction<Tuple2<Tuple2<Long, Long>, Integer>, Tuple2<Long, Long>, Integer>) it -> new Tuple2<>(it._1, 1))
  >                .reduceByKey(Integer::sum);
  >        //计算单跳转换率
  >        dataRDD.foreach(it -> System.out.format("页面%s跳转到页面%s单跳转换率为:%f\n", it._1()._1, it._1()._2, it._2 / (float)pageidToCountMap.getOrDefault(it._1()._1, 0L)));
  >        javaSparkContext.stop();
  >    }
  > 
  >    private static <T> List<Tuple2<T, T>> convertZipList(List<T> ids) {
  >        List<Tuple2<T, T>> okFlowIds = new ArrayList<>();
  >        for (int i = 0; i < ids.size() - 1; i++) {
  >            okFlowIds.add(new Tuple2<>(ids.get(i), ids.get(i + 1)));
  >        }
  >        return okFlowIds;
  >    }
  > }
  > 
  > @AllArgsConstructor
  > class UserVisitAction implements Serializable {
  >    public String date;//用户点击行为的日期
  >    public Long user_id;//用户的ID
  >    public String session_id;//Session的ID
  >    public Long page_id;//某个页面的ID
  >    public String action_time;//动作的时间点
  >    public String search_keyword;//用户搜索的关键词
  >    public Long click_category_id;//某一个商品品类的ID
  >    public Long click_product_id;//某一个商品的ID
  >    public String order_category_ids;//一次订单中所有品类的ID集合
  >    public String order_product_ids;//一次订单中所有商品的ID集合
  >    public String pay_category_ids;//一次支付中所有品类的ID集合
  >    public String pay_product_ids;//一次支付中所有商品的ID集合
  >    public Long city_id;//城市id
  > }
  > ```

- 需求4：各区域热门商品Top3
  
  > 数据（导入hive的格式）：
  > 
  > ```
  > user_visit_action：
  > 2019-07-17  95 26070e87-1ad7-49a3-8fb3-cc741facaddf   37 2019-07-17 00:00:02    手机 -1 -1 \N \N \N \N 3
  > 2019-07-17 95 26070e87-1ad7-49a3-8fb3-cc741facaddf   48 2019-07-17 00:00:10    \N 16 98 \N \N \N \N 19
  > 2019-07-17 95 26070e87-1ad7-49a3-8fb3-cc741facaddf   6  2019-07-17 00:00:17    \N 19 85 \N \N \N \N 7
  > city_info:
  > 1    北京    华北
  > 2    上海    华东
  > 3    深圳    华南
  > product_info:
  > 1    商品_1    自营
  > 2    商品_2    自营
  > 3    商品_3    自营
  > ```
  > 
  > ```java
  > package sql;
  > 
  > import org.apache.spark.SparkConf;
  > import org.apache.spark.sql.Encoder;
  > import org.apache.spark.sql.Encoders;
  > import org.apache.spark.sql.SparkSession;
  > import org.apache.spark.sql.expressions.Aggregator;
  > import org.apache.spark.sql.functions;
  > 
  > //https://blog.csdn.net/someby/article/details/103084807
  > public class SparkSQLHive {
  >    public static void main(String[] args) {
  >        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("sparkSQL");
  >        SparkSession sparkSession = SparkSession.builder().enableHiveSupport().config(sparkConf).getOrCreate();
  >        sparkSession.sql("use atguigu");
  >        sparkSession.sql("select a.*,p.product_name,c.area,c.city_name" +
  >                " from user_visit_action a" +
  >                " join product_info p on a.click_product_id = p.product_id" +
  >                " join city_info c on a.city_id = c.city_id" +
  >                " where a.click_product_id > -1").createOrReplaceTempView("t1");
  >        sparkSession.udf().register("cityRemark", functions.udaf(new CityRemarkUDAF(), Encoders.STRING()));
  >        sparkSession.sql("select area,product_name,count(*) as clickCnt,cityRemark(city_name) as city_remark from t1 group by area, product_name").createOrReplaceTempView("t2");
  >        sparkSession.sql("select *,rank() over(partition by area order by clickCnt desc) as rank from t2").createOrReplaceTempView("t3");
  >        sparkSession.sql("select * from t3 where rank<=3").show();
  >        sparkSession.close();
  >    }
  > }
  > 
  > //<输入数据类型，缓存区数据类型，输出数据类型>，这个udfa实际上也是求了count
  > class CityRemarkUDAF extends Aggregator<String, Integer, String> {
  > 
  >    @Override
  >    public Integer zero() {
  >        return 0;
  >    }
  > 
  >    @Override
  >    public Integer reduce(Integer b, String a) {
  >        return b+1;
  >    }
  > 
  >    @Override
  >    public Integer merge(Integer b1, Integer b2) {
  >        return b1+b2;
  >    }
  > 
  >    @Override
  >    public String finish(Integer reduction) {
  >        return String.valueOf(reduction);
  >    }
  > 
  >    @Override
  >    public Encoder<Integer> bufferEncoder() {
  >        return Encoders.INT();
  >    }
  > 
  >    @Override
  >    public Encoder<String> outputEncoder() {
  >        return Encoders.STRING();
  >    }
  > }
  > ```

- 实时需求用Scoket模拟Kafka，用hashmap模拟数据库
  
  > ```java
  > import java.io.IOException;
  > import java.io.OutputStream;
  > import java.net.ServerSocket;
  > import java.net.Socket;
  > import java.util.List;
  > 
  > //https://blog.csdn.net/ifenggege/article/details/108544451
  > public interface Mock {
  >    List<String> mockData();
  >    void printLog(List<String> lines);
  >    default void createData(int port) throws IOException, InterruptedException {
  >        ServerSocket socket = new ServerSocket(port);
  >        Socket clientSocket = socket.accept();
  >        OutputStream outputStream = clientSocket.getOutputStream();
  >        while(true){
  >            List<String> mockdata = mockData();
  >            printLog(mockdata);
  >            mockdata.forEach(it->{
  >                try {
  >                    outputStream.write(it.getBytes());
  >                } catch (IOException e) {
  >                    e.printStackTrace();
  >                }
  >            });
  >            Thread.sleep(1000);
  >        }
  >    }
  > }
  > ```
  > 
  > ```java
  > import scala.Tuple3;
  > 
  > import java.text.SimpleDateFormat;
  > import java.util.*;
  > 
  > public class MockData implements Mock {
  >    //模拟用户广告表
  >   HashMap<Tuple3<String, String, String>, Long> userAdCount = new HashMap<>();
  > 
  >    public List<String> mockData(){
  >        ArrayList<String> datas = new ArrayList<>();
  >        String[] users={"user1","user2","user3","user4","user5","user6"};
  >        String[] ads={"ad1","ad2","ad3","ad4","ad5","ad6"};
  >        String[] cities = {"beijing","shanghai","chengdu"};
  >        String[] areas={"huabei","dongbei","xibei"};
  >        for (int i=0;i<new Random().nextInt(50);i++) {
  >            String area = areas[new Random().nextInt(3)];
  >            String city = cities[new Random().nextInt(3)];
  >            String user = users[new Random().nextInt(6)] ;//6个用户
  >            String ad = ads[new Random().nextInt(6)] ;//6个用户
  >            datas.add(System.currentTimeMillis()+" "+area+" "+city+" "+user+" "+ad+"\r\n");
  >        }
  >        return datas;
  >    }
  > 
  >    public void printLog(List<String> lines){
  >        System.out.println("-----------LogStart-------------");
  >        lines.forEach(line->{
  >            System.out.println(line);
  >            String[] s = line.replace("\r\n","").split(" ");
  >            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
  >            String day = simpleDateFormat.format(new Date(Long.parseLong(s[0])));
  >            Tuple3<String, String, String> key = new Tuple3<>(day, s[3], s[4]);
  >            userAdCount.put(key,userAdCount.getOrDefault(key,0L)+1);
  >        });
  >        userAdCount.forEach((k,v)-> System.out.println(k._2()+"用户点击了"+k._3()+"广告"+v+"次"));
  >        System.out.println("-----------LogEnd-------------");
  >    }
  > }
  > ```
  > 
  > ```java
  > import lombok.extern.slf4j.Slf4j;
  > 
  > import java.io.IOException;
  > 
  > @Slf4j
  > public class Main {
  >    public static void main(String[] args) {
  >        try {
  >            Mock mock=new MockData();
  >            mock.createData(9999);
  >        } catch (IOException | InterruptedException e) {
  >            log.error("mock数据失败！");
  >        }
  >    }
  > }
  > ```
  > 
  > 调整Spark的日志输出位warn而不是info
  > 
  > ```properties
  > # Set everything to be logged to the console
  > log4j.rootCategory=WARN, console
  > log4j.appender.console=org.apache.log4j.ConsoleAppender
  > log4j.appender.console.target=System.err
  > log4j.appender.console.layout=org.apache.log4j.PatternLayout
  > log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n
  > 
  > # Settings to quiet third party logs that are too verbose
  > log4j.logger.org.spark-project.jetty=WARN
  > log4j.logger.org.spark-project.jetty.util.component.AbstractLifeCycle=ERROR
  > log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=INFO
  > log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=INFO
  > log4j.logger.org.apache.parquet=ERROR
  > log4j.logger.parquet=ERROR
  > 
  > # SPARK-9183: Settings to avoid annoying messages when looking up nonexistent UDFs in SparkSQL with Hive support
  > log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL
  > log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR
  > ```

- 需求5：广告黑名单
  
  > 实现实时的动态黑名单机制:将每天对某个广告点击超过 100 次的用户拉黑。
  > 
  > ```java
  > import lombok.AllArgsConstructor;
  > import org.apache.spark.SparkConf;
  > import org.apache.spark.api.java.JavaRDD;
  > import org.apache.spark.api.java.function.Function;
  > import org.apache.spark.streaming.Durations;
  > import org.apache.spark.streaming.api.java.JavaPairDStream;
  > import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
  > import org.apache.spark.streaming.api.java.JavaStreamingContext;
  > import scala.Tuple2;
  > import scala.Tuple3;
  > 
  > import java.text.SimpleDateFormat;
  > import java.util.Date;
  > import java.util.HashMap;
  > import java.util.HashSet;
  > import java.util.Set;
  > 
  > /*CREATE TABLE user_ad_count (
  >        dt varchar(255),
  >        userid CHAR (1),
  >        adid CHAR (1),
  >        count BIGINT,
  >        PRIMARY KEY (dt, userid, adid)
  >        );*/
  > public class BlackList {
  >    //模拟用户广告表
  >    static HashMap<Tuple3<String, String, String>, Long> userAdCount = new HashMap<>();
  >    //模拟黑名单表
  >    static Set<String> blackList = new HashSet<>();
  > 
  >    public static void main(String[] args) throws InterruptedException {
  >        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming");
  >        JavaStreamingContext javaStreamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(3));
  > 
  >        JavaReceiverInputDStream<String> lines = javaStreamingContext.socketTextStream("127.0.0.1", 9999);
  >        JavaPairDStream<Tuple3<String, String, String>, Long> ds = lines.map(it -> {
  >            String[] datas = it.split(" ");
  >            System.out.println(datas[0] + " " + datas[1] + " " + datas[2] + " " + datas[3] + " " + datas[4]);
  >            return new AdClickData(datas[0], datas[1], datas[2], datas[3], datas[4]);
  >        }).transform((Function<JavaRDD<AdClickData>, JavaRDD<Tuple2<Tuple3<String, String, String>, Long>>>) s ->
  >                s.filter(data -> !blackList.contains(data.user))
  >                        .mapToPair(data -> {
  >                            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
  >                            String day = simpleDateFormat.format(new Date(Long.parseLong(data.ts)));
  >                            return new Tuple2<>(new Tuple3<>(day, data.user, data.ad), 1L);
  >                        }).reduceByKey(Long::sum)
  >                        .map(it -> new Tuple2<>(it._1, it._2))
  >        ).mapToPair(it -> new Tuple2<>(it._1, it._2));
  >        //第一个each是一批数据，第二个each是对一批数据中的每一条
  >        ds.foreachRDD(it -> it.foreach(data -> {
  >            System.out.println("用户" + data._1._2() + "对广告" + data._1._3() + "的点击次数为：" + (userAdCount.getOrDefault(new Tuple3<>(data._1._1(), data._1._2(), data._1._3()), 0L) + 1L));
  >            if (data._2 >= 30) {//如果一批中的数据超过30的直接拉黑
  >                System.out.println(data._1._2() + "一批次的点击次数超过30，将被拉入黑名单");
  >                blackList.add(data._1._1());
  >            } else {//没超过30的先记到点击表
  >                Tuple3<String, String, String> key = new Tuple3<>(data._1._1(), data._1._2(), data._1._3());
  >                userAdCount.put(key, userAdCount.getOrDefault(key, 0L) + data._2);
  >                if (userAdCount.get(key) >= 30) {
  >                    System.out.println(data._1._2() + "当天的总点击次数超过30，将被拉入黑名单");
  >                    blackList.add(data._1._1());
  >                }
  >            }
  >        }));
  >        javaStreamingContext.start();
  >        javaStreamingContext.awaitTermination();
  >        javaStreamingContext.stop();
  >    }
  > }
  > 
  > @AllArgsConstructor
  > class AdClickData {
  >    public String ts;
  >    public String area;
  >    public String city;
  >    public String user;
  >    public String ad;
  > }
  > ```
  > 
  > 优化后的代码：
  > 
  > ```java
  > import org.apache.spark.SparkConf;
  > import org.apache.spark.api.java.JavaRDD;
  > import org.apache.spark.api.java.function.Function;
  > import org.apache.spark.api.java.function.VoidFunction;
  > import org.apache.spark.streaming.Durations;
  > import org.apache.spark.streaming.api.java.JavaPairDStream;
  > import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
  > import org.apache.spark.streaming.api.java.JavaStreamingContext;
  > import scala.Tuple2;
  > import scala.Tuple3;
  > 
  > import java.text.SimpleDateFormat;
  > import java.util.*;
  > 
  > public class BlackList2 {
  >    //模拟用户广告表
  >    static HashMap<Tuple3<String, String, String>, Long> userAdCount = new HashMap<>();
  >    //模拟黑名单表
  >    static Set<String> blackList = new HashSet<>();
  > 
  >    public static void main(String[] args) throws InterruptedException {
  >        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming");
  >        JavaStreamingContext javaStreamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(3));
  > 
  >        JavaReceiverInputDStream<String> lines = javaStreamingContext.socketTextStream("127.0.0.1", 9999);
  >        JavaPairDStream<Tuple3<String, String, String>, Long> ds = lines.map(it -> {
  >            String[] datas = it.split(" ");
  >            System.out.println(datas[0] + " " + datas[1] + " " + datas[2] + " " + datas[3] + " " + datas[4]);
  >            return new AdClickData(datas[0], datas[1], datas[2], datas[3], datas[4]);
  >        }).transform((Function<JavaRDD<AdClickData>, JavaRDD<Tuple2<Tuple3<String, String, String>, Long>>>) s ->
  >                s.filter(data -> !blackList.contains(data.user))
  >                        .mapToPair(data -> {
  >                            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
  >                            String day = simpleDateFormat.format(new Date(Long.parseLong(data.ts)));
  >                            return new Tuple2<>(new Tuple3<>(day, data.user, data.ad), 1L);
  >                        }).reduceByKey(Long::sum)
  >                        .map(it -> new Tuple2<>(it._1, it._2))
  >        ).mapToPair(it -> new Tuple2<>(it._1, it._2));
  >        /*这里的一个优化点使用了foreachPartition
  >        it.foreach方法会每一条数据创建一个连接
  >        foreach方法是RDD的算子，算子之外的代码是在Driver端执行，算子内的代码是在Executor端执行
  >        这样就会涉及闭包操作，Driver端的数据就需要传递到Executor端，需要将数据进行序列化，数据库的连接对象是不能序列化的。
  >        RDD提供了一个算子可以有效提升效率 : foreachPartition
  >        可以一个分区创建一个连接对象，这样可以大幅度减少连接对象的数量，提升效率*/
  >        ds.foreachRDD(it -> it.foreachPartition((VoidFunction<Iterator<Tuple2<Tuple3<String, String, String>, Long>>>) xx -> xx.forEachRemaining(data -> {
  >            System.out.println("用户" + data._1._2() + "对广告" + data._1._3() + "的点击次数为：" + (userAdCount.getOrDefault(new Tuple3<>(data._1._1(), data._1._2(), data._1._3()), 0L) + 1L));
  >            if (data._2 >= 30) {//如果一批中的数据超过30的直接拉黑
  >                System.out.println(data._1._2() + "一批次的点击次数超过30，将被拉入黑名单");
  >                blackList.add(data._1._1());
  >            } else {//没超过30的先记到点击表
  >                Tuple3<String, String, String> key = new Tuple3<>(data._1._1(), data._1._2(), data._1._3());
  >                userAdCount.put(key, userAdCount.getOrDefault(key, 0L) + data._2);
  >                if (userAdCount.get(key) >= 30) {
  >                    System.out.println(data._1._2() + "当天的总点击次数超过30，将被拉入黑名单");
  >                    blackList.add(data._1._1());
  >                }
  >            }
  >        })));
  >        javaStreamingContext.start();
  >        javaStreamingContext.awaitTermination();
  >        javaStreamingContext.stop();
  >    }
  > }
  > ```

- 需求6：广告点击量实时统计
  
  > 实时统计每天各地区各城市各广告的点击总流量
  > 
  > ```java
  > import org.apache.spark.SparkConf;
  > import org.apache.spark.api.java.function.VoidFunction;
  > import org.apache.spark.streaming.Durations;
  > import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
  > import org.apache.spark.streaming.api.java.JavaStreamingContext;
  > import scala.Tuple2;
  > import scala.Tuple4;
  > 
  > import java.text.SimpleDateFormat;
  > import java.util.Date;
  > import java.util.HashMap;
  > import java.util.Iterator;
  > 
  > public class ClcickCount {
  >    //模拟用户广告表
  >    static HashMap<Tuple4<String, String, String, String>, Long> areaCityAdCount = new HashMap<>();
  > 
  >    public static void main(String[] args) throws InterruptedException {
  >        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming");
  >        JavaStreamingContext javaStreamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(3));
  > 
  >        JavaReceiverInputDStream<String> lines = javaStreamingContext.socketTextStream("127.0.0.1", 9999);
  >        lines.map(it -> {
  >                    String[] datas = it.split(" ");
  >                    System.out.println(datas[0] + " " + datas[1] + " " + datas[2] + " " + datas[3] + " " + datas[4]);
  >                    return new AdClickData(datas[0], datas[1], datas[2], datas[3], datas[4]);
  >                }).mapToPair(it -> {
  >                    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
  >                    String day = simpleDateFormat.format(new Date(Long.parseLong(it.ts)));
  >                    return new Tuple2<>(new Tuple4<>(day, it.area, it.city, it.ad), 1L);
  >                }).reduceByKey(Long::sum)
  >                .foreachRDD(it -> {
  >                            it.foreachPartition((VoidFunction<Iterator<Tuple2<Tuple4<String, String, String, String>, Long>>>) xx ->
  >                                    xx.forEachRemaining(data -> {
  >                                        Tuple4<String, String, String, String> key = new Tuple4<>(data._1._1(), data._1._2(), data._1._3(), data._1._4());
  >                                        areaCityAdCount.put(key, areaCityAdCount.getOrDefault(key, 0L) + data._2);
  >                                    })
  >                            );
  >                            areaCityAdCount.forEach((k, v) -> System.out.println(k._1() + " " + k._2() + " " + k._3() + " " + k._4() + " " + v));
  >                        }
  >                );
  >        javaStreamingContext.start();
  >        javaStreamingContext.awaitTermination();
  >        javaStreamingContext.stop();
  >    }
  > }
  > ```

- 需求7：
  
  > 最近一小时广告点击量
  > 
  > ```java
  > import org.apache.spark.SparkConf;
  > import org.apache.spark.streaming.Durations;
  > import org.apache.spark.streaming.api.java.JavaPairDStream;
  > import org.apache.spark.streaming.api.java.JavaReceiverInputDStream;
  > import org.apache.spark.streaming.api.java.JavaStreamingContext;
  > import scala.Tuple2;
  > 
  > public class Windows {
  >    public static void main(String[] args) throws InterruptedException {
  >        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("SparkStreaming");
  >        //此处注意滑动距离必须是批量执行的整数倍
  >        JavaStreamingContext javaStreamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(5));
  >        JavaReceiverInputDStream<String> lines = javaStreamingContext.socketTextStream("127.0.0.1", 9999);
  >        //每隔10秒统计前60秒的数据
  >        JavaPairDStream<Long, Long> reduceDS = lines.map(it -> {
  >                    String[] datas = it.split(" ");
  >                    return new AdClickData(datas[0], datas[1], datas[2], datas[3], datas[4]);
  >                }).mapToPair(it -> new Tuple2<>(Long.parseLong(it.ts) / 10000 * 10000, 1L))
  >                .reduceByKeyAndWindow(Long::sum, Durations.seconds(60), Durations.seconds(10));
  >        reduceDS.print();
  >        javaStreamingContext.start();
  >        javaStreamingContext.awaitTermination();
  >        javaStreamingContext.stop();
  >    }
  > }
  > ```

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
