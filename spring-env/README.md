### 谈谈SpringCloud SnameYAML RCE（★★★）

该漏洞的利用条件是可出网，可以`POST`访问`/env`接口设置属性，且可以访问`/refresh`刷新配置

在`VPS`上开启`HTTP Server`并放入基于`ScriptEngineManager`和`URLClassLoader`的`yml`，制作特殊的JAR并指定

通过`/env`设置`spring.cloud.bootstrap.location`属性再刷新配置即可利用`SnakeYAML`的反序列化漏洞实现`RCE`



### 谈谈SpringCloud Eureka RCE（★★★）

该漏洞的利用条件同样是可出网，可以`POST`访问`/env`接口设置属性，且可以访问`/refresh`刷新配置

首先搭建恶意的`XStream Server`其中包含了`Payload`

通过`/env`设置`eureka.client.serviceUrl.defaultZone`属性再刷新配置即可访问远程`XStream Payload`触发反序列化达到`RCE`



### 谈谈SpringBoot Jolokia Logback JNDI RCE（★★★）

如果目标可出网且存在`/jolokia`或`/actuator/jolokia`接口

通过`/jolokia/list`查看是否存在`ch.qos.logback.classic.jmx.JMXConfigurator`和`reloadByURL`关键词

搭建一个`HTTP Server`保存`XML`配置文件，再启动恶意的`JNDI Server`，请求指定的`URL`即可触发`JNDI`注入漏洞达到`RCE`



### 谈谈SpringBoot Jolokia Realm JNDI RCE（★★★）

如果目标可出网且存在`/jolokia`或`/actuator/jolokia`接口

启动恶意的`JNDI Server`后调用`createJNDIRealm`创建`JNDIRealm`，然后写入`JNDI`相关的配置文件，重启后触发`JNDI`注入漏洞达到`RCE`



### 谈谈SpringBoot Restart H2 Database Query RCE（★★★）

漏洞利用条件是可以访问`/env`设置属性，可以访问`/restart`重启应用

设置`spring.datasource.hikari.connection-test-query`属性为创建自定义函数的`SQL`语句，再数据库连接之前会执行该`SQL`语句

通过重启应用，建立新的数据库连接，触发包含命令执行的自定义函数，达到`RCE`



### 谈谈SpringBoot H2 Database Console JNDI RCE（★★★）

目标可出网且存在`spring.h2.console.enabled=true`属性时可以利用

首先通过`/h2-console`中记录下`JSESSIONID`值，然后启动恶意的`JNDI Server`，构造对应域名和`JSESSIONID`的特殊`POST`请求触发`JNDI`注入漏洞达到`RCE`



### 谈谈SpringBoot Mysql JDBC RCE（★★★）

该漏洞的利用条件同样是可出网，可以`POST`访问`/env`接口设置属性，且可以访问`/refresh`刷新配置，不同的是需要存在`mysql-connector-java`依赖

通过`/env`得到`mysql`版本等信息，测试是否存在常见的`Gadget`依赖，访问`spring.datasource.url`设置指定的`MySQL JDBC URL`地址。刷新配置后当网站进行数据库操作时，会使用恶意的`MySQL JDBC URL`建立连接

恶意的`MySQL Server`会返回序列化`Payload`数据，利用本地的`Gadget`反序列化造成`RCE`



### 谈谈SpringBoot Restart logging.config Logback JNDI RCE（★★★）

该漏洞的利用条件同样是可出网，可以`POST`访问`/env`接口设置属性，且可以访问`/restart`重启

搭建一个`HTTP Server`保存`XML`配置文件，再启动恶意的`JNDI Server`

通过`/env`接口设置`logging.config`属性为恶意的`XML`配置文件，重启触发`JNDI`注入漏洞达到`RCE`



### 谈谈SpringBoot Restart logging.config Groovy RCE（★★★）

该漏洞的利用条件同样是可出网，可以`POST`访问`/env`接口设置属性，且可以访问`/restart`重启

启动恶意的`JNDI Server`并通过`/env`接口设置`logging.config`属性为恶意的`Groovy`文件在重启后生效

在`logback-classic`组件的`ch.qos.logback.classic.util.ContextInitializer`中会判断`url`是否以`groovy`结尾，如果是则最终会执行文件内容中的`groovy`代码达到`RCE`




### 谈谈SpringBoot Restart spring.main.sources Groovy RCE（★★★）

类似SpringBoot Restart logging.config Groovy RCE

组件中的`org.springframework.boot.BeanDefinitionLoader`判断`url`是否以`groovy`结尾，如果是则最终会执行文件内容中的`groovy`代码，造成`RCE`漏洞



### 谈谈SpringBoot Restart spring.datasource.data H2 Database RCE（★★★）

该漏洞的利用条件同样是可出网，可以`POST`访问`/env`接口设置属性，且可以访问`/restart`重启

开一个`HTTP Server`保存恶意SQL语句，这是一个执行命令的函数，设置属性`spring.datasource.data`为该地址，重启后设置生效

组件中的`org.springframework.boot.autoconfigure.jdbc.DataSourceInitializer`使用`runScripts`方法执行请求`URL`内容中的`SQL`代码，造成`RCE`漏洞