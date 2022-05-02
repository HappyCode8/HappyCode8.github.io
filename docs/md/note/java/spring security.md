# spring security

- principal：简单理解为用户名

- credentials：简单理解为密码

- UsernamePasswordAuthenticationToken(Object principal, Object credentials)//创建了一个token,里边有一些用户名等资料

## 原理

![img](./images/up-9fe26f325971ba04422c382a32b58a1f76d.png)

​        图片来源[参考](https://www.e-learn.cn/topic/3143567)

​        由上图我们可以看到，Spring Security其实就是一个过滤器链，它里面有很多很多的过滤器，就图上的第一个过滤器UsernamePasswordAuthenticationFilter是用来做表单认证过滤的；如果我们没有配置表单认证，而是Basic认证，则第二个过滤器BasicAuthenticationFilter会发挥作用。最后一个FilterSecurityInterceptor则是用来最后一个过滤器，它的作用是用来根据前面的过滤器是否生效以及生效的结果来判断你的请求是否可以访问REST接口。如果无法通过FilterSecurityInterceptor的判断的情况下，会抛出异常。而ExceptionTranslationFIlter会捕获抛出的异常来进行相应的处理。

## 