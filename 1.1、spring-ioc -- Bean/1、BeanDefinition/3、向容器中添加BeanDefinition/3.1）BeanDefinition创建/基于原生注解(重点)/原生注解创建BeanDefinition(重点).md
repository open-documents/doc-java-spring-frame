Spring提供@Component、@Configuration、@Repository、@Service、@Controller 和 包扫描来共同实现通过注解实例化bean的功能。

其中@Component是泛化性最强的，@Configuration、@Repository、@Service、@Controller 是@Component用于不同用途类的特殊化，即这四类注解也是@Component，只是标注的类的用途不一样而已。
@Configuration表示配置类，@Repository用于持久层，@Service用于服务层，@Controller用于控制层。

@Repository、@Service、@Controller更容易使用工具处理，也更容易与切面联系起来。这三类注解是理想的切面target。

@Repository、@Service、@Controller会在将来的Spring版本有更多的语义。

# 一、理论知识

1）所有@Configuration类默认情况下会通过CGLIB进行代理，即所有非final、非static、非private和包权限的方法语义会被增强，更加具体地讲，对于被@Bean注解标注的方法，子类（CGLIB产生的类）中的方法会首先检查容器中缓存的scope的bean，存在缓存的话，则是直接从容器中取，没有缓存才会调用父类方法创建新实例。（P168）

There are a few restrictions due to the fact that CGLIB dynamically adds features at startup-time. In particular, configuration classes must not be final. However, as of 4.3, any constructors are allowed on configuration classes, including the use of @Autowired or a single non-default constructor declaration for default injection. 
If you prefer to avoid any CGLIB-imposed limitations, consider declaring your @Bean methods on non-@Configuration classes (for example, on plain @Component classes instead). Cross-method calls between @Bean methods are not then intercepted, so you have to exclusively rely on dependency injection at the constructor or method level there.


# 二、所有注解的定义

## 1）@Component注解
@Configuration、@Service、@Controller注解都是被@Component注解标注的注解，使得能以处理@Conponent注解的方式得到处理。下面是@Component注解的定义：
```java

```
## 2）@Configuration注解
```java
@Target({ElementType.TYPE})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Component  
public @interface Configuration {}
```
## 3）@Service
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {   // ... }
```
## 4）@Controller
```java

```



# 五、指定组件名称


当一个组件被扫描自动检测到时，它的名称由 scanner中的 `BeanNameGenerator` 生成，默认名称是类的名称（首字母小写）。
举个栗子：（下面栗子中，实例名称为movieFinderImpl）。
```java
@Repository 
public class MovieFinderImpl implements MovieFinder {}
```

通过原生注解的 `value属性` 显示地指定实例名称。
举个栗子：（下面栗子中，实例名称为myMovieLister）。
```java
@Service("myMovieLister") 
public class SimpleMovieLister {}
```

## 自定义名称生成策略

通过实现BeanNameGenerator接口（保证有一个无参构造）。

默认的名称生成策略可能会出现名称冲突的情况（虽然不在同一个包中）。

到了Spring5.2.3，FullyQualifiedAnnotationBeanNameGenerator 提供的名称生成策略是使用类的全类名。可以用来处理默认名称生成策略带来的问题。

## 使用自定义名称策略

```java
@Configuration 
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {}
```
```xml
<context:component-scan base-package="org.example" name-generator="org.example.MyNameGenerator" />
```

# 六、指定组件作用范围（@Scope）

默认情况，组件的作用范围是Singleton，如果需要改变组件的作用范围，通过@Scope注解进行指定。
举个栗子：
```java
@Scope("prototype")
@Repository 
public class MovieFinderImpl implements MovieFinder {}
```

## 自定义作用范围解析策略

如果希望提供一个自定义的范围解析策略，而不是依赖注解的方法，实现 `ScopeMetadataResolver`接口（必须有一个无参构造）。

## 使用自定义作用范围解析策略

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class) 
public class AppConfig {}
```
```xml
<beans>
	<context:component-scan base-package="org.example" scoperesolver="org.example.MyScopeResolver"/> </beans>
```

# 指定组件Qualifier

与大多数基于注释的替代方案一样，注释元数据与类定义本身绑定在一起，而XML的使用允许同一类型的多个bean提供其限定符元数据的变化，因为元数据是按实例而不是按类提供的。

栗子一：
```java
@Component @Qualifier("Action") public class ActionMovieCatalog implements MovieCatalog {   // ... }
```
栗子二：
```java
@Component @Genre("Action") public class ActionMovieCatalog implements MovieCatalog {   // ... }
```
栗子三：
```java
@Component @Offline public class CachingMovieCatalog implements MovieCatalog {   // ... }
```

# 七、针对prototype的代理

对于prototype的作用范围的对象，往往需要对其进行代理，下面是两种对prototype作用范围对象代理的指定方式：
```java
@Configuration @ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES) public class AppConfig {   // ... }
```
```xml
<beans>   <context:component-scan base-package="org.example" scoped-proxy="interfaces"/> </beans>
```

# 八、包扫描性能优化

在应用中有大量使用`@ComponentScan`扫描的package包含的类越多的时候，Spring注解解析耗时就越长。

## 使用方法

在项目中使用的时候需要导入一个`spring-context-indexer` jar包，有Maven和Gradle 两种导入方式，以maven为栗，引入jar配置如下：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.1.12.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```
然后在代码中，对于使用了模式注解的类上加上`@Indexed`注解即可。如下：
```java
@Indexed
@Controller
public class HelloController {}
```

## 原理
在项目中使用了`@Indexed`之后，编译打包的时候会在项目中自动生成`META-INT/spring.components`文件。  
当Spring应用上下文执行`ComponentScan`扫描时，`META-INT/spring.components`将会被`CandidateComponentsIndexLoader` 读取并加载，转换为`CandidateComponentsIndex`对象，这样的话`@ComponentScan`不在扫描指定的package，而是读取`CandidateComponentsIndex`对象，从而达到提升性能的目的。

## 注意事项

1）在IDE中使用这种模式时，必须将spring-context- indexer注册为注解解析器，以确保在更新候选组件时索引是最新的。
2）假设Spring应用中存在一个包含`META-INT/spring.components`资源的a.jar，b.jar没有使用这种方式，如果在整个项目中使用这种方式，那么使用`@ComponentScan`扫描这两个JAR中的package时，b.jar 中的bean实例就不能被获取。

# 、@Configuration类

@Configuration类中的非静态@Bean方法会被代理，静态@Bean方法不会被代理（因为CGLIB的特性）。因此，

- @Configuration类中的非静态@Bean方法不能是private或final。
- 必须将返回BeanFactoryPostProcessor或BeanPostProcessor的@Bean方法声明为静态，这样当这两种类型的类在容器启动早期被初始化时不会触发容器其他部分。



