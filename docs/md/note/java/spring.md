

# Small-Spring

todo: 这是小傅哥的那个手撸spring的笔记

## Step1

​		**BeanFactory**是一个里面包含了hashmap对象的类，当向BeanFactory注册一个对象的时候，就是把一个Object用**BeanDefination**包裹起来，然后put到hashmap里，get的时候根据名字从hashmap里边拿出来，做个类型强转。

```java
BeanFactory beanFactory = new BeanFactory();
BeanDefinition beanDefinition = new BeanDefinition(new UserService());
beanFactory.registerBeanDefinition("userService", beanDefinition);
UserService userService = (UserService) beanFactory.getBean("userService");
userService.queryUserInfo();
```

## Step2

- **BeanFactory**变为了一个接口只负责根据id获取实例化好的bean，**AbstractBeanFactory**实现了其接口（getbean方法实现放在了其中）

  ```java
  @Override
  public Object getBean(String name) throws BeansException {
      Object bean = getSingleton(name);//有创建好的就用创建好的
      if (bean != null) {
          return bean;
      }
  
      //没有创建好的就获取类定义创建bean
      BeanDefinition beanDefinition = getBeanDefinition(name);
      //创建bean的同时将其放置到缓存里
      return createBean(name, beanDefinition);
  }
  ```

- **BeanDefinitionRegistry**作为一个接口只负责注册BeanDefinition，**DefaultListableBeanFactory**实现了其接口

  >Map<String, BeanDefinition> 类的BeanDefinition定义
  >
  >registerBeanDefinition 放入BeanDefinitio
  >
  >getBeanDefinition 得到BeanDefinitio

- **SingletonBeanRegistry**作为一个接口只负责放创建好的对象与获取创建好的对象，**DefaultSingletonBeanRegistry**实现了其接口

  >Map<String, Object> 类创建好的单例对象的缓存
  >
  >getSingleton(String beanName)
  >
  >addSingleton(String beanName)

## Step3

- **AbstractBeanFactory**

  带参数的创建实际上是由反射完成的，可以选择cglib或者jdk自带的反射方法完成，这部分实现在**AbstractAutowireCapableBeanFactory**里完成

  ```java
  protected <T> T doGetBean(final String name, final Object[] args) {
    Object bean = getSingleton(name);
    if (bean != null) {
      return (T) bean;
    }
  
    BeanDefinition beanDefinition = getBeanDefinition(name);
    return (T) createBean(name, beanDefinition, args);
  }
  
  protected Object createBeanInstance(BeanDefinition beanDefinition, String beanName, Object[] args) {
          Constructor constructorToUse = null;
          Class<?> beanClass = beanDefinition.getBeanClass();
          Constructor<?>[] declaredConstructors = beanClass.getDeclaredConstructors();
          for (Constructor ctor : declaredConstructors) {
              if (null != args && ctor.getParameterTypes().length == args.length) {
                  constructorToUse = ctor;
                  break;
              }
          }
          return getInstantiationStrategy().instantiate(beanDefinition, beanName, constructorToUse, args);
      }
  ```

## Step4

- 如果属性有值的话需要填充属性，属性采用**PropertyValue**来包装属性

  ```java
  public class PropertyValue {
      private final String name;//属性名
      private final Object value;//属性值
  }
  
  //属性值可以是一个BeanReference指明属性引用的对象
  public class BeanReference {
      private final String beanName;
  }
  
  @Override
      protected Object createBean(String beanName, BeanDefinition beanDefinition, Object[] args) throws BeansException {
          Object bean = null;
          try {
              bean = createBeanInstance(beanDefinition, beanName, args);
              // 给 Bean 填充属性，填充属性时如果是一个BeanReference，那就要就行调用getBean方法获取
              applyPropertyValues(beanName, bean, beanDefinition);
          } catch (Exception e) {
              throw new BeansException("Instantiation of bean failed", e);
          }
  
          addSingleton(beanName, bean);
          return bean;
      }
  
  ```

## Step5

- 引入配置文件,之前写入BeanDefinition是手写的，实际在XML中解析完成放入BeanDefinition

## Step6

- 在getbean前后要实现一些特定的方法做一些预处理，实现机制是写一个类实现BeanPostProcessor（在 Bean 对象实例化之后修改 Bean 对象，也可以替换 Bean 对象）、BeanFactoryPostProcessor（在 Bean 对象注册后但未实例化之前，对 Bean 的定义信息BeanDefinition执行修改操作）

  ```java
  public interface BeanPostProcessor {
      /**
       * 在 Bean 对象执行初始化方法之前，执行此方法
       */
      Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
  
      /**
       * 在 Bean 对象执行初始化方法之后，执行此方法
       */
      Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
  }
  public abstract class AbstractBeanFactory{
      private final List<BeanPostProcessor> beanPostProcessors = new ArrayList<BeanPostProcessor>();//允许用户注册BeanPostProcessor，把所有的都加入每次实例化一个bean的时候都试图执行他前后置方法
  }
  ```

- 属性填充完成以后遍历bean的beanPostProcessors，然后执行其前后置方法

- 将原来的直接自己实例化工厂的方式改为使用ApplicationContext，因为我们不太可能让面向 Spring 本身开发的 DefaultListableBeanFactory 服务，直接给予用户使用

  ```java
  //向ClassPathXmlApplicationContext传入配置文件地址，同时调用refresh方法
  public ClassPathXmlApplicationContext(String[] configLocations) throws BeansException {
          this.configLocations = configLocations;
          refresh();
  }
  
  public void refresh() throws BeansException {
          // 1. 创建 BeanFactory，并加载 BeanDefinition
          //    1.1 创建DefaultListableBeanFactory
          //    1.2 载入配置文件并解析，注册类信息
          refreshBeanFactory();
  
          // 2. 获取 BeanFactory
          ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  
          // 3. 在 Bean 实例化之前，执行 BeanFactoryPostProcessor (Invoke factory processors registered as beans in the context.)
          //执行的方法是先扫描beandefinition，找到其中所有类型是BeanFactoryPostProcessor的类调用其postProcessBeanFactory方法
          invokeBeanFactoryPostProcessors(beanFactory);
  
          // 4. BeanPostProcessor 需要提前于其他 Bean 对象实例化之前执行注册操作
          //方法是先扫描beandefinition，找到其中所有类型是BeanPostProcessor的类调用beanFactory的add方法将其加入beanFactory的BeanPostProcessor，以供以后执行getbean时调用
          registerBeanPostProcessors(beanFactory);
  
          // 5. 提前实例化单例Bean对象，实例化的方法就是执行beanDefinition map里的每一个getbean方法
          beanFactory.preInstantiateSingletons();
  }
  ```

## Step7

- 初始化和销毁

  ```java
  //在配置文件中加入这两个方法init-method、destroy-method，在 BeanDefinition 的属性当中记录，这样在 initializeBean 初始化操作的工程中，就可以通过反射的方式来调用配置在 Bean 定义属性当中的方法信息了
  private void invokeInitMethods(String beanName, Object bean, BeanDefinition beanDefinition) throws Exception {
          // 1. 实现接口 InitializingBean。调用其afterPropertiesSet()
          if (bean instanceof InitializingBean) {
              ((InitializingBean) bean).afterPropertiesSet();
          }
  
          // 2. 注解配置 init-method {判断是为了避免二次执行销毁}
          String initMethodName = beanDefinition.getInitMethodName();
          if (StrUtil.isNotEmpty(initMethodName)) {
              Method initMethod = beanDefinition.getBeanClass().getMethod(initMethodName);
              if (null == initMethod) {
                  throw new BeansException("Could not find an init method named '" + initMethodName + "' on bean with name '" + beanName + "'");
              }
              initMethod.invoke(bean);
          }
      }
  //另外如果是接口实现的方式(实现InitializingBean、DisposableBean接口)，那么直接可以通过 Bean 对象调用对应接口定义的方法即可，((InitializingBean) bean).afterPropertiesSet()
  
  //在createbean 注册实现了 DisposableBean 接口的 Bean 对象
   registerDisposableBeanIfNecessary(beanName, bean, beanDefinition);
  //销毁方法最后注册为一个map
  private final Map<String, DisposableBean> disposableBeans = new HashMap<>();
  //需要在 ConfigurableApplicationContext 接口中定义注册虚拟机钩子的方法 registerShutdownHook 和手动执行关闭的方法 close,最终就是依次调用map里的销毁方法
  public void destroySingletons() {
          Set<String> keySet = this.disposableBeans.keySet();
          Object[] disposableBeanNames = keySet.toArray();
          
          for (int i = disposableBeanNames.length - 1; i >= 0; i--) {
              Object beanName = disposableBeanNames[i];
              DisposableBean disposableBean = disposableBeans.remove(beanName);
              try {
                  disposableBean.destroy();
              } catch (Exception e) {
                  throw new BeansException("Destroy method on bean with name '" + beanName + "' threw an exception", e);
              }
          }
      }
  ```

## Step8

- 感知Aware 的接口包括：BeanFactoryAware、BeanClassLoaderAware、BeanNameAware和ApplicationContextAware

  ```java
  public void refresh() throws BeansException {
          // 1. 创建 BeanFactory，并加载 BeanDefinition
          refreshBeanFactory();
  
          // 2. 获取 BeanFactory
          ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  
          // 3. 添加 ApplicationContextAwareProcessor，让继承自 ApplicationContextAware 的 Bean 对象都能感知所属的 ApplicationContext
          beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
  
          // 4. 在 Bean 实例化之前，执行 BeanFactoryPostProcessor (Invoke factory processors registered as beans in the context.)
          invokeBeanFactoryPostProcessors(beanFactory);
  
          // 5. BeanPostProcessor 需要提前于其他 Bean 对象实例化之前执行注册操作
          registerBeanPostProcessors(beanFactory);
  
          // 6. 提前实例化单例Bean对象
          beanFactory.preInstantiateSingletons();
      }
  ```

## Step9

- 单例、多例以及FctoryBean

  ```java
  //单例、多例写在配置文件中
  @Override
      protected Object createBean(String beanName, BeanDefinition beanDefinition, Object[] args) throws BeansException {
          Object bean = null;
          try {
              bean = createBeanInstance(beanDefinition, beanName, args);
              // 给 Bean 填充属性
              applyPropertyValues(beanName, bean, beanDefinition);
              // 执行 Bean 的初始化方法和 BeanPostProcessor 的前置和后置处理方法
              bean = initializeBean(beanName, bean, beanDefinition);
          } catch (Exception e) {
              throw new BeansException("Instantiation of bean failed", e);
          }
  
          // 注册实现了 DisposableBean 接口的 Bean 对象
          registerDisposableBeanIfNecessary(beanName, bean, beanDefinition);
  
          // 判断 SCOPE_SINGLETON、SCOPE_PROTOTYPE
          if (beanDefinition.isSingleton()) {
              addSingleton(beanName, bean);
          }
          return bean;
      }
  //单例对象不销毁
  ```

  ```java
  //如果对象是实现了 FactoryBean#getObject就调用
  protected <T> T doGetBean(final String name, final Object[] args) {
          Object sharedInstance = getSingleton(name);
          if (sharedInstance != null) {
              // 如果是 FactoryBean，则需要调用 FactoryBean#getObject
              return (T) getObjectForBeanInstance(sharedInstance, name);
          }
  
          BeanDefinition beanDefinition = getBeanDefinition(name);
          Object bean = createBean(name, beanDefinition, args);
          return (T) getObjectForBeanInstance(bean, name);
      }
  ```

## Step10

- 本质上就是一个观察者模式

  ```java
  public void refresh() throws BeansException {
          // 1. 创建 BeanFactory，并加载 BeanDefinition
          refreshBeanFactory();
  
          // 2. 获取 BeanFactory
          ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  
          // 3. 添加 ApplicationContextAwareProcessor，让继承自 ApplicationContextAware 的 Bean 对象都能感知所属的 ApplicationContext
          beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
  
          // 4. 在 Bean 实例化之前，执行 BeanFactoryPostProcessor (Invoke factory processors registered as beans in the context.)
          invokeBeanFactoryPostProcessors(beanFactory);
  
          // 5. BeanPostProcessor 需要提前于其他 Bean 对象实例化之前执行注册操作
          registerBeanPostProcessors(beanFactory);
  
          // 6. 初始化事件发布者
          initApplicationEventMulticaster();
  
          // 7. 注册事件监听器
          registerListeners();
  
          // 8. 提前实例化单例Bean对象
          beanFactory.preInstantiateSingletons();
  
          // 9. 发布容器刷新完成事件
          finishRefresh();
      }
  ```

  



# Spring

## 获取Spring容器

- 实现BeanFactoryAware接口获取Spring容器

  ```java
  @Service
  public  class PersonService implements BeanFactoryAware {
      private BeanFactory beanFactory;
  
      @Override
      public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
          this.beanFactory = beanFactory;
      }
  
      public void add() {
          Person person = (Person) beanFactory.getBean("person");
      }
  }
  ```

- 实现ApplicationContextAware接口

  ```java
  @Service
  public  class PersonService2 implements ApplicationContextAware {
      private ApplicationContext applicationContext;
  
      @Override
      public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
          this.applicationContext = applicationContext;
      }
  
      public void add() {
          Person person = (Person) applicationContext.getBean("person");
      }
  }
  ```

- 实现ApplicationListener接口

  ```java
  @Service
  public  class PersonService3 implements ApplicationListener<ContextRefreshedEvent> {
      private ApplicationContext applicationContext;
  
  
      @Override
      public void onApplicationEvent(ContextRefreshedEvent event) {
          applicationContext = event.getApplicationContext();
      }
  
      public void add() {
          Person person = (Person) applicationContext.getBean("person");
      }
  
  }
  ```

## 初始化bean

- 使用@PostConstruct注解

  ```java
  @Service
  public  class AService {
  
      @PostConstruct
      public void init() {
          System.out.println("===初始化===");
      }
  }
  ```

- 实现InitializingBean接口

  ```java
  @Service
  public  class BService implements InitializingBean {
  
      @Override
      public void afterPropertiesSet() throws Exception {
          System.out.println("===初始化===");
      }
  }
  ```

- 还有一种init-method方法在xml文件定义，太古老不常用，这些方法的调用顺序是：postConstruct->InitializingBean->inti-method

## 自定义Scope

我们都知道`spring`默认支持的`Scope`只有两种：

- singleton 单例，每次从spring容器中获取到的bean都是同一个对象。
- prototype 多例，每次从spring容器中获取到的bean都是不同的对象。

`spring web`又对`Scope`进行了扩展，增加了：

- RequestScope 同一次请求从spring容器中获取到的bean都是同一个对象。
- SessionScope 同一个会话从spring容器中获取到的bean都是同一个对象。

即便如此，有些场景还是无法满足我们的要求。

比如，我们想在同一个线程中从spring容器获取到的bean都是同一个对象，该怎么办？

这就需要自定义`Scope`了。

第一步实现`Scope`接口：

```java
public  class ThreadLocalScope implements Scope {

    private  static  final ThreadLocal THREAD_LOCAL_SCOPE = new ThreadLocal();

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Object value = THREAD_LOCAL_SCOPE.get();
        if (value != null) {
            return value;
        }

        Object object = objectFactory.getObject();
        THREAD_LOCAL_SCOPE.set(object);
        return object;
    }

    @Override
    public Object remove(String name) {
        THREAD_LOCAL_SCOPE.remove();
        return  null;
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {

    }

    @Override
    public Object resolveContextualObject(String key) {
        return  null;
    }

    @Override
    public String getConversationId() {
        return  null;
    }
}
```

第二步将新定义的`Scope`注入到spring容器中：

```java
@Component
public  class ThreadLocalBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        beanFactory.registerScope("threadLocalScope", new ThreadLocalScope());
    }
}
```

第三步使用新定义的`Scope`：

```java
@Scope("threadLocalScope")
@Service
public  class CService {

    public void add() {
    }
}
```

## FactoryBean使用

说起`FactoryBean`就不得不提`BeanFactory`，因为面试官老喜欢问它们的区别。

- BeanFactory：spring容器的顶级接口，管理bean的工厂。
- FactoryBean：并非普通的工厂bean，它隐藏了实例化一些复杂Bean的细节，给上层应用带来了便利。

如果你看过spring源码，会发现它有70多个地方在用FactoryBean接口。特别提一句：`mybatis`的`SqlSessionFactory`对象就是通过`SqlSessionFactoryBean`类创建的。

``` java
@Component
public  class MyFactoryBean implements FactoryBean {

    @Override
    public Object getObject() throws Exception {
        String data1 = buildData1();
        String data2 = buildData2();
        return buildData3(data1, data2);
    }

    private String buildData1() {
        return  "data1";
    }

    private String buildData2() {
        return  "data2";
    }

    private String buildData3(String data1, String data2) {
        return data1 + data2;
    }


    @Override
    public Class<?> getObjectType() {
        return  null;
    }
}
```

```java
@Service
public  class MyFactoryBeanService implements BeanFactoryAware {
    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    public void test() {
        Object myFactoryBean = beanFactory.getBean("myFactoryBean");
        System.out.println(myFactoryBean);
        Object myFactoryBean1 = beanFactory.getBean("&myFactoryBean");
        System.out.println(myFactoryBean1);
    }
}
```

- `getBean("myFactoryBean");`获取的是MyFactoryBeanService类中getObject方法返回的对象
- `getBean("&myFactoryBean");`获取的才是MyFactoryBean对象

## 自定义类型转换

接口中接收参数的实体对象中，有个字段的类型是Date，但是实际传参的是字符串类型：2021-01-03 10:20:15，要如何处理呢？

第一步，定义一个实体`User`：

```java
@Data
public  class User {

    private Long id;
    private String name;
    private Date registerDate;
}
```

第二步，实现`Converter`接口：

```java
public  class DateConverter implements Converter<String, Date> {

    private SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    @Override
    public Date convert(String source) {
        if (source != null && !"".equals(source)) {
            try {
                simpleDateFormat.parse(source);
            } catch (ParseException e) {
                e.printStackTrace();
            }
        }
        return  null;
    }
}
```

第三步，将新定义的类型转换器注入到spring容器中：

```java
@Configuration
public  class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new DateConverter());
    }
}
```

第四步，调用接口

```java
@RequestMapping("/user")
@RestController
public  class UserController {

    @RequestMapping("/save")
    public String save(@RequestBody User user) {
        return  "success";
    }
}
```

请求接口时`User`对象中`registerDate`字段会被自动转换成`Date`类型

## Spring MVC拦截器

spring mvc拦截器根spring拦截器相比，它里面能够获取`HttpServletRequest`和`HttpServletResponse` 等web对象实例。

spring mvc拦截器的顶层接口是：`HandlerInterceptor`，包含三个方法：

- preHandle 目标方法执行前执行
- postHandle 目标方法执行后执行
- afterCompletion 请求完成时执行

为了方便我们一般情况会用HandlerInterceptor接口的实现类`HandlerInterceptorAdapter`类。

假如有权限认证、日志、统计的场景，可以使用该拦截器。

第一步，继承`HandlerInterceptorAdapter`类定义拦截器：

```java
public  class AuthInterceptor extends HandlerInterceptorAdapter {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        String requestUrl = request.getRequestURI();
        if (checkAuth(requestUrl)) {
            return  true;
        }

        return  false;
    }

    private boolean checkAuth(String requestUrl) {
        System.out.println("===权限校验===");
        return  true;
    }
}
```

第二步，将该拦截器注册到spring容器：

```java
@Configuration
public  class WebAuthConfig extends WebMvcConfigurerAdapter {
 
    @Bean
    public AuthInterceptor getAuthInterceptor() {
        return  new AuthInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AuthInterceptor());
    }
}
```

第三步，在请求接口时spring mvc通过该拦截器，能够自动拦截该接口，并且校验权限。

## Enable开关

不知道你有没有用过`Enable`开头的注解，比如：`EnableAsync`、`EnableCaching`、`EnableAspectJAutoProxy`等，这类注解就像开关一样，只要在`@Configuration`定义的配置类上加上这类注解，就能开启相关的功能。

实现一个自己的开关：

第一步，定义一个LogFilter：

```java
public  class LogFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        System.out.println("记录请求日志");
        chain.doFilter(request, response);
        System.out.println("记录响应日志");
    }

    @Override
    public void destroy() {
        
    }
}
```

第二步，注册LogFilter：

```java
@ConditionalOnWebApplication
public  class LogFilterWebConfig {

    @Bean
    public LogFilter timeFilter() {
        return  new LogFilter();
    }
}
```

注意，这里用了`@ConditionalOnWebApplication`注解，没有直接使用`@Configuration`注解。

第三步，定义开关`@EnableLog`注解：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(LogFilterWebConfig.class)
public @interface EnableLog {

}
```

第四步，只需在`springboot`启动类加上`@EnableLog`注解即可开启LogFilter记录请求和响应日志的功能。

## RestTemplate拦截器

我们使用`RestTemplate`调用远程接口时，有时需要在`header`中传递信息，比如：traceId，source等，便于在查询日志时能够串联一次完整的请求链路，快速定位问题。

这种业务场景就能通过`ClientHttpRequestInterceptor`接口实现，具体做法如下：

第一步，实现`ClientHttpRequestInterceptor`接口：

```java
public  class RestTemplateInterceptor implements ClientHttpRequestInterceptor {

    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        request.getHeaders().set("traceId", MdcUtil.get());
        return execution.execute(request, body);
    }
}
```

第二步，定义配置类：

```java
@Configuration
public  class RestTemplateConfiguration {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.setInterceptors(Collections.singletonList(restTemplateInterceptor()));
        return restTemplate;
    }

    @Bean
    public RestTemplateInterceptor restTemplateInterceptor() {
        return  new RestTemplateInterceptor();
    }
}
```

其中MdcUtil其实是利用`MDC`工具在`ThreadLocal`中存储和获取traceId

```java
public  class MdcUtil {

    private  static  final String TRACE_ID = "TRACE_ID";

    public static String get() {
        return MDC.get(TRACE_ID);
    }

    public static void add(String value) {
        MDC.put(TRACE_ID, value);
    }
}
```

当然，这个例子中没有演示MdcUtil类的add方法具体调的地方，我们可以在filter中执行接口方法之前，生成traceId，调用MdcUtil类的add方法添加到`MDC`中，然后在同一个请求的其他地方就能通过MdcUtil类的get方法获取到该traceId。

## Spring缓存

## @Conditional

不知道你们有没有遇到过这些问题：

1. 某个功能需要根据项目中有没有某个jar判断是否开启该功能。
2. 某个bean的实例化需要先判断另一个bean有没有实例化，再判断是否实例化自己。
3. 某个功能是否开启，在配置文件中有个参数可以对它进行控制。

如果你有遇到过上述这些问题，那么恭喜你，本节内容非常适合你。

@ConditionalOnClass

问题1可以用`@ConditionalOnClass`注解解决，代码如下：

```java
public  class A {
}

public  class B {
}

@ConditionalOnClass(B.class)
@Configuration
public class TestConfiguration {

    @Bean
    public A a() {
      return    new A();
    }
}
```

如果项目中存在B类，则会实例化A类。如果不存在B类，则不会实例化A类。

有人可能会问：不是判断有没有某个jar吗？怎么现在判断某个类了？

> 直接判断有没有该jar下的某个关键类更简单。

这个注解有个升级版的应用场景：比如common工程中写了一个发消息的工具类mqTemplate，业务工程引用了common工程，只需再引入消息中间件，比如rocketmq的jar包，就能开启mqTemplate的功能。而如果有另一个业务工程，通用引用了common工程，如果不需要发消息的功能，不引入rocketmq的jar包即可。

这个注解的功能还是挺实用的吧？

@ConditionalOnBean

问题2可以通过`@ConditionalOnBean`注解解决，代码如下：

```java
@Configuration
public  class TestConfiguration {

    @Bean
    public B b() {
        return  new B();
    }

    @ConditionalOnBean(name="b")
    @Bean
    public A a() {
      return    new A();
    }
}
```

实例A只有在实例B存在时，才能实例化。

@ConditionalOnProperty

问题3可以通过`@ConditionalOnProperty`注解解决，代码如下：

```java
@ConditionalOnProperty(prefix = "demo",name="enable", havingValue = "true",matchIfMissing=true )
@Configuration
public  class TestConfiguration {

    @Bean
    public A a() {
      return    new A();
    }
}
```

在applicationContext.properties文件中配置参数：

```properties
demo.enable=false
```

各参数含义：

- prefix 表示参数名的前缀，这里是demo
- name 表示参数名
- havingValue 表示指定的值，参数中配置的值需要跟指定的值比较是否相等，相等才满足条件
- matchIfMissing 表示是否允许缺省配置。

这个功能可以作为开关，相比EnableXXX注解的开关更优雅，因为它可以通过参数配置是否开启，而EnableXXX注解的开关需要在代码中硬编码开启或关闭。

其他的Conditional注解

当然，spring用得比较多的Conditional注解还有：`ConditionalOnMissingClass`、`ConditionalOnMissingBean`、`ConditionalOnWebApplication`等。

## 自定义Conditioinal

说实话，个人认为springboot自带的Conditional系列已经可以满足我们绝大多数的需求了。但如果你有比较特殊的场景，也可以自定义自定义Conditional。

第一步，自定义注解：

```java
@Conditional(MyCondition.class)
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
public  @interface MyConditionOnProperty {
    String name() default "";

    String havingValue() default "";
}
```

第二步，实现Condition接口：

```java
public  class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        System.out.println("实现自定义逻辑");
        return  false;
    }
}
```

第三步，使用@MyConditionOnProperty注解。

## @Import

- 普通类

  这种引入方式是最简单的，被引入的类会被实例化bean对象。

  ```java
  public  class A {
  }
  
  @Import(A.class)
  @Configuration
  public class TestConfiguration {
  }
  ```

  通过`@Import`注解引入A类，spring就能自动实例化A对象，然后在需要使用的地方通过`@Autowired`注解注入即可：

  ```java
     @Autowired
  private A a;
  ```

  是不是挺让人意外的？不用加`@Bean`注解也能实例化bean。

- Configuration注解的配置类

  这种引入方式是最复杂的，因为`@Configuration`注解还支持多种组合注解，比如：

  - `@Import`
  - `@ImportResource`
  - `@PropertySource`等。

  ```java
  public  class A {
  }
  
  public  class B {
  }
  
  @Import(B.class)
  @Configuration
  public class AConfiguration {
  
      @Bean
      public A a() {
          return  new A();
      }
  }
  
  @Import(AConfiguration.class)
  @Configuration
  public class TestConfiguration {
  }
  ```

  通过`@Import`注解引入@Configuration注解的配置类，会把该配置类相关`@Import`、`@ImportResource`、`@PropertySource`等注解引入的类进行递归，一次性全部引入。

- 实现ImportSelector接口的类

  这种引入方式需要实现`ImportSelector`接口：

  ```java
  public  class AImportSelector implements ImportSelector {
  
  private  static  final String CLASS_NAME = "com.sue.cache.service.test13.A";
      
   public String[] selectImports(AnnotationMetadata importingClassMetadata) {
          return  new String[]{CLASS_NAME};
      }
  }
  
  @Import(AImportSelector.class)
  @Configuration
  public class TestConfiguration {
  }
  ```

  这种方式的好处是`selectImports`方法返回的是数组，意味着可以同时引入多个类，还是非常方便的。

- 实现ImportBeanDefinitionRegistrar接口的类

  这种引入方式需要实现`ImportBeanDefinitionRegistrar`接口：

  ```java
  public  class AImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
      @Override
      public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
          RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(A.class);
          registry.registerBeanDefinition("a", rootBeanDefinition);
      }
  }
  
  @Import(AImportBeanDefinitionRegistrar.class)
  @Configuration
  public class TestConfiguration {
  }
  ```

  这种方式是最灵活的，能在`registerBeanDefinitions`方法中获取到`BeanDefinitionRegistry`容器注册对象，可以手动控制`BeanDefinition`的创建和注册。

  当然`@import`注解非常人性化，还支持同时引入多种不同类型的类。

  ```java
  @Import({B.class,AImportBeanDefinitionRegistrar.class})
  @Configuration
  public class TestConfiguration {
  }
  ```

  这四种引入类的方式各有千秋，总结如下：

  1. 普通类，用于创建没有特殊要求的bean实例。
  2. @Configuration注解的配置类，用于层层嵌套引入的场景。
  3. 实现ImportSelector接口的类，用于一次性引入多个类的场景，或者可以根据不同的配置决定引入不同类的场景。
  4. 实现ImportBeanDefinitionRegistrar接口的类，主要用于可以手动控制BeanDefinition的创建和注册的场景，它的方法中可以获取BeanDefinitionRegistry注册容器对象。

## @ConfigurationProperties

我们在项目中使用配置参数是非常常见的场景，比如，我们在配置线程池的时候，需要在`applicationContext.propeties`文件中定义如下配置：

```properties
thread.pool.corePoolSize=5
thread.pool.maxPoolSize=10
thread.pool.queueCapacity=200
thread.pool.keepAliveSeconds=30
```

方法一：通过`@Value`注解读取这些配置。

```java
public  class ThreadPoolConfig {

    @Value("${thread.pool.corePoolSize:5}")
    private  int corePoolSize;

    @Value("${thread.pool.maxPoolSize:10}")
    private  int maxPoolSize;

    @Value("${thread.pool.queueCapacity:200}")
    private  int queueCapacity;

    @Value("${thread.pool.keepAliveSeconds:30}")
    private  int keepAliveSeconds;

    @Value("${thread.pool.threadNamePrefix:ASYNC_}")
    private String threadNamePrefix;

    @Bean
    public Executor threadPoolExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(queueCapacity);
        executor.setKeepAliveSeconds(keepAliveSeconds);
        executor.setThreadNamePrefix(threadNamePrefix);
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}
```

这种方式使用起来非常简单，但建议在使用时都加上`:`，因为`:`后面跟的是默认值，比如：@Value("${thread.pool.corePoolSize:5}")，定义的默认核心线程数是5。

> 假如有这样的场景：business工程下定义了这个ThreadPoolConfig类，api工程引用了business工程，同时job工程也引用了business工程，而ThreadPoolConfig类只想在api工程中使用。这时，如果不配置默认值，job工程启动的时候可能会报错。

如果参数少还好，多的话，需要给每一个参数都加上`@Value`注解，是不是有点麻烦？

此外，还有一个问题，`@Value`注解定义的参数看起来有点分散，不容易辨别哪些参数是一组的。

这时，`@ConfigurationProperties`就派上用场了，它是springboot中新加的注解。

第一步，先定义ThreadPoolProperties类

```java
@Data
@Component
@ConfigurationProperties("thread.pool")
public  class ThreadPoolProperties {

    private  int corePoolSize;
    private  int maxPoolSize;
    private  int queueCapacity;
    private  int keepAliveSeconds;
    private String threadNamePrefix;
}
```

第二步，使用ThreadPoolProperties类

```java
@Configuration
public  class ThreadPoolConfig {

    @Autowired
    private ThreadPoolProperties threadPoolProperties;

    @Bean
    public Executor threadPoolExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(threadPoolProperties.getCorePoolSize());
        executor.setMaxPoolSize(threadPoolProperties.getMaxPoolSize());
        executor.setQueueCapacity(threadPoolProperties.getQueueCapacity());
        executor.setKeepAliveSeconds(threadPoolProperties.getKeepAliveSeconds());
        executor.setThreadNamePrefix(threadPoolProperties.getThreadNamePrefix());
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}
```

使用`@ConfigurationProperties`注解，可以将`thread.pool`开头的参数直接赋值到ThreadPoolProperties类的同名参数中，这样省去了像`@Value`注解那样一个个手动去对应的过程。

这种方式显然要方便很多，我们只需编写xxxProperties类，spring会自动装配参数。此外，不同系列的参数可以定义不同的xxxProperties类，也便于管理，推荐优先使用这种方式。

它的底层是通过：`ConfigurationPropertiesBindingPostProcessor`类实现的，该类实现了`BeanPostProcessor`接口，在`postProcessBeforeInitialization`方法中解析`@ConfigurationProperties`注解，并且绑定数据到相应的对象上。

绑定是通过`Binder`类的`bindObject`方法完成的

## 事务

- 声明式事务

  大多数情况下，我们在开发过程中使用更多的可能是`声明式事务`，即使用`@Transactional`注解定义的事务，因为它用起来更简单，方便。

  只需在需要执行的事务方法上，加上`@Transactional`注解就能自动开启事务，这种声明式事务之所以能生效，是因为它的底层使用了AOP，创建了代理对象，调用`TransactionInterceptor`拦截器实现事务的功能。

  >spring事务有个特别的地方：它获取的数据库连接放在`ThreadLocal`中的，也就是说同一个线程中从始至终都能获取同一个数据库连接，可以保证同一个线程中多次数据库操作在同一个事务中执行。

  正常情况下是没有问题的，但是如果使用不当，事务会失效，主要原因如下：

  - 事务方法错误的权限
  - 事务方法被定义成final
  - 当前service对象没被spring管理
  - 错误的事务传播特性
  - 方法中直接调方法，无法生成代理
  - 多线程中调用嵌套事务，导致外层事务和嵌套事务不在同一个事务中
  - 捕获了异常，导致事务不能正常回滚
  - 抛出非RuntimeException，导致事务不回滚

- 编程式事务

  一般情况下编程式事务我们可以通过`TransactionTemplate`类开启事务功能。有个好消息，就是`springboot`已经默认实例化好这个对象了，我们能直接在项目中使用。

  ```java
  @Service
  public  class UserService {
     @Autowired
     private TransactionTemplate transactionTemplate;
     
     ...
     
     public void save(final User user) {
           transactionTemplate.execute((status) => {
              doSameThing...
              return Boolean.TRUE;
           })
     }
  }
  ```

  使用`TransactionTemplate`的编程式事务能避免很多事务失效的问题，但是对大事务问题，不一定能够解决，只是说相对于使用`@Transactional`注解要好些。

## 跨域

## 自定义starter

## 项目启动时附加功能

## AOP

https://mp.weixin.qq.com/s/VW9pJ8NWKouKFzfbQO-LIw

## AOP的一些坑

- 失效问题

```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Timer {
}

@Aspect
@Component
public class TimerAspect {

    @Pointcut("@annotation(com.wyj.service.Timer)")
    public void pointCut() {
    }

    @Before("pointCut()")
    public void before() {
        System.out.println("before");
    }

    @After("pointCut()")
    public void after() {
        System.out.println("after");
    }
}

@Service
public class TimerService {
    public void helloWithoutAop() {
        // 没有走横切逻辑
        hello("aop");
    }

    @Timer
    public void hello(String content) {
        System.out.println("hello" + content);
    }
}

public class TimerController {

    @Autowired
    private TimerService timerService;

    @RequestMapping(method = RequestMethod.POST, value = "/download")
    public void downLoad() {
        System.out.println(timerService.hashCode());
        timerService.helloWithoutAop();
        timerService.hello("qwewq");
    }
}

//输出：
helloaop
before
helloqwewq
after
```

可以看到第一次没有走AOP，第二次走了，原理其实很简单，动态代理的基础是静态代理，静态代理的原理如下：

```java
public interface Subject {//共用接口
    void request();
}

public class RealSubject implements Subject{//被代理类
    @Override
    public void request() {
        System.out.println("真实的请求");
    }
}

public class ProxySubject implements Subject{//代理类
    RealSubject realSubject;//持有被代理类的引用

    @Override
    public void request() {
        if(realSubject==null){
          //此处可以做增强
            realSubject = new RealSubject();
          //此处也可以做增强
        }
        realSubject.request();
    }
    
    public void request2() {
        realSubject.request();
    }
    
}

public class Main {
    public static void main(String[] args) {
        ProxySubject proxySubject = new ProxySubject();
        proxySubject.request();
    }
}
```

- 动态代理后生成的代码类似于上面静态代理的代码，它调用的是代理类持有的真实类的方法而不是代理的方法，因此在非增强的方法上调用增强的方法就调不到切面方法。

# SpringBoot

## 自动配置原理

- 什么是自动配置

  >1. 只要我们在pom.xml文件中引入spring-boot-starter-data-redis-xxx.jar包，然后只要在配置文件中配置redis连接，如：
  >
  >```properties
  >spring.redis.database = 0
  >spring.redis.timeout =10000
  >spring.redis.host = 10.72.16.9
  >spring.redis.port = 6379spring.redis.pattern = 1
  >```
  >
  >就可以在service方法中直接注入`StringRedisTemplate`对象的实例，可以直接使用了。
  >
  >```java
  >@Autowired
  >private StringRedisTemplate stringRedisTemplate;
  >```
  >
  >2. 在项目中只要引入spring-boot-starter-xxx.jar，事务就自动生效了，并且可以直接在service方法中直接注入TransactionTemplate，用它开发编程式事务代码。
  >
  >3. 使用@ConfigurationProperties可以把指定路径下的属性，直接注入到实体对象中，看看下面这个例子：
  >
  >   ```properties
  >   jump.threadpool.corePoolSize=8
  >   jump.threadpool.maxPoolSize=16
  >   jump.threadpool.keepAliveSeconds=10
  >   jump.threadpool.queueCapacity=100
  >   ```
  >
  >   ```java
  >   @Data
  >   @Component
  >   @ConfigurationProperties("jump.threadpool")
  >   public class ThreadPoolProperties {
  >   
  >       private int corePoolSize;
  >       private int maxPoolSize;
  >       private int keepAliveSeconds;
  >       private int queueCapacity;
  >   }
  >   ```

  - 自动配置原理

    >1. bean的自动配置
    >
    >   Spring Boot的启动类上有一个@SpringBootApplication，这个注解包括了@EnableAutoConfiguration，该注解的关键功能由@Import提供，其导入的AutoConfigurationImportSelector的selectImports()方法通过SpringFactoriesLoader.loadFactoryNames()扫描所有具有META-INF/spring.factories的jar包下面key是EnableAutoConfiguration全名的，所有自动配置类。比如springboot的spring-boot-autoconfigure-xxx.jar下的META-INF/spring.factories文件
    >
    >```properties
    >org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
    >org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
    >org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
    >org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
    >```
    >
    >加载过程大概是这样的：
    >
    >SpringApplication.run(...)方法->
    >
    >AbstractApplicationContext.refresh()方法->
    >
    >invokeBeanFactoryPostProcessors(...)方法->
    >
    >**PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(...)** 方法->
    >
    >ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(..)方法->
    >
    >AutoConfigurationImportSelector.selectImports
    >
    >该方法会找到自动配置的类，并给打了@Bean注解的方法创建对象。
    >
    >postProcessBeanDefinitionRegistry方法是最核心的方法，它负责解析@Configuration、@Import、@ImportSource、@Component、@ComponentScan、@Bean等，完成bean的自动配置功能。
    >
    >2. 属性的自动配置
    >
    >   属性的自动配置是通过ConfigurationPropertiesBindingPostProcessor类的postProcessBeforeInitialization方法完成，它会解析@ConfigurationProperties注解上的属性，将配置文件中对应key的值绑定到属性上。
    >
    >3. 自动配置的生效条件
    >
    >   每个xxxxAutoConfiguration类上都可以定义一些生效条件，这些条件基本都是从@Conditional派生出来的。
    >
    >   ```
    >   @ConditionalOnBean：当容器里有指定的bean时生效@ConditionalOnMissingBean：当容器里不存在指定bean时生效@ConditionalOnClass：当类路径下有指定类时生效@ConditionalOnMissingClass：当类路径下不存在指定类时生效@ConditionalOnProperty：指定的属性是否有指定的值，比如@ConditionalOnProperties(prefix=”xxx.xxx”, value=”enable”, matchIfMissing=true)，代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。
    >   ```

  # 基础问题
  
  https://mp.weixin.qq.com/s/Y17S85ntHm_MLTZMJdtjQQ
  
  ## 特性
  
  Spring 是一个轻量级、非入侵式的控制反转 (IoC) 和面向切面 (AOP) 的框架
  
  - IOC、DI
  - AOP
  - 声明式事务
  - 快捷测试
  - 快速集成
  - 复杂API模板封装
  
  ## 模块
  
  - Core
  - Context
  - Web
  - MVC
  - DAO
  - ORM
  - AOP
  
  ## 常用注解
  
  https://mp.weixin.qq.com/s/u_O1m7Wyg0icIKffl20U1Q
  
  - web
  
    @Controller、@RestController、@RequesuMapping(get、post、put、delete)、@ResponseBody、@RequestBody、@PathVariable
  
  - 容器
  
    @Componet、@Service、@Repository、@Autowired、@Qualifier、@Configuration、@Value、@Bean、@Scope
  
  - AOP
  
    @Aspect、@After、@Before、@Around、@PointCut
  
  - 事务
  
    @Transactional
  
  ## 设计模式
  
  - 工厂： Spring 容器本质是一个大工厂，使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象
  - 代理：Spring AOP 功能功能就是通过代理模式来实现的，分为动态代理和静态代理
  - 单例：Spring 中的 Bean 默认都是单例的，这样有利于容器对Bean的管理
  - 模板：Spring 中 JdbcTemplate、RestTemplate 等以 Template结尾的对数据库、网络等等进行操作的模板类
  - 观察者：Spring 事件驱动模型就是观察者模式很经典的一个应用
  - 适配器：Spring AOP 的增强或通知 (Advice) 使用到了适配器模式、Spring MVC 中也是用到了适配器模式适配 Controller
  - 策略：Spring中有一个Resource接口，它的不同实现类，会根据不同的策略去访问资源
  
  ## 什么是IOC?什么是DI？
  
  ## Spring IOC的实现机制
  
  ## BeanFactory和ApplicantContext
  
  ## Spring容器启动阶段会干什么
  
  ## Spring Bean生命周期
  
  ## Bean定义和依赖定义有哪些方式
  
  ## 有哪些依赖注入的方法
  
  ## 有哪些自动装配的方式
  
  ## Bean 的作用域有哪些
  
  ## 单例 Bean 会存在线程安全问题吗
  
  ## 循环依赖
  
  - 为什么要三级缓存？⼆级不⾏吗？
  
  ## @Autowired的实现原理
  
  ## AOP有哪些概念
  
  ## JDK 动态代理和 CGLIB 代理
  
  ## Spring AOP 和 AspectJ AOP 区别
  
  ## Spring 事务
  
  - 种类
  - 隔离级别
  - 传播机制
  - 失效情况
  - 声明式事务实现原理
  
  ## MVC核心组件
  
  ## MVC工作流程
  
  ## SpringMVC Restful风格的接口的流程
  
  ## SpringBoot
  
  - 自动配置原理
  - 如何自定义一个SpringBoot Srarter
  - Springboot 启动原理
  
  ## SpringCloud
  
  
  
  ```properties
  手写模拟Spring
  手写模拟Spring扫描底层实现
  手写模拟BeanDefinition的生成
  手写模拟getBean方法底层实现
  手写模拟Bean创建流程
  手写模拟依赖注入
  手写模拟Aware回调机制
  手写模拟Spring初始化机制
  手写模拟BeanPostProcessor机制
  手写模拟SpringAOP机制
  Bean生命周期底层原理
  依赖注入底层原理
  单例池底层原理
  @PostConstruct底层原理
  初始化底层原理源码实现
  推断构造方法底层原理
  AOP底层实现原理
  Spring事务底层实现原理
  Spring事务失效原理
  @Configuration的底层实现原理
  为什么会出现循环依赖？
  如何打破循环依赖？
  什么是提前进行AOP？
  第二级缓存earlySingletonObjects的作用
  第三级缓存singletonFactories的作用
  为什么@Lazy能解决循环依赖
  Spring整合Mybatis底层原理
  整合Mybatis代理对象
  ImportBeanDefinitionRegistrar的作用
  Mapper扫描的底层原理
  Spring扫描入口
  到底什么是配置类？
  Spring中的beanName生成机制
  ScopedProxyMode的作用
  ExcludeFilter机制
  扫描路径解析源码分析
  扫描中的ASM技术
  扫描中的IncludeFilter机制
  扫描中的独立类、接口、抽象类
  @Lookup注解与ComponentsIndex
  
  SpringBoot的优势
  快速搭建一个Web工程
  Start机制的作用与原理
  SpringBoot的自动配置与手动配置
  @Configuration的作用与底层原理
  spring.factories机制的作用
  @SpringBootApplication注解解析
  TypeExcludeFilter机制
  自动配置总结
  @ConditionalOnBean与@ConditionalOnMissingBean
  @ConditionalOnSingleCandidate
  @ConditionalOnClass和ConditionalOnMissingClass
  ConditionalOnExpression
  @ConditionalOnJava
  @ConditionalOnWebApplication
  ConditionalOnProperty和ConditionalOnResource
  @ConditionalOnCloudPlatform
  SpringBoot中的属性绑定
  yml配置与配置优先级
  Spring Boot中的Profiles机制
  Spring Boot整合JdbcTemplate
  Spring Boot整合Mybatis
  Spring Boot整合Mybatis Plus
  Spring Boot整合JPA
  Spring Boot整合Redis
  源码篇-启动流程梳理
  源码篇-推断应用类型
  源码篇-读取spring.factories文件
  源码篇-获取SpringApplicationRunListener
  源码篇-启动剩余步骤
  源码篇-ApplicationRunner和CommandLineRunner
  源码篇-自动配置解析顺序
  源码篇-自动配置类读取与过滤
  源码篇-自动配置类条件检查
  源码篇-tomcat和jetty决策实现
  源码篇-tomcat自定义配置生效过程
  MVC模式解读
  SpringMvc执行流程图解形式深度剖析
  SpringMvc底层源码深度剖析
  SpringMvc控制器不同实现方式与底层源码详解
  SpringMvc参数注入解密
  SpringMVC拦截器执行流程详解
  SpringMvc框架核心功能手写实现
  Spring容器与SpringMvc容器关系分析
  ```
  
  
  
  
  
  
  
  

