# Small-Spring

## Step1

​		BeanFactory是一个里面包含了hashmap对象的类，当向BeanFactory注册一个对象的时候，就是把一个对象用BeanDefination包裹起来，然后put到hashmap里，get的时候根据名字从hashmap里边拿出来，做个类型强转。
​        BeanFactory beanFactory = new BeanFactory();
​        BeanDefinition beanDefinition = new BeanDefinition(new UserService());
​        beanFactory.registerBeanDefinition("userService", beanDefinition);
​        UserService userService = (UserService) beanFactory.getBean("userService");
​        userService.queryUserInfo();

## Step2

![image-20210920212153292](./images/image-20210920212153292.png)

- 首先BeanDefinition包裹的对象由Object变成了Class，此后向Spring容器注册Class而不是Object
- BeanFactory只是获取Bean，注册Bean的方法放到BeanDefinitionRegistry接口
- DefaultSingletonBeanRegistry实现了SingletonBeanRegistry接口，实现仍然是一个HashMap，可以放入Map里一个Object，也可以得到
- AbstractBeanFactory继承了DefaultSingletonBeanRegistry，实现了BeanFactory接口，它里边有两个抽象方法，getBean时首先得到