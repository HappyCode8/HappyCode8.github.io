# Postman

## 测试接口输入对象

- 方法选择Post
- headers中的`Content-Type`选择`application/json`
- body选择raw,形式选json，下边输入一个json包含对象属性即可

## BA鉴权

Pre-request Script脚本

```js
//设置id和secrent
var client_secret='fd6652baad314c5189c30702456d3c47';
var client_id = '22d8c3a80f38e535'
//获取http时间
var timespan = new Date().toUTCString();
//通过request.data获取body的内容，这个是postman内置变量
var method = request.method;
var uri = pm.request.url.getPath();
string_to_sign = method + " " + uri + "\n" + timespan;
console.log(string_to_sign);
//使用CryptoJS,postman的内置js库加密
var hash = CryptoJS.HmacSHA1(string_to_sign, client_secret);
console.log(hash);
var signature = CryptoJS.enc.Base64.stringify(hash);
console.log(signature);
var Authorization = "MWS" + " " + client_id + ":" + signature;
console.log(Authorization);
//设置环境变量
postman.setEnvironmentVariable("Authorization", Authorization.toString());
postman.setEnvironmentVariable("Date", timespan);
```

在请求头中加入如下参数

```
Date {{Date}}
Authorization {{Authorization}}
```

常见的时间戳、随机数、随机字符串、MD5等的测试

```js
//设置当前时间戳
postman.setGlobalVariable("time",Math.round(new Date().getTime()));
postman.setGlobalVariable("time",Math.round(new Date()/1000));//标注平台接收这一种
time = postman.getGlobalVariable('time');

postman.setGlobalVariable("rand",1+Math.round(Math.random() * 1000));
random=postman.getGlobalVariable('rand');

postman.setGlobalVariable("key","bfd94fa7-3612-579d-9714-fc9178789ab9");
key=postman.getGlobalVariable('key');

postman.setGlobalVariable("taskid", ("wangyingjie09" + (Math.random()*Math.pow(36,4) << 0).toString(36))); 
var taskid=postman.getGlobalVariable("taskid")

var str = key+taskid+time+random;
var strmd5= CryptoJS.MD5(str).toString();
postman.setGlobalVariable("sign",strmd5)
```

**有一些业务方规定了数据不能以body形式提交，而是要以参数的形式提交，那么以表单的形式提交可以代替参数形式，这样避免了URL长度的限制**

贴出页面的cookie，在控制台输出

```
document.cookie
```

# Jemeter

- 启动

  >bin目录下，./jmeter启动

- HTTP接口压测

  >添加线程组：add->threads->thread group
  >
  >在线程组下添加http请求：add->samper->http request
  >
  >在http请求下添加一个assertation：add->assertations->
  >
  >添加结果：add->listener->

# wrk

- 压测命令

  >10线程50连接压测10秒
  >
  >wrk -t 10 -c 50 -d 10s http://localhost:8080/redis --latency

