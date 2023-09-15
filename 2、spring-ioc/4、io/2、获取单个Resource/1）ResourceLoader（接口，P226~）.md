
ResourceLoader接口提供getResource()方法。

根据源码说明，getResource(String location)方法返回的是一个Resource handle。

关于Resource handle：
- 代表的资源并不一定存在，还需要通过 Resource.exists()来进一步判断。
- 是一个可重复使用的资源描述符（resource descriptor），允许多次Resource.getInputStream()调用。

根据源码说明，ResourceLoader接口的实现类提供的getResource(String location)：
- 必须支持全限定URL，如file:C:/test.dat。
- 必须支持classpath路径，如classpath:test.dat。
- 应该支持相对路径，如WEB-INF/test.dat。ApplicationContext实现往往提供。

下面是ResourceLoader接口的定义：
```java
public interface ResourceLoader {   
	Resource getResource(String location);   
	ClassLoader getClassLoader();
}
```

# 内置实现类

ResourceLoader接口主要有3个实现类：
## 1）DefaultResourceLoader

AbstractApplicationContext抽象类  继承了 DefaultResourceLoader类。

DefaultResourceLoader能单独使用。

- 如果传递给getResource()方法的是 URL String（即`"file:"`、`https:`前缀的String），那么会返回URLResource实例。
- 如果传递给getResource()方法的是 "classpath:" 前缀的伪URL，那么会返回ClassPathResource实例。
- 如果没有前缀，那么根据ApplicationContext的具体类型确定返回的Resource具体类型。











