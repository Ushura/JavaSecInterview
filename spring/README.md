## Spring

### 谈谈Spring Whitelabel SPEL RCE（★★★）

Spring处理参数值出错时会将参数中`${}`中的内容当作`SPEL`解析实现，造成`RCE`漏洞



### 谈谈Spring Data REST SPEL RCE（★★）

当使用`JSON PATCH`对数据修改时，传入的`PATH`参数会解析`SPEL`



### 谈谈Spring Web Flow SPEL RCE（★★）

在`Model`的数据绑定上存在漏洞，但漏洞出发条件比较苛刻

由于没有明确指定相关`Model`的具体属性，导致从表单可以提交恶意的表达式`SPEL`被执行



### 谈谈Spring Messaging SPEL RCE（★★）

其中的`STOMP`模块发送订阅命令时，支持选择器标头，该选择器充当基于内容路由的筛选器

这个筛选器`selector`属性的值会解析`SPEL`导致RCE



### 谈谈Spring Data Commons SPEL RCE（★★）

请求参数中如何包含`SPEL`会被解析，参考下方Payload

```text
username[#this.getClass().forName("java.lang.Runtime").getRuntime().exec("calc.exe")]
```



### 谈谈最新的Spring Cloud Gateway SPEL的RCE漏洞（★★★）

本质还是`SPEL`表达式，本来这是一个需要修改配置文件导致的鸡肋`RCE`漏洞

但因为`Gateway`提供了`Actuator`相关的`API`可以动态地注册`Filter`，而在注册的过程中可以设置`SPEL`表达式

实战利用程度可能不高，目标未必开着`Actuator`接口，就算开放也不一定可以正常访问注册`Filter`的接口



### Spring Cloud Gateway SPEL的RCE漏洞可以回显吗（★★★★）

P牛在漏洞爆出的凌晨就发布了相关的环境和POC

参考P牛的回显代码：在相应头里面添加一个新的头，利用工具类把执行回显写入

```json
{
    "name": "AddResponseHeader",
    "args": {
        "value": "#{new java.lang.String(T(org.springframework.util.StreamUtils).copyToByteArray(T(java.lang.Runtime).getRuntime().exec(new String[]{\"whoami\"}).getInputStream()))}",
        "name": "cmd123"
    }
}
```



### Spring Cloud Gateway SPEL的RCE漏洞如何修复的（★★）

参考很多`SPEL`漏洞的修复手段，默认情况使用`StandardContext`可以执行`Runtime.getRuntime().exec()`这样的危险方法

修复是重写一个`GatewayContext`用来执行`SPEL`，这个`context`的底层是`SimpleEvaluationContext`只能执行有限的操作



### Spring Cloud Function RCE漏洞了解吗（★★）

这也是`Spring Cloud`种的一个组件，不过并不常用

利用方式是某个请求头支持`SpEL`的格式并且会执行

```http
POST / HTTP/1.1
...
spring.cloud.function.routing-expression: SPEL
```

修复方案比较简单，使用`SimpleEvaluationContext`即可



### SPEL拒绝服务漏洞了解吗（★★）

参考先知社区`4ra1n`师傅的文章：https://xz.aliyun.com/t/11114

危害不大，但影响较广，所有能够执行`SpEL`的框架，都可以通过初始化巨大的数组造成拒绝服务漏洞

修复方案是限制`SpEL`种数组初始化的长度（一般业务也不可能在`SpEL`种初始化很大的数组）



### 谈谈Spring RCE的基本原理（★★★★）

该漏洞与很久以前的`SpringMVC`对象绑定漏洞有关，曾经的修复方案是：如果攻击者尝试以`class.classloader`获取任意`class`对象的`loader`时跳过

这里的对象绑定是指将请求中的参数绑定到控制器（Controller）方法中的参数对象的成员变量，例如通过`username`和`password`等参数绑定到`User`对象

由于在`JDK9`中加入了模块`module`功能，可以通过`class.module.classLoader`得到某`class`对应的`classloader`进而利用

在`Tomcat`环境下拿到的`classloader`对象中包含了`context`，进而通过`pipeline`拿到`AccessLogValue`对象，该类用于处理`Tomcat`访问日志相关。通过修改其中的字段信息，可以将`webshell`写入指定目录下的指定文件中，以达到`RCE`的目的



### 谈谈Spring RCE的利用条件（★★★）

1. JDK9+（核心是利用到`module`功能）
2. Tomcat（为了拿到可利用的`Classloader`对象）
3. 必须存在对象绑定，如果是`String`和`int`等基本类型参数则不生效



### Spring RCE为什么在SpringBoot中不生效（★★★★）

因为在`SpringBoot`中拿到的`classloader`是`AppClassloader`类，该类不存在无参的`getResources`方法且没有其他可操作的空间，所以无法利用



### 谈谈Spring RCE的修复（★★★）

当`beanClass`为`Class`时只允许参数名为`name`并以`Name`结尾且属性返回类型不能为`Classloader`及`Classloader`子类