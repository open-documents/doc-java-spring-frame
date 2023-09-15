
在Spring容器里的对象可以通过@Autowired来注入，但是如果这个对象是你自己new出来的，恐怕很难通过@Autowired来获得对象了。

@Configurable这个注解就是为了给非Spring容器管理的对象提供注入Spring容器内对象的一个注解。

Spring生成一个新的和标注类型一致的实例，默认名称是全类名（`com.xyz.myapp.domain.Account`）。
```java
package com.xyz.myapp.domain
@Configurable 
public class Account {   
	// ...
}
```
```xml
<bean class="com.xyz.myapp.domain.Account" scope="prototype">   
	<property name="fundsTransferService" ref="fundsTransferService"/>
</bean>
```

# 指定名称
```java
@Configurable("account") 
public class Account {   
	// ...
}
```
Spring寻找一个叫account的bean definition，然后用它来配置一个新的Account实例。

# preConstruction属性

	参考文档:P385

被@Configurable注解标注的类，先实例，后初始化后，根据@Configurable注解中的属性，使用Spring来对新实例化的对象进行配置。

“初始化”是针对新实例化的实例（例如通过new关键字实例的实例）。

“初始化后意味着”：新实例化的对象的依赖已经注入完成。这就意味着构造函数内不能访问依赖。

如果想要在构造函数运行之前注入依赖，以使依赖能在构造函数中访问到，使用@Configurable的preConstruction属性。
```java
@Configurable(preConstruction = true)
```



# 启用@Configurable

在任意@Configuration类上标注@EnableSpringConfigured注解。
```java
@Configuration 
@EnableSpringConfigured 
public class AppConfig { }
```
使用context命名空间下的`<context:spring-configured/>`标签。
```xml
<context:spring-configured/>
```