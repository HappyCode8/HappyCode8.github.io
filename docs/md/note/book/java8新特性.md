# java8新特性

```
《Java 8实战》
[英]厄马（Raoul-Gabriel Urma）
[意] 弗斯科（Mario Fusco）
[英] 米克罗夫特（Alan Mycroft）
```

## 第二章

<font color='red'>筛选绿苹果，很简单</font>

```java
public static List<Apple> filterGreenApples(List<Apple> inventory) {
    List<Apple> result = new ArrayList<>();    ←─累积苹果的列表
    for(Apple apple: inventory){
        if( "green".equals(apple.getColor() ) {    ←─仅仅选出绿苹果
            result.add(apple);
        }
    }
    return result;
}
```

<font color='red'>但是可能还要筛选各种颜色的苹果，那就加个参数，像下面这样</font>

```java
public static List<Apple> filterGreenApples(List<Apple> inventory, String color) {
   ...//代码类似上边
}
```

<font color='red'>但是可能还要筛选重量，类似于上面，复制粘贴修改一下</font>

```java
public static List<Apple> filterApplesByWeight(List<Apple> inventory, int weight) {
    ...//代码略
}
```

<font color='red'>如果你此刻想要改变筛选遍历方式来提升性能呢？那就得修改上面所有方法的实现，而不是只改一个</font>

<font color='green'>有一个解决办法是把两个方法结合起来，类似下面这样，用flag标志筛选哪个</font>

```java
public static List<Apple> filterApples(List<Apple> inventory, String color,
                                       int weight, boolean flag) {
  //略
}
```

<font color='red'>但是这么设计仍然有缺陷，flag意思不好区分，当客户端有更多属性而且要组合查询时，就没法了</font>

### 行为参数化

有没有办法把筛选条件传进去？可以这么做：首先定义一个筛选的接口，接口中定义一个筛选方法，然后多种类实现这个接口，各自判断自己的筛选条件（**策略模式,算法族就是ApplePredicate，不同的策略就是AppleHeavyWeightPredicate和AppleGreenColorPredicate**），代码类似下面这样

```java
public interface ApplePredicate{
    boolean test (Apple apple);
}

public class AppleHeavyWeightPredicate implements ApplePredicate{    ←─仅仅选出重的苹果
    public boolean test(Apple apple){
        return apple.getWeight() > 150;
 }
}
public class AppleGreenColorPredicate implements ApplePredicate{    ←─仅仅选出绿苹果
    public boolean test(Apple apple){
        return "green".equals(apple.getColor());
    }
}

public static List<Apple> filterApples(List<Apple> inventory,//客户端代码
                                       ApplePredicate p){
    List<Apple> result = new ArrayList<>();
    for(Apple apple: inventory){
        if(p.test(apple)){    ←─谓词对象封装了测试苹果的条件
            result.add(apple);
        }
    }
    return result;
}

```

### 测验

写一个不同的打印苹果的方法，客户端可以按需求打印苹果

```java
public interface AppleFormatter{
    String accept(Apple a);
}
```

<font color='red'>上边的写法仍然有缺陷，因为筛选条件只用一次，但是仍然不得不实例化很多代码，比如实例化了AppleHeavyWeightPredicate和AppleGreenColorPredicate这些类，可以使用匿名类实现，比如像下面这样，用到的时候实例化</font>

```java
List<Apple> redApples = filterApples(inventory, new ApplePredicate() {  ←─直接内联参数化filterapples方法的行为
    public boolean test(Apple apple){
        return "red".equals(apple.getColor());
    }
});
```

<font color='red'>但是匿名类仍然有缺陷，首先是格式看起来笨重，其次是有时候理解让人费解，可以使用如下代码来解决这个问题</font>

```java
List<Apple> result =
    filterApples(inventory, (Apple apple) -> "red".equals(apple.getColor()));
```

<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FUUFUQWpjeGVEMVB2YjRiVUZfeFpLa0JrUEx0QmJXQVFxRnRuYW8tQ0hjeHV3.png" alt="block" style="zoom:50%;" />

<font color='red'>以上代码已经可以解决问题，但是到目前为止只能解决苹果问题，而不能解决别的问题，可以使用如下代码进一步解决</font>

```java
public interface Predicate<T>{
    boolean test(T t);
}

public static <T> List<T> filter(List<T> list, Predicate<T> p){    ←─引入类型参数T
    List<T> result = new ArrayList<>();
    for(T e: list){
        if(p.test(e)){
            result.add(e);
        }
    }
    return result;
}
```

这样这段代码使用的范围更加广泛了

```java
List<Apple> redApples =
    filter(inventory, (Apple apple) -> "red".equals(apple.getColor()));

List<Integer> evenNumbers =
    filter(numbers, (Integer i) -> i % 2 == 0);
```

## 第三章

### Lambda表达式

一个完整的lambda表达式如下所示

![image-20201206174540974](https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FU1p5MTZDVXJEdEVuU1FIN2pENTFLSUIxNlNScm9GM1JiSHBqV3p6WEt5Rmt3.png)

 ### 测验

下面哪些表达式不是有效的lambda表达式

```java
(1) () -> {}
(2) () -> "Raoul"
(3) () -> {return "Mario";}
(4) (Integer i) -> return "Alan" + i;
(5) (String s) -> {"IronMan";}
```

<font color='red'>4,5不是，4应该带大括号，5应该去掉大括号</font>

<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FVUhPN1ZNTzhNWkVuZEVMQkhldDRaMEIxWk80b1NZalFBeTlHWmFLRXhfTXhR.png" alt="block" style="zoom:50%;">

### 什么时候使用Lambda表达式

<font color='red'>可以在函数式接口上使用Lambda表达式，函数式接口就是只定义一个抽象方法的接口，一定是只定义了一个，哪怕有很多默认方法，只要接口只定义了一个抽象方法。</font>

### 测验

下面哪些是函数式接口？

```java
public interface Adder{
    int add(int a, int b);
}
public interface SmartAdder extends Adder{
    int add(double a, double b);
}
public interface Nothing{
}
```

<font color='red'>只有1，因为2有两个抽象方法（继承了一个），3一个都没有</font>

### Lamda 实例

读取操作一个文件

```java
public static String processFile(){
        try(var br=new BufferedReader(new FileReader("/Users/wyj/Desktop/data.txt"))){
            return br.readline();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
```

上述代码操作不够灵活，比如要实现读取两行，就需要改代码，下面对其进行Labda改造

- 首先要进行**行为参数化**，类似于下面这样调用它

  ```java
  processFile((BufferedReader br)->br.readLine()+br.readLine())
  ```

- 那么就要在`processFile`中传递行为，那么久需要一个函数式接口

  ```java
  @FunctionalInterface
  public interface BufferedReaderProcessor {
      String process(BufferedReader br) throws IOException;
  }
  ```

- 把这个接口作为参数传递给`processFile`

  ```java
  public static String processFile(BufferedReaderProcessor p){
          try(var br=new BufferedReader(new FileReader("/Users/wyj/Desktop/data.txt"))){
              return p.process(br);
          } catch (IOException e) {
              e.printStackTrace();
          }
          return null;
      }
  ```

### java8常见的函数式接口

- Predicate，接收泛型T对象，返回一个boolean

  ```java
  @FunctionalInterface
  public interface Predicate<T>{
      boolean test(T t);
  }
  ```

  ```java
  public static boolean filter(Predicate<String> p, String str){
          return p.test(str);
  }
  public static void main(String[] args) {
    System.out.println(filter((String s)->s.equals("123"), "123"));
  }
  ```

- Consumer，接收泛型T对象，返回一个void

  ```java
  @FunctionalInterface
  public interface Consumer<T>{
      void accept(T t);
  }
  ```

  ```java
  public static <T> void print(Consumer<T> c, T x){
      c.accept(x);
  }
  public static void main(String[] args) {
   	 	print((String s)-> System.out.println(s),"qqw");
  }
  ```

- Function，接收泛型T对象，返回泛型R对象

  ```java
  @FunctionalInterface
  public interface Function<T, R>{
      R apply(T t);
  }
  ```

  ```java
  public static <T, R> R getLength(Function<T, R> f, T t){
          return f.apply(t);
      }
  public static void main(String[] args) {
      		System.out.println(getLength(((String s)->s.length()),"123"));
  }
  ```

### 原始类型特化

装箱后的值本质上就是把原始类型包裹起来，并保存在堆里。因此，装箱后的值需要更多的内存，并需要额外的内存搜索来获取被包裹的原始值，函数式接口带来了一个专门的版本，以便在输入和输出都是原始类型时避免自动装箱的操作。一般来说，针对专门的输入参数类型的函数式接口的名称都要加上对应的原始类型前缀，比如DoublePredicate、IntConsumer、LongBinaryOperator、IntFunction等。

```java
IntPredicate evenNumbers = (int i) -> i % 2 == 0;
System.out.println(evenNumbers.test(1000));
```

### 常用函数式接口
<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FYmhkME1oYmYzRklzNWgzWUM0My1lMEJ0QnlfN2J1NWZYdE5tbmpzdmpDOXBR.png" style="zoom:35%;">
<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FYkgxMzhsNGNnVkdtTkNIYnkzVlJjRUJQWGhoRE5lWTB1aGdDdHVuTGdacHpn.png" style="zoom:40%;">

### 异常处理

函数式接口都不允许抛出受检异常，有两种办法来处理Lambda抛出的异常，一种是自己定义一个函数式接口，另一种是把Lambda表达式包含在一个try/catch块中。

第一种

```java
@FunctionalInterface
public interface BufferedReaderProcessor {
    String process(BufferedReader b) throws IOException;
}
BufferedReaderProcessor p = (BufferedReader br) -> br.readLine();
```

第二种,对于API使用这种

```java
Function<BufferedReader, String> f = (BufferedReader b) -> {
    try {
        return b.readLine();
    }
    catch(IOException e) {
        throw new RuntimeException(e);
    }
}
```

### 类型检查

```java
List<Apple> heavierThan150g = filter(inventory, (Apple a) -> a.getWeight() > 150);
```

<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FYTE3UVkwRFBGNUZ2Z1d2dEhzbDZuOEIxVXU0M0Vtd2haTFlNV0NZNzZTREpB.png" style="zoom:50%">

### 特殊的void兼容规则

​        如果一个Lambda的主体是一个语句表达式， 它就和一个返回void的函数描述符兼容（当然需要参数列表也兼容）。例如，以下两行都是合法的，尽管List的add方法返回了一个boolean，而不是Consumer上下文（T -> void）所要求的void.

```java
// Predicate返回了一个boolean
Predicate<String> p = s -> list.add(s);
// Consumer返回了一个void
Consumer<String> b = s -> list.add(s);
```

### 类型推断

```java
List<Apple> greenApples =
    filter(inventory, a -> "green".equals(a.getColor()));    ←─参数a没有显式类型
```

```java
Comparator<Apple> c =
    (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());    ←─没有类型推断

Comparator<Apple> c =
    (a1, a2) -> a1.getWeight().compareTo(a2.getWeight());    ←─有类型推断
```

### 使用局部变量

可以使用局部变量，但是变量必须是final或者事实上是final的，以下是可以的

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);
```

但是这样不可以:

```java
int portNumber = 1337;
Runnable r = () -> System.out.println(portNumber);    ←─错误：Lambda表达式引用的局部变量必须是最终的（final）或事实上最终的
portNumber = 31337;
```

### 方法引用

方法引用就是Lambda表达式(Apple a) -> a.getWeight()的快捷写法，Apple::getWeight就是引用了Apple类中定义的方法getWeight。

方法引用主要有三类：

1. 指向静态方法的方法引用（例如Integer的parseInt方法，写作Integer::parseInt）。

2. 指向任意类型实例方法的方法引用（例如String的length方法，写作String::length）。

3. 指向现有对象的实例方法的方法引用（假设你有一个局部变量expensiveTransaction用于存放Transaction类型的对象，它支持实例方法getValue，那么你就可以写expensiveTransaction::getValue）。”

2与3的区别在于，2是你在引用一个对象的方法，而这个对象本身是Lambda对象的一个参数，例如`Lambda表达式(String s) -> s.toUppeCase()可以写作String::toUpperCase`，3是在Lambda中调用一个已经存在的外部对象中的方法，例如`()->expensiveTransaction.getValue()`

<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FYS1zdjlPNzFjUk1nZ19IZHJWM19Ra0IwSmE2dEJzZlVMaFpaTUxOYmZvd3Z3.png" style="zoom:50%;" />

### 

### 比较器复合

1. 逆序

   ```java
    List.of().sort(comparing(Apple::getWeight).reversed());
   ```

2. 比较器链

   ```java
    List.of().sort(comparing(Apple::getWeight)
                   .reversed()
                   .thenComparing(Apple::getCountry));
   ```

### 谓词复合

1. 使用negate方法来返回一个Predicate的非

```
Predicate<Apple> notRedApple = redApple.negate();
```

2. and方法的组合

```java
Predicate<Apple> redAndHeavyApple =
    redApple.and(a -> a.getWeight() > 150); 
```

```java
Predicate<Apple> redAndHeavyAppleOrGreen =
    redApple.and(a -> a.getWeight() > 150)
            .or(a -> "green".equals(a.getColor()));
```

### 函数复合

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.andThen(g);  
int result = h.apply(1);//4
```

```java
Function<Integer, Integer> f = x -> x + 1;
Function<Integer, Integer> g = x -> x * 2;
Function<Integer, Integer> h = f.compose(g); 
int result = h.apply(1);//3
```

## 第四章

### 流只能消费一次

```java
List<String> title = Arrays.asList("Java8", "In", "Action");
Stream<String> s = title.stream();
s.forEach(System.out::println);
s.forEach(System.out::println);
```

## 第五章

<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FWUNDdkhyWldQSkx1bHVjUHJFby1rb0J5QXlsZ09mMzk0OTllXy12RE5nRG13.png" style="zoom:40%;">

### 原始类型流特化

映射到数值流

```java
menu.stream().mapToInt(Dish::getCalories).sum()
```

转换回对象流

```java
menu.stream().mapToInt(Dish::getCalories).boxed()
```

默认值OptionalInt，下面代码如果没有默认值的话就可以显示调用orElse指定一个

```java
menu.stream().mapToInt(Dish::getCalories).max()
```

数值范围

```java
IntStream.rangeClosed(1, 100)//生成1-100的闭区间数
IntStream.range(1, 100)//开区间
```

## 第六章

```java
menu.stream().collect(Collectors.toList())//收集为列表
menu.stream().collect(Collectors.counting())//收集数目,也可以menu.stream().count()
```

### 寻找最大或者最小值

```java
Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream().collect(Collectors.maxBy(dishCaloriesComparator));
```

### 求和、求平均

```java
menu.stream().collect(summingInt(Dish::getCalories));//求和
menu.stream().collect(averagingInt(Dish::getCalories));//求平均
IntSummaryStatistics menuStatistics = menu.stream().collect(Collectors.summarizingInt(Dish::getCalories));//一次性求出所有的，包括最大、最小、和、平均、数目
```

### 拼接字符串

```java
menu.stream().map(Dish::getName).collect(joining());//joining里边可以带分隔符
menu.stream().collect(joining());//这个用到了类里边的toString方法
```

### reduce与collect方法对比

reduce方法旨在把两个值结合起来生成一个新值，它是一个不可变的归约。与此相反，collect方法的设计就是要改变容器，从而累积要输出的结果。

### 分组

```java
menu.stream().collect(groupingBy(Dish::getType));
```

### 超越属性访问器的分组

```java
Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
        groupingBy(dish -> {
               if (dish.getCalories() <= 400) return CaloricLevel.DIET;
               else if (dish.getCalories() <= 700) return
    CaloricLevel.NORMAL;
        else return CaloricLevel.FAT;
         } ));
```

### 多级分组

```java
Map<Dish.Type, Map<CaloricLevel, List<Dish>>> dishesByTypeCaloricLevel = menu.stream().collect(
      groupingBy(Dish::getType,    ←─一级分类函数
         groupingBy(dish -> {    ←─二级分类函数
            if (dish.getCalories() <= 400) return CaloricLevel.DIET;
                else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
           else return CaloricLevel.FAT;
          } )
      )
);
```

### 按子组收集数据

```java
menu.stream().collect(groupingBy(Dish::getType, counting()));//统计子组有多少数目
Map<Dish.Type, Optional<Dish>> mostCaloricByType=menu.stream().collect(groupingBy(Dish::getType,maxBy(comparingInt(Dish::getCalories))));//统计子组中最大的
```

### 收集器结果转换为另一种类型

```java
Map<Dish.Type, Dish> mostCaloricByType = menu.stream()
                .collect(groupingBy(Dish::getType,    ←─分类函数
                 collectingAndThen(
                     maxBy(comparingInt(Dish::getCalories)),    ←─包装后的收集器
                 Optional::get))); 
```

<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FZUk5MkFTZndIWkV1TzRtWkpqY043OEJFVWdMZWNpR2g1WUNURUZrV2lycEZB.png" style="zoom:50%;">

## 第七章

### 顺序流转换为并行流

```java
Stream.iterate(1L, i -> i + 1)
                 .limit(n)
                 .parallel()    ←─将流转换为并行流
                 .reduce(0L, Long::sum);
```

### 并行流转换为顺序流

```java
stream.parallel().sequential()
```

可以通过系统属性java.util.concurrent.ForkJoinPool.common. parallelism来改变线程池大小，但是不建议这么做，而且改变的是所有使用并行流的线程池，如果没有很好地理由，不建议这么做。

```java
System.setProperty("java.util.concurrent.ForkJoinPool.common. parallelism")
```

## 第八章

### 策略模式

```java
public interface ValidationStrategy {//接口
    boolean execute(String s);
}
public class IsAllLowerCase implements ValidationStrategy {//对接口的实现
    public boolean execute(String s){
        return s.matches("[a-z]+");
    }
}

public class IsNumeric implements ValidationStrategy {
    public boolean execute(String s){
        return s.matches("\\d+");
    }
}

public class Validator{//校验器
    private final ValidationStrategy strategy;

    public Validator(ValidationStrategy v){
        this.strategy = v;
    }

    public boolean validate(String s){
        return strategy.execute(s);
    }
}

Validator numericValidator = new Validator(new IsNumeric());//客户端
boolean b1 = numericValidator.validate("aaaa");    ←─返回false
Validator lowerCaseValidator = new Validator(new IsAllLowerCase ());
boolean b2 = lowerCaseValidator.validate("bbbb");    ←─返回true”
```

使用Lambda表达式重构,避免了写接口的实现

```java
Validator numericValidator = new Validator((String s)->s.matches("[a-z]+"));//客户端
boolean b1 = numericValidator.validate("aaaa");    ←─返回false
Validator lowerCaseValidator = new Validator((String s)->s.matches("[a-z]+"));
boolean b2 = lowerCaseValidator.validate(s.matches("\\d+"));    ←─返回true
```

### 模板方法

```java
abstract class OnlineBanking {

    public void processCustomer(int id){
        Customer c = Database.getCustomerWithId(id);
        makeCustomerHappy(c);
    }

    abstract void makeCustomerHappy(Customer c);
}
```

使用Lambda表达式

```java
public void processCustomer(int id, Consumer<Customer> makeCustomerHappy){
    Customer c = Database.getCustomerWithId(id);
    makeCustomerHappy.accept(c);
}

new OnlineBankingLambda().processCustomer(1337, (Customer c) ->
    System.out.println("Hello " + c.getName());
```

### 观察者模式

```java
interface Observer {
    void notify(String tweet);
}

class NYTimes implements Observer{
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("money")){
            System.out.println("Breaking news in NY! " + tweet);
        }
    }
}
class Guardian implements Observer{
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("queen")){
            System.out.println("Yet another news in London... " + tweet);
        }
    }
}
class LeMonde implements Observer{
    public void notify(String tweet) {
        if(tweet != null && tweet.contains("wine")){
            System.out.println("Today cheese, wine and news! " + tweet);
        }
    }
}

interface Subject{
    void registerObserver(Observer o);
    void notifyObservers(String tweet);
}
class Feed implements Subject{

    private final List<Observer> observers = new ArrayList<>();

    public void registerObserver(Observer o) {
        this.observers.add(o);
    }

    public void notifyObservers(String tweet) {
        observers.forEach(o -> o.notify(tweet));
    }
}

Feed f = new Feed();
f.registerObserver(new NYTimes());
f.registerObserver(new Guardian());
f.registerObserver(new LeMonde());
f.notifyObservers("The queen said her favourite book is Java 8 in Action!");
```

使用Lambda表达式实现：

```java
f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("money")){
        System.out.println("Breaking news in NY! " + tweet);
    }
});

f.registerObserver((String tweet) -> {
    if(tweet != null && tweet.contains("queen")){
        System.out.println("Yet another news in London... " + tweet);
    }
});
```

### 责任链模式

```java
public abstract class ProcessingObject<T> {

    protected ProcessingObject<T> successor;
    public void setSuccessor(ProcessingObject<T> successor){
        this.successor = successor;
    }

    public T handle(T input){
        T r = handleWork(input);
        if(successor != null){
            return successor.handle(r);
        }
        return r;
    }

    abstract protected T handleWork(T input);
}

public class HeaderTextProcessing extends ProcessingObject<String> {
    public String handleWork(String text){
        return "From Raoul, Mario and Alan: " + text;
    }
}

public class SpellCheckerProcessing extends ProcessingObject<String> {
    public String handleWork(String text){
        return text.replaceAll("labda", "lambda");    ←─糟糕，我们漏掉了Lambda中的m字符
    }
}

ProcessingObject<String> p1 = new HeaderTextProcessing();
ProcessingObject<String> p2 = new SpellCheckerProcessing();

p1.setSuccessor(p2);                               ←─将两个处理对象链接起来

String result = p1.handle("Aren't labdas really sexy?!!");
System.out.println(result);
```

使用Lambda表达式

```java
UnaryOperator<String> headerProcessing =
    (String text) -> "From Raoul, Mario and Alan: " + text;    ←─第一个处理对象

UnaryOperator<String> spellCheckerProcessing =
    (String text) -> text.replaceAll("labda", "lambda");    ←─第二个处理对象

Function<String, String> pipeline =
    headerProcessing.andThen(spellCheckerProcessing);    ←─将两个方法结合起来，结果就是一个操作链

String result = pipeline.apply("Aren't labdas really sexy?!!");
```

工厂模式

```java
public class ProductFactory {
    public static Product createProduct(String name){
        switch(name){
            case "loan": return new Loan();
            case "stock": return new Stock();
            case "bond": return new Bond();
            default: throw new RuntimeException("No such product " + name);
        }
    }
}

Product p = ProductFactory.createProduct("loan");
```

使用Lmbda表达式

```java
final static Map<String, Supplier<Product>> map = new HashMap<>();
static {
    map.put("loan", Loan::new);
    map.put("stock", Stock::new);
    map.put("bond", Bond::new);
}

public static Product createProduct(String name){
    Supplier<Product> p = map.get(name);
    if(p != null) return p.get();
    throw new IllegalArgumentException("No such product " + name);
}
```

### peek方法

流提供的peek方法在分析Stream流水线时，能将中间变量的值输出到日志中，是非常有用的工具。

```java
List<Integer> result =
  numbers.stream()
         .peek(x -> System.out.println("from stream: " + x))    ←─输出来自数据源的当前元素值
         .map(x -> x + 17)
         .peek(x -> System.out.println("after map: " + x))    ←─输出map 操作的结果
         .filter(x -> x % 2 == 0)
         .peek(x -> System.out.println("after filter: " + x))    ←─输出经过filter操作之后，剩下的元素个数
         .limit(3)
         .peek(x -> System.out.println("after limit: " + x))    ←─输出经过limit操作之后，剩下的元素个数
         .collect(toList());
```

![image-20201219200622976](<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FUlJocV9OemZqQk91emZTNmVsQjNxWUJHTHRJaEp6S185RFF4RUM3VGJEbGl3.png">)

## 第十章

解决空指针的方法

- 深层质疑

  <img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FYnV1VWFhWmRONURrbGp3dDZsd3F3Z0JNak1JckIzUDVadUJrUDZtLWJnNGJB.png" style="zoom:50%;">

- 多层退出

  <img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FWnBRS1NRbElybEttSUU1dWo1MXRUWUJIM2lGUkMyV2E3VWs4NXVpNUQwX3pB.png" style="zoom:50%;">

```java
public int readDuration(Properties props, String name) {
    return Optional.ofNullable(props.getProperty(name))
                   .flatMap(OptionalUtility::stringToInt)
                   .filter(i -> i > 0)
                   .orElse(0);
}
```

## 第十一章

Future接口

```java
public class FutureTest {
    public static Double test() throws InterruptedException {
        ExecutorService executor = Executors.newCachedThreadPool();
        Future<Double> future = executor.submit(new Callable<Double>() {
            public Double call() throws InterruptedException {
                Thread.sleep(3000);
                return 2D;
            }});
        Thread.sleep(1000);
        try {
            Double result = future.get(1, TimeUnit.SECONDS);
            return result;
        } catch (ExecutionException ee) {
            // 计算抛出一个异常
        } catch (InterruptedException ie) {
            // 当前线程在等待过程中被中断
        } catch (TimeoutException te) {
            // 在Future对象完成之前超过已过期
            System.out.println("超时");
        }
        return null;
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println(test());
    }
}
```

CompletableFuture接口

```java
public class Shop {
    public Future<Double> getPriceAsync(String product) {
        /*CompletableFuture<Double> futurePrice=new CompletableFuture<>();
        new Thread(()->{try {
            double price=calculatePrice(product);
            futurePrice.complete(price);
        }catch (Exception e) {
            futurePrice.completeExceptionally(e);
        }
        }).start();
        return futurePrice;*/
        return CompletableFuture.supplyAsync(() -> calculatePrice(product));
    }

    private double calculatePrice(String product) {
        delay();
        var random=new Random();
        return random.nextDouble() * product.charAt(0) + product.charAt(1);
    }

    public static void delay() {
        try {
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        Shop shop = new Shop();
        long start = System.nanoTime();
        Future<Double> futurePrice = shop.getPriceAsync("my favorite product");
        long invocationTime = ((System.nanoTime() - start) / 1_000_000);
        System.out.println("Invocation returned after " + invocationTime + " msecs");
        try {
            double price = futurePrice.get();
            System.out.printf("Price is %.2f%n", price);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        long retrievalTime = ((System.nanoTime() - start) / 1_000_000);
        System.out.println("Price returned after " + retrievalTime + " msecs");
    }
}

```

### 使用并行流还是CompletableFutures：

使用并行流进行请求

```java
public static List<String> findPrices(String product) {
        List<Shop> shops = Arrays.asList(new Shop("BestPrice"),
                new Shop("LetsSaveBig"),
                new Shop("MyFavoriteShop"),
                new Shop("BuyItAll"),
                new Shop("LetsSaveBig"),
                new Shop("MyFaiteShop"),
                new Shop("BuyzaItAll"),
                new Shop("LeasastsSaveBig"));
        return shops.parallelStream()
                .map(shop -> String.format("%s price is %.2f",
                        shop.getName(), shop.getPrice(product)))
                .collect(Collectors.toList());
    }
```

使用CompletableFutures进行请求

```java
public static List<String> findPrices(String product) {
        List<Shop> shops = Arrays.asList(new Shop("BestPrice"),
                new Shop("LetsSaveBig"),
                new Shop("MyFavoriteShop"),
                new Shop("BuyItAll"),
                new Shop("LetsSaveBig"),
                new Shop("MyFaiteShop"),
                new Shop("BuyzaItAll"),
                new Shop("LeasastsSaveBig"));
        List<CompletableFuture<String>> priceFutures=shops.stream()
                .map(shop->CompletableFuture.supplyAsync(
                        ()->shop.getName() + " price is " +
                                shop.getPrice(product)
                )).collect(Collectors.toList());
        return priceFutures.stream().map(CompletableFuture::join).collect(Collectors.toList());//多用一个流
    }
```

<img src="https://link.jscdn.cn/sharepoint/aHR0cHM6Ly8xZHJpdi1teS5zaGFyZXBvaW50LmNvbS86aTovZy9wZXJzb25hbC9zdG9yXzFkcml2X29ubWljcm9zb2Z0X2NvbS9FUVE1ZXBrQWZlWkZ1SUY4Z1dfZFBOZ0JwV1RLWV9ldFhfbXg2SWVLU0V3SnVB.png" style="zoom:50%;">

- 如果你进行的是计算密集型的操作，并且没有I/O，那么推荐使用Stream接口，因为实现简单，同时效率也可能是最高的（如果所有的线程都是计算密集型的，那就没有必要创建比处理器核数更多的线程）。

- 如果你并行的工作单元还涉及等待I/O的操作（包括网络连接等待），那么使用CompletableFuture灵活性更好，你可以像前文讨论的那样，依据等待/计算，或者W/C的比率设定需要使用的线程数。这种情况不使用并行流的另一个原因是，处理流的流水线中如果发生I/O等待，流的延迟特性会让我们很难判断到底什么时候触发了等待。


### 如何调整线程池的大小

线程数 = 处理器核的数目 * 期望的CPU利用率 * (1 + 等待时间/计算时间)

- 处理器核的数目是指`System.out.println(Runtime.getRuntime().availableProcessors())`，它返回的是可用的计算资源，而不是CPU物理核心数，对于支持超线程的CPU来说，单个物理处理器相当于拥有两个逻辑处理器，能够同时执行两个线程。
- 等待时间一般指IO时间

来自：Java并发编程实战

```java
private final Executor executor =
        Executors.newFixedThreadPool(Math.min(shops.size(), 100), new ThreadFactory() {
            public Thread newThread(Runnable r) {
                Thread t = new Thread(r);
                t.setDaemon(true);    ←─使用守护线程——这种方式不会阻止程序的关停
                return t;
            }
});

CompletableFuture.supplyAsync(() -> shop.getName() + " price is " +  shop.getPrice(product), executor);
```

## 第十三章

命令式编程：指令和计算机底层的词汇非常相近

声明式编程：关注于要做什么

## 第十四章

科里化：

```java
static double converter(double x, double f, double b) {
    return x * f + b;
}
```

```java
static DoubleUnaryOperator curriedConverter(double f, double b){
    return (double x) -> x * f + b;
}
```

```java
curriedConverter(1,2).applyAsDouble(3)
```

