
下面是ResourcePatternResolver接口的定义：
```java
public interface ResourcePatternResolver extends ResourceLoader {
	String CLASSPATH_ALL_URL_PREFIX = "classpath*:";
	Resource[] getResources(String locationPattern) throws IOException;
}
```
ResourcePatternResolver接口是ResourceLoader 接口的扩展接口，能够对路径（如包含通配符）进行解析，并对其进行加载。



ApplicationContext实现了ResourcePatternResolver接口，内部将接口功能代理给PathMatchingResourcePatternResolver完成。




# PathMatchingResourcePatternResolver 

PathMatchingResourcePatternResolver是Spring中内置的唯一一个ResourcePatternResolver接口实现类。

ResourceLoader接口部分的功能由ResourceLoader的实现类DefaultResourceLoader实现。
```java
private final ResourceLoader resourceLoader;
public PathMatchingResourcePatternResolver() {  
    this.resourceLoader = new DefaultResourceLoader();  
}
```

## 解析Ant风格路径

PathMatchingResourcePatternResolver内部利用org.springframework.util.AntPathMatcher工具类对Ant风格的路径进行解析。
```java
private PathMatcher pathMatcher = new AntPathMatcher();
```
以 `/WEB-INF/*-context.xml` 为例来介绍解析Ant风格路径的过程。

1）首先找到到最后一个不包含通配符的路径片段（`/WEB-INF/`），将它实例化为一个URL实例（通过`ClassLoader.getResource()`）。
2.1）如果URL不是一个 `jar:URL` ，则通过URL获得一个 `java.io.File`对象，然后通过遍历文件系统来解析通配符。
2.2）如果URL是一个 `jar:URL`，则通过URL获得一个 `java.net.JarURLConnection`对象 或 手动解析 `jar:URL`，然后通过遍历jar文件内容来解析通配符。




被ResourceArrayPropertyEditor使用，从而对bean中`Resource[]`类型的属性进行填充。
被ApplicationContext使用，用来对接口方法的代理。



