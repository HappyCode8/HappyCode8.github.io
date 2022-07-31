# Small-Spring

[Spring手撸专栏](https://bugstack.cn/md/spring/develop-spring/2021-05-16-%E7%AC%AC1%E7%AB%A0%EF%BC%9A%E5%BC%80%E7%AF%87%E4%BB%8B%E7%BB%8D%EF%BC%8C%E6%89%8B%E5%86%99Spring%E8%83%BD%E7%BB%99%E4%BD%A0%E5%B8%A6%E6%9D%A5%E4%BB%80%E4%B9%88%EF%BC%9F.html)

## Step1：简单的Bean容器

​        **BeanFactory**是一个里面包含了hashmap对象的类，当向BeanFactory注册一个对象的时候，就是把一个Object用**BeanDefination**包裹起来，然后put到hashmap里，get的时候根据名字从hashmap里边拿出来，做个类型强转。

```java
BeanFactory beanFactory = new BeanFactory();
BeanDefinition beanDefinition = new BeanDefinition(new UserService());
beanFactory.registerBeanDefinition("userService", beanDefinition);
UserService userService = (UserService) beanFactory.getBean("userService");
userService.queryUserInfo();
```

## Step2：Bean定义、注册与获取

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
  
  > Map<String, BeanDefinition> 类的BeanDefinition定义
  > 
  > registerBeanDefinition 放入BeanDefinitio
  > 
  > getBeanDefinition 得到BeanDefinitio

- **SingletonBeanRegistry**作为一个接口只负责放创建好的对象与获取创建好的对象，**DefaultSingletonBeanRegistry**实现了其接口
  
  > Map<String, Object> 类创建好的单例对象的缓存
  > 
  > getSingleton(String beanName)
  > 
  > addSingleton(String beanName)

## Step3：带构造函数的类的实例化

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

## Step4：注入属性和依赖对象

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

## Step5：解析XML文件注册对象

- 引入配置文件,之前写入BeanDefinition是手写的，实际在XML中解析完成放入BeanDefinition

## Step6：实现应用上下文

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

## Step7：初始化和销毁方法

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

## Step8：Aware感知容器对象

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

## Step9：对象作用域与FactoryBean

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

## Step10：容器事件和容器监听器

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

## Step11：基于JDK和Cglib动态代理，实现AOP

```java
public class JdkDynamicAopProxy implements AopProxy, InvocationHandler {

    private final AdvisedSupport advised;

    public JdkDynamicAopProxy(AdvisedSupport advised) {
        this.advised = advised;
    }

    @Override
    public Object getProxy() {
        return Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), advised.getTargetSource().getTargetClass(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if (advised.getMethodMatcher().matches(method, advised.getTargetSource().getTarget().getClass())) {
            MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
            return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(), method, args));
        }
        return method.invoke(advised.getTargetSource().getTarget(), args);
    }

}
```

## Step12：将AOP融入Bean生命周期

实现AOP的原理是用BeanPostProcessor在对象初始化之前执行方法

```java
//InstantiationAwareBeanPostProcessor这个接口继承了BeanPostProcessor，然后在对象初始化之前调用其postProcessBeforeInstantiation方法完成AOP
public class DefaultAdvisorAutoProxyCreator implements InstantiationAwareBeanPostProcessor, BeanFactoryAware {

    private DefaultListableBeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = (DefaultListableBeanFactory) beanFactory;
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {

        if (isInfrastructureClass(beanClass)) return null;

        Collection<AspectJExpressionPointcutAdvisor> advisors = beanFactory.getBeansOfType(AspectJExpressionPointcutAdvisor.class).values();

        for (AspectJExpressionPointcutAdvisor advisor : advisors) {
            ClassFilter classFilter = advisor.getPointcut().getClassFilter();
            if (!classFilter.matches(beanClass)) continue;

            AdvisedSupport advisedSupport = new AdvisedSupport();

            TargetSource targetSource = null;
            try {
                targetSource = new TargetSource(beanClass.getDeclaredConstructor().newInstance());
            } catch (Exception e) {
                e.printStackTrace();
            }
            advisedSupport.setTargetSource(targetSource);
            advisedSupport.setMethodInterceptor((MethodInterceptor) advisor.getAdvice());
            advisedSupport.setMethodMatcher(advisor.getPointcut().getMethodMatcher());
            advisedSupport.setProxyTargetClass(false);

            return new ProxyFactory(advisedSupport).getProxy();

        }

        return null;
    }

}
```

## Step13：通过注解配置和包自动扫描的方式完成Bean对象的注册

- 自动扫描注册主要是扫描添加了自定义注解的类，在xml加载过程中提取类的信息，组装 BeanDefinition 注册到 Spring 容器中。
- 所以我们会用到 `<context:component-scan />` 配置包路径并在 XmlBeanDefinitionReader 解析并做相应的处理。*这里的处理会包括对类的扫描、获取注解信息等*
- 最后还包括了一部分关于 `BeanFactoryPostProcessor` 的使用，因为我们需要完成对占位符配置信息的加载，所以需要使用到 BeanFactoryPostProcessor 在所有的 BeanDefinition 加载完成后，实例化 Bean 对象之前，修改 BeanDefinition 的属性信息。*这一部分的实现也为后续处理关于占位符配置到注解上做准备*

## Step14：通过注解给属性注入配置和Bean对象

- 要处理自动扫描注入，包括属性注入、对象注入，则需要在对象属性 `applyPropertyValues` 填充之前 ，把属性信息写入到 PropertyValues 的集合中去。这一步的操作相当于是解决了以前在 spring.xml 配置属性的过程。
- 而在属性的读取中，需要依赖于对 Bean 对象的类中属性的配置了注解的扫描，`field.getAnnotation(Value.class);` 依次拿出符合的属性并填充上相应的配置信息。*这里有一点 ，属性的配置信息需要依赖于 BeanFactoryPostProcessor 的实现类 PropertyPlaceholderConfigurer，把值写入到 AbstractBeanFactory的embeddedValueResolvers集合中，这样才能在属性填充中利用 beanFactory 获取相应的属性值*
- 还有一个是关于 @Autowired 对于对象的注入，其实这一个和属性注入的唯一区别是对于对象的获取 `beanFactory.getBean(fieldType)`，其他就没有什么差一点了。
- 当所有的属性被设置到 PropertyValues 完成以后，接下来就到了创建对象的下一步，属性填充，而此时就会把我们一一获取到的配置和对象填充到属性上，也就实现了自动注入的功能

## Step15：给代理对象的属性设置值

```java
 @Override
    protected Object createBean(String beanName, BeanDefinition beanDefinition, Object[] args) throws BeansException {
        Object bean = null;
        try {
            // 判断是否返回代理 Bean 对象
            bean = resolveBeforeInstantiation(beanName, beanDefinition);
            if (null != bean) {
                return bean;
            }
            // 实例化 Bean
            bean = createBeanInstance(beanDefinition, beanName, args);
            // 实例化后判断
            boolean continueWithPropertyPopulation = applyBeanPostProcessorsAfterInstantiation(beanName, bean);
            if (!continueWithPropertyPopulation) {
                return bean;
            }
            // 在设置 Bean 属性之前，允许 BeanPostProcessor 修改属性值
            applyBeanPostProcessorsBeforeApplyingPropertyValues(beanName, bean, beanDefinition);
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
            registerSingleton(beanName, bean);
        }
        return bean;
    }
```

## 总结

- bean实例化的过程
  
  ```java
  @Override
      public void refresh() throws BeansException {
          /*
          1. 创建 BeanFactory，并加载 BeanDefinition
             1.1 创建 BeanFactory就是new DefaultListableBeanFactory()(包含Map<String, BeanDefinition>         beanDefinitionMap beanDefinitionMap)
             1.2 然后用loadBeanDefinitions方法，解析xml文件将得到的bean的属性全部解析到，主要包括id,name,class,inti-method,destroy-method,scope等，解析完成以后调用DefaultListableBeanFactory的registerBeanDefinition方法将放入 beanDefinitionMap方法里
          */
          refreshBeanFactory();
  
          // 2. 获取 BeanFactory，获取的就是1中创建的DefaultListableBeanFactory()
          ConfigurableListableBeanFactory beanFactory = getBeanFactory();
  
          /* 3. 添加 ApplicationContextAwareProcessor，让继承自 ApplicationContextAware 的 Bean 对象都能感知所属的 ApplicationContext
          */
          beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
  
          /* 4. 在 Bean 实例化之前，执行 BeanFactoryPostProcessor
                 4.1 收集所有BeanFactoryPostProcessor类型的类为一个Map<String, T>,收集过程中已经开始通过doGetBean实例化而且仅实例化这些类（在getBeanTypeOf的方法中判断bean的类型时）
                 4.2 执行每个BeanFactoryPostProcessor的postProcessBeanFactory方法
        */
          invokeBeanFactoryPostProcessors(beanFactory);
  
          /*5. BeanPostProcessor 需要提前于其他Bean对象实例化之前执行注册操作
                  5.1 收集所有BeanPostProcessor类型的类为一个Map<String, T>,收集过程中已经开始通过doGetBean实例化而且仅实例化这些类（在getBeanTypeOf的方法中判断bean的类型时）
                  5.2 加入所有的BeanPostProcessor为一个list
          */
          registerBeanPostProcessors(beanFactory);
  
          // 6. 初始化事件发布者，创建一个事件发布者
          initApplicationEventMulticaster();
  
          /* 7. 注册事件监听器
             7.1 找到所有类型ApplicationListener.class的类
             7.2 将其放入一个Set<ApplicationListener<ApplicationEvent>> applicationListeners
         */
          registerListeners();
  
          /* 8. 提前实例化单例Bean对象
                  8.1 实例化的方法就是依次调一次getBean方法，最后调用doGetBean方法
            */
          beanFactory.preInstantiateSingletons();
  
          // 9. 发布容器刷新完成事件
          finishRefresh();
      }
  ```

- doGetBean方法
  
  ```java
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

- createBean
  
  ```java
  protected Object createBean(String beanName, BeanDefinition beanDefinition, Object[] args) throws BeansException {
          // 判断是否返回代理 Bean 对象,判断的方法是生成代理对象的类实现了InstantiationAwareBeanPostProcessor，这个接口实现了PostProcessor，在处理PostProcessor时如果能通过InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation方法返回一个切后的对象，那就返回这个对象
          Object bean = resolveBeforeInstantiation(beanName, beanDefinition);
          if (null != bean) {
              return bean;
          }
  
          return doCreateBean(beanName, beanDefinition, args);
      }
  ```

- doCreateBean
  
  ```java
  protected Object doCreateBean(String beanName, BeanDefinition beanDefinition, Object[] args) {
          Object bean = null;
          try {
              // 实例化 Bean
              bean = createBeanInstance(beanDefinition, beanName, args);
  
              // 处理循环依赖，将实例化后的Bean对象提前放入缓存中暴露出来
              if (beanDefinition.isSingleton()) {
                  Object finalBean = bean;
                  //如果是一个代理对象返回代理对象，如果不是返回普通对象，讲这个对象放入三级缓存
                  addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, beanDefinition, finalBean));
              }
  
              // 实例化后判断
              boolean continueWithPropertyPopulation = applyBeanPostProcessorsAfterInstantiation(beanName, bean);
              if (!continueWithPropertyPopulation) {
                  return bean;
              }
              // 在设置 Bean 属性之前，允许 BeanPostProcessor 修改属性值
              applyBeanPostProcessorsBeforeApplyingPropertyValues(beanName, bean, beanDefinition);
              // 给 Bean 填充属性
              applyPropertyValues(beanName, bean, beanDefinition);
              // 执行 Bean 的初始化方法和 BeanPostProcessor 的前置和后置处理方法，先执行前置(postProcessBeforeInitialization)，再执行初始化（afterpropertieset、inti-method），再执行后置(postProcessAfterInitialization)
              bean = initializeBean(beanName, bean, beanDefinition);
          } catch (Exception e) {
              throw new BeansException("Instantiation of bean failed", e);
          }
  
          // 注册实现了 DisposableBean 接口的 Bean 对象
          registerDisposableBeanIfNecessary(beanName, bean, beanDefinition);
  
          // 判断 SCOPE_SINGLETON、SCOPE_PROTOTYPE
          Object exposedObject = bean;
          if (beanDefinition.isSingleton()) {
              // 获取代理对象，放入二级缓存
              exposedObject = getSingleton(beanName);
             //放入一级缓存
              registerSingleton(beanName, exposedObject);
          }
          return exposedObject;
  
      }
  ```

- getSingleton方法
  
  ```java
          // 一级缓存，普通对象，beanName->Bean，其中存储的就是实例化，属性赋值成功之后的单例对象
      private Map<String, Object> singletonObjects = new ConcurrentHashMap<>();
      // 二级缓存，提前暴露对象，早期的单例对象，beanName->Bean，其中存储的是实例化之后，属性未赋值的单例对象
      protected final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>();
      // 三级缓存，存放代理对象，单例工厂的缓存，beanName->ObjectFactory，添加进去的时候实例还未具备属性，用于保存beanName和创建bean的工厂之间的关系map，单例Bean在创建之初过早的暴露出去的Factory，为什么采用工厂方式，是因为有些Bean是需要被代理的，总不能把代理前的暴露出去那就毫无意义了
      private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>();
  public Object getSingleton(String beanName) {
                //首先看一级缓存里是否存在
          Object singletonObject = singletonObjects.get(beanName);
          if (null == singletonObject) {
              singletonObject = earlySingletonObjects.get(beanName);
              // 判断二级缓存中是否有对象，这个对象就是代理对象，因为只有代理对象才会放到三级缓存中
              if (null == singletonObject) {
                  ObjectFactory<?> singletonFactory = singletonFactories.get(beanName);
                  if (singletonFactory != null) {
                      singletonObject = singletonFactory.getObject();
                      // 把三级缓存中的代理对象中的真实对象获取出来，放入二级缓存中
                      earlySingletonObjects.put(beanName, singletonObject);
                      singletonFactories.remove(beanName);
                  }
              }
          }
          return singletonObject;
      }
  ```

- bena的生命周期
  
  <img src="https://s2.loli.net/2022/07/03/VkKNp3nYQPAqmXe.png" alt="spring生命周期.png" style="zoom:25%;" />
  
  > 1. 对象的实例化（相当于new了出来）
  > 
  > 2. 填充属性
  > 
  > 3. 调用BeanNameAware的setBeanName方法
  > 
  > 4. 调用BeanFactoryAware的setBeanFacotry方法
  > 
  > 5. 调用ApplicationContextAware的setApplicationContext方法
  > 
  > 6. 调用BeanPostProcessor的postProcessBeforeInitialization方法
  > 
  > 7. 调用InitializingBean的afterPropertySet方法
  > 
  > 8. 调用自定义的初始化方法
  > 
  > 9. 调用BeanPostProcessor的postProcessAfterInitialization方法
  > 
  > 10. bean可以使用了
  > 
  > 11. 容器关闭时调用DisposableBean的destroy方法
  > 
  > 12. 调用自定义的销毁方法

- AbstractAutoProxyCreator创建代理的流程
  
  > 1. 先确认是否已经创建过代理对象(earlyProxyReferences，避免对代理对象在进行代理)
  > 2. 如果没有，则考虑是否需要进行代理(通过wrapIfNecessary)
  > 3. 如果是特殊的Bean 或者之前判断过不用创建代理的Bean则不创建代理
  > 4. 否则看是否有匹配的Advise(匹配方式就是上文介绍的通过PointCut或者IntroducationAdvisor可以直接匹配类)
  > 5. 如果找到了Advisor，说明需要创建代理，进入createProxy
  > 6. 首先会创建ProxyFactory,这个工厂是用来创建AopProxy的，而AopProxy才是用来创建代理对象的。因为底层代理方式有两种(JDK动态代理和CGLIB，对应到AopProxy的实现就是JdkDynamicAopProxy和ObjenesisCglibAopProxy)，所以这里使用了一个简单工厂的设计。ProxyFactory会设置此次代理的属性，然后根据这些属性选择合适的代理方式，创建代理对象。
  > 7. 创建的对象会替换掉被代理对象(Target)，被保存在BeanFactory.singletonObjects,因此当有其他Bean希望注入Target时，其实已经被注入了Proxy。以上就是Spring实现动态代理的过程。

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
  ```

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

```java
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
      return new A();
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
  
  > spring事务有个特别的地方：它获取的数据库连接放在`ThreadLocal`中的，也就是说同一个线程中从始至终都能获取同一个数据库连接，可以保证同一个线程中多次数据库操作在同一个事务中执行。
  
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
  
  > 1. 只要我们在pom.xml文件中引入spring-boot-starter-data-redis-xxx.jar包，然后只要在配置文件中配置redis连接，如：
  > 
  > ```properties
  > spring.redis.database = 0
  > spring.redis.timeout =10000
  > spring.redis.host = 10.72.16.9
  > spring.redis.port = 6379spring.redis.pattern = 1
  > ```
  > 
  > 就可以在service方法中直接注入`StringRedisTemplate`对象的实例，可以直接使用了。
  > 
  > ```java
  > @Autowired
  > private StringRedisTemplate stringRedisTemplate;
  > ```
  > 
  > 2. 在项目中只要引入spring-boot-starter-xxx.jar，事务就自动生效了，并且可以直接在service方法中直接注入TransactionTemplate，用它开发编程式事务代码。
  > 
  > 3. 使用@ConfigurationProperties可以把指定路径下的属性，直接注入到实体对象中，看看下面这个例子：
  >    
  >    ```properties
  >    jump.threadpool.corePoolSize=8
  >    jump.threadpool.maxPoolSize=16
  >    jump.threadpool.keepAliveSeconds=10
  >    jump.threadpool.queueCapacity=100
  >    ```
  >    
  >    ```java
  >    @Data
  >    @Component
  >    @ConfigurationProperties("jump.threadpool")
  >    public class ThreadPoolProperties {
  >    
  >      private int corePoolSize;
  >      private int maxPoolSize;
  >      private int keepAliveSeconds;
  >      private int queueCapacity;
  >    }
  >    ```

- 自动装配原理
  
  > 1. bean的自动配置
  >    
  >    Spring Boot的启动类上有一个@SpringBootApplication，这个注解包括了@EnableAutoConfiguration，该注解的关键功能由@Import提供，其导入的AutoConfigurationImportSelector的selectImports()方法通过SpringFactoriesLoader.loadFactoryNames()扫描所有具有META-INF/spring.factories的jar包下面key是EnableAutoConfiguration全名的，所有自动配置类。比如springboot的spring-boot-autoconfigure-xxx.jar下的META-INF/spring.factories文件
  > 
  > ```properties
  > org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  > org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
  > org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
  > org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
  > ```
  > 
  > 加载过程大概是这样的：
  > 
  > SpringApplication.run(...)方法->
  > 
  > AbstractApplicationContext.refresh()方法->
  > 
  > invokeBeanFactoryPostProcessors(...)方法->
  > 
  > **PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(...)** 方法->
  > 
  > ConfigurationClassPostProcessor.postProcessBeanDefinitionRegistry(..)方法->
  > 
  > AutoConfigurationImportSelector.selectImports
  > 
  > 该方法会找到自动配置的类，并给打了@Bean注解的方法创建对象。
  > 
  > postProcessBeanDefinitionRegistry方法是最核心的方法，它负责解析@Configuration、@Import、@ImportSource、@Component、@ComponentScan、@Bean等，完成bean的自动配置功能。
  > 
  > 2. 属性的自动配置
  >    
  >    属性的自动配置是通过ConfigurationPropertiesBindingPostProcessor类的postProcessBeforeInitialization方法完成，它会解析@ConfigurationProperties注解上的属性，将配置文件中对应key的值绑定到属性上。
  > 
  > 3. 自动配置的生效条件
  >    
  >    每个xxxxAutoConfiguration类上都可以定义一些生效条件，这些条件基本都是从@Conditional派生出来的。
  >    
  >    ```
  >    @ConditionalOnBean：当容器里有指定的bean时生效@ConditionalOnMissingBean：当容器里不存在指定bean时生效@ConditionalOnClass：当类路径下有指定类时生效@ConditionalOnMissingClass：当类路径下不存在指定类时生效@ConditionalOnProperty：指定的属性是否有指定的值，比如@ConditionalOnProperties(prefix=”xxx.xxx”, value=”enable”, matchIfMissing=true)，代表当xxx.xxx为enable时条件的布尔值为true，如果没有设置的情况下也为true。
  >    ```

# 面经

[参考](https://mp.weixin.qq.com/s/Y17S85ntHm_MLTZMJdtjQQ)

## 特性

Spring 是一个轻量级、非入侵式的控制反转 (IoC) 和面向切面 (AOP) 的框架

```
IOC、DI
AOP
声明式事务
快捷测试
快速集成
复杂API模板封装
```

## 模块

```
Spring Core：Spring 核心，它是框架最基础的部分，提供 IOC 和依赖注入 DI 特性。
Spring Context：Spring 上下文容器，它是 BeanFactory 功能加强的一个子接口。
Spring Web：它提供 Web 应用开发的支持。
Spring MVC：它针对 Web 应用中 MVC 思想的实现。
Spring DAO：提供对 JDBC 抽象层，简化了 JDBC 编码，同时，编码更具有健壮性。
Spring ORM：它支持用于流行的 ORM 框架的整合，比如：Spring + Hibernate、Spring + iBatis、Spring + JDO 的整合等。
Spring AOP：即面向切面编程，它提供了与 AOP 联盟兼容的编程实现。
```

## 常用注解

[参考](https://mp.weixin.qq.com/s/u_O1m7Wyg0icIKffl20U1Q)

- web
  
  @Controller、@RestController、@RequesuMapping(get、post、put、delete)、@ResponseBody、@RequestBody、@PathVariable、@ControllerAdvice（需要和`@ExceptionHandler`、`@InitBinder`以及`@ModelAttribute`注解搭配使用）、@ModelAttribute、@CrossOrigin、

- 容器
  
  @Componet、@Service、@Repository、@Autowired、@Qualifier、@Configuration、@Value、@Bean、@Scope、@ComponentScan、

- AOP
  
  @Aspect、@After、@Before、@Around、@PointCut

- 事务
  
  @Transactional

- Springboot
  
  @SpringBootApplication**、**@EnableAutoConfiguration**、**@ConditionalOnClass与@ConditionalOnMissingClass**、**@ConditionalOnBean与@ConditionalOnMissingBean**、**@ConditionalOnProperty**、**@ConditionalOnResource**、**@ConditionalOnWebApplication与@ConditionalOnNotWebApplication**、**@ConditionalExpression**、**@Conditional

## 设计模式

- 工厂： Spring 容器本质是一个大工厂，使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象
- 代理：Spring AOP 功能功能就是通过代理模式来实现的，分为动态代理和静态代理
- 单例：Spring 中的 Bean 默认都是单例的，这样有利于容器对Bean的管理
- 模板：Spring 中 JdbcTemplate、RestTemplate 等以 Template结尾的对数据库、网络等等进行操作的模板类
- 观察者：Spring 事件驱动模型就是观察者模式很经典的一个应用
- 适配器：Spring AOP 的增强或通知 (Advice) 使用到了适配器模式、Spring MVC 中也是用到了适配器模式适配 Controller
- 策略：Spring中有一个Resource接口，它的不同实现类，会根据不同的策略去访问资源

## BeanFactory和ApplicantContext

```
BeanFactory（Bean工厂）是Spring框架的基础设施，面向Spring本身。
BeanFactory接口位于类结构树的顶端，它最主要的方法就是getBean(String var1)，这个方法从容器中返回特定名称的Bean。
BeanFactory的功能通过其它的接口得到了不断的扩展，比如AbstractAutowireCapableBeanFactory定义了将容器中的Bean按照某种规则（比如按名字匹配、按类型匹配等）进行自动装配的方法。

ApplicantContext（应用上下文）建立在BeanFactoty基础上，面向使用Spring框架的开发者。
ApplicationContext由BeanFactory派生而来，提供了更多面向实际应用的功能,包含 BeanFactory 的所有特性。
ApplicationContext 继承了HierachicalBeanFactory和ListableBeanFactory接口，在此基础上，还通过其他的接口扩展了BeanFactory的功能，包括：
1. Bean instantiation/wiring
2. Bean 的实例化/串联
3. 自动的 BeanPostProcessor 注册
4. 自动的 BeanFactoryPostProcessor 注册
5. 方便的 MessageSource 访问（i18n）
6. ApplicationContext 的发布与 BeanFactory 懒加载的方式不同，它是预加载，所以，每一个 bean 都在 ApplicationContext 启动之后实例化
```

## Spring容器启动阶段会干什么

```
其中容器启动阶段主要做的工作是加载和解析配置文件，保存到对应的Bean定义中。
```

## 有哪些依赖注入的方法

```
Spring支持构造方法注入、属性注入、工厂方法注入,其中工厂方法注入，又可以分为静态工厂方法注入和非静态工厂方法注入。
```

## 有哪些自动装配的方式

```
Spring IOC容器知道所有Bean的配置信息，此外，通过Java反射机制还可以获知实现类的结构信息，如构造方法的结构、属性等信息。掌握所有Bean的这些信息后，Spring IOC容器就可以按照某种规则对容器中的Bean进行自动装配，而无须通过显式的方式进行依赖配置。

byName：根据名称进行自动匹配，假设Boss又一个名为car的属性，如果容器中刚好有一个名为car的bean，Spring就会自动将其装配给Boss的car属性
byType：根据类型进行自动匹配，假设Boss有一个Car类型的属性，如果容器中刚好有一个Car类型的Bean，Spring就会自动将其装配给Boss这个属性
constructor：与 byType类似， 只不过它是针对构造函数注入而言的。如果Boss有一个构造函数，构造函数包含一个Car类型的入参，如果容器中有一个Car类型的Bean，则Spring将自动把这个Bean作为Boss构造函数的入参；如果容器中没有找到和构造函数入参匹配类型的Bean，则Spring将抛出异常。
autodetect：根据Bean的自省机制决定采用byType还是constructor进行自动装配，如果Bean提供了默认的构造函数，则采用byType，否则采用constructor
```

## Bean 的作用域有哪些

```
singleton : 在Spring容器仅存在一个Bean实例，Bean以单实例的方式存在，是Bean默认的作用域。
prototype : 每次从容器重调用Bean时，都会返回一个新的实例。
以下三个作用域于只在Web应用中适用：

request : 每一次HTTP请求都会产生一个新的Bean，该Bean仅在当前HTTP Request内有效。
session : 同一个HTTP Session共享一个Bean，不同的HTTP Session使用不同的Bean。
globalSession：同一个全局Session共享一个Bean，只用于基于Protlet的Web应用，Spring5中已经不存在了。
```

## 单例 Bean 会存在线程安全问题吗

```
首先结论在这：Spring中的单例Bean不是线程安全的。

因为单例Bean，是全局只有一个Bean，所有线程共享。如果说单例Bean，是一个无状态的，也就是线程中的操作不会对Bean中的成员变量执行查询以外的操作，那么这个单例Bean是线程安全的。比如Spring mvc 的 Controller、Service、Dao等，这些Bean大多是无状态的，只关注于方法本身。

假如这个Bean是有状态的，也就是会对Bean中的成员变量进行写操作，那么可能就存在线程安全的问题。
常见的有这么些解决办法：
1. 将Bean定义为多例
这样每一个线程请求过来都会创建一个新的Bean，但是这样容器就不好管理Bean，不能这么办。
2. 在Bean对象中尽量避免定义可变的成员变量
削足适履了属于是，也不能这么干。
3. 将Bean中的成员变量保存在ThreadLocal中⭐
我们知道ThredLoca能保证多线程下变量的隔离，可以在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal里，这是推荐的一种方式。
```

## 循环引用

- spring解决不了构造方法参数的循环依赖，A的构造方法里调用了B的方法，B的构造方法里调用了A的方法，谁也解决不了，能解决的，只是类成员变量（具有set方法）的循环依赖。A里有B，B里有A，并且各自都有set方法。

- 实例化，放到二级缓存，初始化，放到一级缓存，完事。

- 如果类型A与B发生了循环依赖，那它的创建过程就是：实例化A，放到二级缓存，实例化B，放到二级缓存，初始化B（从二级缓存拿到A的引用），将B放到一级缓存，初始化A，将A放到一级缓存，完事！

- 关于三级引用，如果使用了三级缓存，A中有代理的情况下创建bean的过程如下[参考](https://www.jianshu.com/p/abda18eaa848)
  
  ```
  开始创建A->实例化A->为A注入属性->开始创建B->实例化B->为B注入属性->为A创建代理->初始化B->结束创建B->初始化A->结束创建A
  ```
  
  如果不使用三级缓存，A中有代理的情况下创建bean的过程如下
  
  ```
  开始创建A->实例化A->为A创建代理->为A注入属性->开始创建B->实例化B->为B注入属性->初始化B->结束创建B->初始化A->结束创建A
  ```
  
  上面两个流程的唯一区别在于为A对象创建代理的时机不同，在使用了三级缓存的情况下为A创建代理的时机是在B中需要注入A的时候，而不使用三级缓存的话在A实例化后就需要马上为A创建代理然后放入到二级缓存中去。对于整个A、B的创建过程而言，消耗的时间是一样的（所以常见的三级缓存提高了效率这种说法都是错误的）
  
  上述这种情况下，差别就是在哪里创建代理。如果不用三级缓存，使用二级缓存，违背了Spring在结合AOP跟Bean的生命周期的设计！Spring结合AOP跟Bean的生命周期本身就是通过AbstractAutoProxyCreator这个后置处理器来完成的，在这个后置处理的postProcessAfterInitialization方法中对初始化后的Bean完成AOP代理。如果出现了循环依赖，那没有办法，只有给Bean先创建代理，但是没有出现循环依赖的情况下，设计之初就是让Bean在生命周期的最后一步完成代理而不是在实例化后就立马完成代理。

- Spring可以解决哪些情况的循环依赖？
  
  ```
  当循环依赖的实例都采用setter方法注入的时候，Spring可以支持
  都采用构造器注入的时候，不支持
  构造器注入和setter注入同时存在的时候，看天(Spring 在创建 Bean 时默认会根据自然排序进行创建)
  ```

## @Autowired的实现原理

```
实现@Autowired的关键是：AutowiredAnnotationBeanPostProcessor

在Bean的初始化阶段，会通过Bean后置处理器来进行一些前置和后置的处理。

实现@Autowired的功能，也是通过后置处理器来完成的。这个后置处理器就是AutowiredAnnotationBeanPostProcessor。

Spring在创建bean的过程中，最终会调用到doCreateBean()方法，在doCreateBean()方法中会调用populateBean()方法，来为bean进行属性填充，完成自动装配等工作。

在populateBean()方法中一共调用了两次后置处理器，第一次是为了判断是否需要属性填充，如果不需要进行属性填充，那么就会直接进行return，如果需要进行属性填充，那么方法就会继续向下执行，后面会进行第二次后置处理器的调用，这个时候，就会调用到AutowiredAnnotationBeanPostProcessor的postProcessPropertyValues()方法，在该方法中就会进行@Autowired注解的解析，然后实现自动装配。
```

## AOP有哪些概念

- **切面**（Aspect）：类是对物体特征的抽象，切面就是对横切关注点的抽象

- **连接点**（Joinpoint）：被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在 Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器

- **切点**（Pointcut）：对连接点进行拦截的定位

- **通知**（Advice）：所谓通知指的就是指拦截到连接点之后要执行的代码，也可以称作**增强**

- **目标对象** （Target）：代理的目标对象

- **织入**（Weabing）：织入是将增强添加到目标类的具体连接点上的过程。
  
  - 编译期织入：切面在目标类编译时被织入
  
  - 类加载期织入：切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。
  
  - 运行期织入：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面。
    
    Spring采用运行期织入，而AspectJ采用编译期织入和类加载器织入。

- **AOP有哪些环绕方式**

- - 前置通知 (@Before)
  - 返回通知 (@AfterReturning)
  - 异常通知 (@AfterThrowing)
  - 后置通知 (@After)
  - 环绕通知 (@Around)

## 动态代理

- JDK 动态代理和 CGLIB 代理
  
  ```
  JDK 动态代理
  Interface：对于 JDK 动态代理，目标类需要实现一个Interface。
  InvocationHandler：InvocationHandler是一个接口，可以通过实现这个接口，定义横切逻辑，再通过反射机制（invoke）调用目标类的代码，在次过程，可能包装逻辑，对目标方法进行前置后置处理。
  Proxy：Proxy利用InvocationHandler动态创建一个符合目标类实现的接口的实例，生成目标类的代理对象。
  CgLib 动态代理
  
  CgLib 动态代理是使用字节码处理框架 ASM，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。
  CgLib 创建的动态代理对象性能比 JDK 创建的动态代理对象的性能高不少，但是 CGLib 在创建代理对象时所花费的时间却比 JDK 多得多，所以对于单例的对象，因为无需频繁创建对象，用 CGLib 合适，反之，使用 JDK 方式要更为合适一些。同时，由于 CGLib 由于是采用动态创建子类的方法，对于 final 方法，无法进行代理。
  ```

- 动态代理的使用场景
  
  ```
  1. AOP 切面编程：为切点配置包含的类生成代理 bean 实例。
  2. @lazy：懒加载，被注解的bean 不是不实例化，而是先创建一个代理bean 注入。
  3. @Configuration：配置类注解，生成的配置类bean 实例是代理，执行@bean 的方法之前 先判断单例池中是否已有该对象实例，确保@bean 注解的对象单例。
  4. @Async ：动态生成 实现异步调用的代理类。
  ```

- 什么是提前进行AOP？
  
  ```
  在循环引用中如果循环应用的对象涉及到AOP，那就不得不提前进行AOP,除非使用第三级缓存
  ```

- @Lazy解决循环依赖
  
  ```java
  @Service
  public class A extends GenericBaseService {
      @Autowired
      @Lazy
      private B b;
  }
  加了lazy的原理如下:
  1. A的创建: A a=new A();
  2. 属性注入:发现需要B,查询字段b的所有注解,发现有@lazy注解,那么就不直接创建B了,而是使用动态代理创建一个代理类B
  3. 此时A跟B就不是相互依赖了,变成了A依赖一个代理类B1,B依赖A
  Spring构造器注入循环依赖的解决方案是@Lazy，其基本思路是：对于强依赖的对象，一开始并不注入对象本身，而是注入其代理对象，以便顺利完成实例的构造，形成一个完整的对象，这样与其它应用层对象就不会形成互相依赖的关系；当需要调用真实对象的方法时，通过TargetSource去拿到真实的对象[DefaultListableBeanFactory#doResolveDependency]，然后通过反射完成调用
  ```

## Spring 事务

- 声明式事务实现原理
  
  ```
  1. 查找@Transactional标记的方法
  2. TransactionInterceptor （事务拦截器）在目标方法执行前后进行拦截
     2.1 事务管理器PlatformTransactionManager新建一个数据库连接，将自动提交设置为false
     2.2 使用try...catch包裹原来的方法，正常时commit，异常时rollback
  3. 真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的
  ```

- 传播机制
  
  ```
  PROPAGATION_REQUIRED:支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是 Spring 默认的事务的传播。
  PROPAGATION_REQUIRES_NEW:新建事务，如果当前存在事务，把当前事务挂起。新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作
  PROPAGATION_SUPPORTS:支持当前事务，如果当前没有事务，就以非事务方式执行。
  PROPAGATION_MANDATORY:支持当前事务，如果当前没有事务，就抛出异常。
  PROPAGATION_NOT_SUPPORTED:以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
  PROPAGATION_NEVER:以非事务方式执行，如果当前存在事务，则抛出异常。
  PROPAGATION_NESTED:如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器起效。
  ```

- 失效情况
  
  ```
  1、数据库引擎不支持事务（InnoDB才支持）
  2、没有被 Spring 管理（类上没有打@Service等@component的子注解）
  3、方法不是 public 的（切面无法切入到private方法）
  4、在非事务方法中调用事务方法或者在同一个类中一个事务方法调用另一个事务方法（同一个类之中，方法互相调用，切面无效 ，而不仅仅是事务）
  5、数据源没有配置事务管理器
  6、异常被吞掉了
  7、异常类型错误（默认是RuntimeException，如果要抛出其他异常需要显示指定）
  ```

- Spring事务与数据库事务的关系
  
  ```
  Spring事务隔离级别是在数据库隔离级别之上又进一步进行了封装。
  1.数据库是可以控制事务的传播和隔离级别的，Spring在之上又进一步进行了封装，可以在不同的项目、不同的操作中再次对事务的传播行为和隔离级别进行策略控制，spring中设置的隔离是数据库的会话隔离级别（数据库还有全局隔离）
  2.项目中，以 Spring 事务为准，因为他重写了数据库的隔离级别，但没有直接修改数据库的隔离级别；
  3.项目中，如果 Spring 事务隔离级别设置为（isolation = Isolation.DEFAULT）默认数据库优先时，以数据库的隔离级别为准。
  ```

- 编程式事务和声明式事务
  
  ```java
  声明式事务注解方式3个步骤
  //1. 在spring配置类上加上@EnableTransactionManagement注解
  @EnableTransactionManagement
  public class MainConfig4 {
  }
  //2. 定义一个事务管理器
  @Bean
  public PlatformTransactionManager transactionManager(DataSource dataSource) {
      return new DataSourceTransactionManager(dataSource);
  }
  //3. 使用事务的目标上加@Transaction注解
  @Transactional
  public void insertBatch(String... names) {
    jdbcTemplate.update("truncate table t_user");
    for (String name : names) {
      jdbcTemplate.update("INSERT INTO t_user(name) VALUES (?)", name);
    }
  }
  
  编程式事务
  //1.使用TransactionTemplate
  @Autowired
  private TransactionTemplate transactionTemplate;
  
  /**
       * 转账方法
       * @param from 金额减少的用户
       * @param to 金额增加的用户
       * @param money 转账的金额
       */
  public void transferMoney(String from, String to, Double money) {
    //可以用lamda表达式替换
    transactionTemplate.execute(new TransactionCallbackWithoutResult() {
      @Override
      protected void doInTransactionWithoutResult(TransactionStatus status) {
        try {
          accountDao.minusMoney(from, money);
          accountDao.addMoney(to, money);
        } catch (Exception e) {
          e.printStackTrace();
          //回滚
          status.setRollbackOnly();
        }
      }
    });
  }
  //2. 使用TransactionManager
  @Autowired
  private PlatformTransactionManager platformTransactionManager;
  
  public void TransferMoney(String from, String to, Double money) {
    DefaultTransactionDefinition definition = new DefaultTransactionDefinition();
    //开启事务
    TransactionStatus status = platformTransactionManager.getTransaction(definition);
    try {
      accountDao.minusMoney(from, money);
      accountDao.addMoney(to, money);
      platformTransactionManager.commit(status);
    } catch (Exception e) {
      e.printStackTrace();
      platformTransactionManager.rollback(status);
    }
  }
  ```

- 如何确定方法有没有用到spring事务
  
  ```
  方式1：断点调试
  spring事务是由TransactionInterceptor拦截器处理的，最后会调用下面这个方法，设置个断点就可以看到详细过程了。
  
  方式2：看日志
  spring处理事务的过程，有详细的日志输出，开启日志，控制台就可以看到事务的详细过程了。
  ```

## @PostConstruct底层原理

```
@PostConstruct底层原理
spring的Bean在创建的时候会进行初始化，而初始化过程会解析出@PostConstruct注解的方法，并反射调用该方法。从而，在启动的时候该方法被执行,执行的时机是PostProcessor的前置方法
```

## @Configuration原理

```
@Configuration类中的@Bean地方会被CGLIB进行代理。Spring会拦截该方法的执行，在默认单例情况下，容器中只有一个Bean，所以我们多次调用user()方法，获取的都是同一个对象。

对于@Configuration注解的类中@Bean标记的方法，返回的都是一个bean，在增强的方法中，Spring会先去容器中查看一下是否有这个bean的实例了，如果有了的话，就返回已有对象，没有的话就创建一个，然后放到容器中。
```

## MVC核心组件

1. **DispatcherServlet**：前置控制器，是整个流程控制的**核心**，控制其他组件的执行，进行统一调度，降低组件之间的耦合性，相当于总指挥。
2. **Handler**：处理器，完成具体的业务逻辑，相当于 Servlet 或 Action。
3. **HandlerMapping**：DispatcherServlet 接收到请求之后，通过 HandlerMapping 将不同的请求映射到不同的 Handler。
4. **HandlerInterceptor**：处理器拦截器，是一个接口，如果需要完成一些拦截处理，可以实现该接口。
5. **HandlerExecutionChain**：处理器执行链，包括两部分内容：Handler 和 HandlerInterceptor（系统会有一个默认的 HandlerInterceptor，如果需要额外设置拦截，可以添加拦截器）。
6. **HandlerAdapter**：处理器适配器，Handler 执行业务方法之前，需要进行一系列的操作，包括表单数据的验证、数据类型的转换、将表单数据封装到 JavaBean 等，这些操作都是由 HandlerApater 来完成，开发者只需将注意力集中业务逻辑的处理上，DispatcherServlet 通过 HandlerAdapter 执行不同的 Handler。
7. **ModelAndView**：装载了模型数据和视图信息，作为 Handler 的处理结果，返回给 DispatcherServlet。
8. **ViewResolver**：视图解析器，DispatcheServlet 通过它将逻辑视图解析为物理视图，最终将渲染结果响应给客户端。

## SpringMVC Restful风格的接口的流程

1. 客户端向服务端发送一次请求，这个请求会先到前端控制器DispatcherServlet

2. DispatcherServlet接收到请求后会调用HandlerMapping处理器映射器。由此得知，该请求该由哪个Controller来处理

3. DispatcherServlet调用HandlerAdapter处理器适配器，告诉处理器适配器应该要去执行哪个Controller

4. Controller被封装成了ServletInvocableHandlerMethod，HandlerAdapter处理器适配器去执行invokeAndHandle方法，完成对Controller的请求处理

5. HandlerAdapter执行完对Controller的请求，会调用HandlerMethodReturnValueHandler去处理返回值，主要的过程：
   
   5.1. 调用RequestResponseBodyMethodProcessor，创建ServletServerHttpResponse（Spring对原生ServerHttpResponse的封装）实例
   
   5.2.使用HttpMessageConverter的write方法，将返回值写入ServletServerHttpResponse的OutputStream输出流中
   
   5.3.在写入的过程中，会使用JsonGenerator（默认使用Jackson框架）对返回值进行Json序列化

6. 执行完请求后，返回的ModealAndView为null，ServletServerHttpResponse里也已经写入了响应，所以不用关心View的处理

## SpringBoot

- 自动配置原理
- 如何自定义一个SpringBoot Srarter
- Springboot 启动原理

## SpringCloud

- 什么是微服务？
- 微服务架构主要要解决哪些问题？
- 有哪些主流微服务框架？
- SpringCloud有哪些核心组件？

```properties
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

# 其它

## 其它参考

[一些springboot的示例](https://github.com/smltq/spring-boot-demo)  [循环依赖](https://mp.weixin.qq.com/s/5VHU2qRQMPL0IOZuEOPmQA)
