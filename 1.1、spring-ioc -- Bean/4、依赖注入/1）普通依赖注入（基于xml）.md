依赖注入( Dependency Injection，DI )是对象仅通过构造函数参数、工厂方法参数或对象实例构造或从工厂方法返回后在对象实例上设置的属性来定义其依赖关系(也就是说,他们所工作的其他对象)的过程。然后容器在创建bean时注入这些依赖关系。这个过程从根本上说是bean本身的反向(因此得名,控制反转)，通过直接构造类或服务定位器模式来控制它依赖自身的实例化或定位。

DI存在于两种主要的变体中：`基于构造函数的依赖注入` 和 `基于Setter的依赖注入`。

# 一、向构造函数注入参数依赖

## 1）通过参数类型

如果bean definition中的构造函数参数不存在歧义，那么Bean Definition中构造函数参数的定义顺序就是bean实例化时这些参数被提供给相应构造函数的顺序。举个栗子：
```java
public class Clazz1 {
	private Clazz2 clazz2;
	private boolean bool;
	// 构造函数
	public Clazz1(Clazz2 clazz2, boolean bool) {
	   // ...  
	}
}
```

```xml
<beans>
	<bean id="clazz1" class="Clazz1">
		<constructor-arg ref="clazz2"/>
		<!--按官方示例，必须显示指定primitive类型的type-->
		<constructor-arg type="boolean" value="true"/>
		<!--idea中不用显示指定也可以-->
		<constructor-arg value="true"/>
	</bean>
	<bean id="clazz2" class="Clazz2"/>
</beans>
```

注入引用类型时，使用<ref/>标签（如注入clazz2属性）。
注入primitive类型时，使用<value/>标签（如注入bool属性）。

### 基于注解


## 2）通过参数索引

可以使用index属性显式地指定构造函数参数的索引。

除了解决多个简单值的歧义外，指定一个索引可以解决一个构造函数有两个相同类型参数的歧义。

### 基于XML（<constructor-arg/>的index属性）

```xml
<bean id="clazz1" class="Clazz1">
	<constructor-arg index="0" ref="clazz2"/>
	<constructor-arg index="1" value="true"/>
</bean>
<bean id="clazz2" class="Clazz2"/>
```

## 3）通过参数名称

这种方式必须在编译时指定能够通过反射获取到函数形参名称。

### 基于XML
```xml
<bean id="exampleBean" class="examples.ExampleBean">
	<constructor-arg name="years" value="7500000"/>
	<constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

如果编译后不保存形参名称，那么可以用 `java.beans.@ConstructorProperties注解` 代替。

```java
public class ExampleBean {
	// Fields omitted
	@ConstructorProperties({"years", "ultimateAnswer"})
	public ExampleBean(int years, String ultimateAnswer) {
	   this.years = years;
	   this.ultimateAnswer = ultimateAnswer;
	}
}
```

## 4）使用c（constructor）命名空间
```xml
<!-- c-namespace index declaration --> 
<bean id="beanOne" class="x.y.ThingOne" c:_0-ref="beanTwo" c:_1-ref="beanThree"   c:_2="something@somewhere.com"/>
```

由于XML语法的原因，索引表示法要求前导符_的存在，因为XML属性名不能以数字(即使有些IDE允许)开头。相应的索引表示法也可用于元素，但不常用，因为简单的声明顺序通常是足够的。

# 二、向Setter函数注入参数依赖

基于Setter的DI是通过调用无参数构造函数或无参数静态工厂方法实例化bean后，容器调用bean上的setter方法来完成的。
```java
public class Clazz1 {
	private Clazz2 clazz2;
	public void setClazz2(Clazz2 clazz2) {
	   this.clazz2 = clazz2;
	}
}
public class Clazz2{}
```
## 1）基于XML
```xml
<bean id="clazz1" class="Clazz1">
	<property name="clazz2" ref="clazz2"/>
</bean>
<bean id="clazz2" class="Clazz2"/>
```

## 2）基于p(property)命名空间

1）设置primitive类型值，可以用 `<bean>` 标签的 `p:属性名称=""` 代替 `<property name="" value=""/> `

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroymethod="close">
	<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
	<property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
	<property name="username" value="root"/>
	<property name="password" value="misterkaoli"/>
</bean>
```

等价于：

```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroymethod="close"
	  p:driverClassName="com.mysql.jdbc.Driver"
	  p:url="jdbc:mysql://localhost:3306/mydb"
	  p:username="root"
	  p:password="misterkaoli"/>
```
2）设置引用类型
```xml
<bean name="john-modern"   class="com.example.Person"   p:name="John Doe"   p:spouse-ref="jane"/>
```



# 三、向（静态）工厂方法注入参数依赖

## 向静态工厂方法注入参数依赖

```java

```
## 向实例工厂方法注入参数依赖



# 四、向普通方法参数进行注入（116~120）

## 1.1）@Value注解的普通类型值
```java

```
## 1.2）@Value注解的${}类型值
@Value注解一般用于从外部注入属性。举个栗子：
```java
@Component
public class MovieRecommender {   
	private final String catalog;   
	public MovieRecommender(@Value("${catalog.name}") String catalog) {   
		this.catalog = catalog;
	}
}
```
配置类：
```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```
配置文件：
```properties
# file: application.properties
catalog.name=MovieCatalog
```
## 1.3）@Value注解的默认值
```java
@Component public class MovieRecommender {   private final String catalog;   public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {   this.catalog = catalog;  } }
```
## 1.4）@Value注解的SpEL表达式
如果@Value包含SpEL表达式，那么这个值将会在运行时被动态计算。举个栗子：
```java
@Component public class MovieRecommender {   private final String catalog;   public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {   this.catalog = catalog;  } }
@Component public class MovieRecommender {   private final Map<String, Integer> countOfMoviesPerCatalog;   public MovieRecommender(   @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {   this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;  } }
```

## 2）理论知识

对于@Value注解的value中此类型值的解析由Spring内部提供。内置解析器默认尝试解析`${}` 中的内容，如果无法解析，默认情况，就会直接将${}中的内容进行注入。
修改解析器的行为，需要配置PropertySourcesPlaceholderConfigurer bean。Spring boot对于PropertySourcesPlaceholderConfigurer的配置来自application.properties和application.yml文件。
在将占位符解析成具体的String类型值后，将@Value中的String类型值转换成目标类型是通过BeanPostProcessor中的ConversionService实现的。


# 、属性或构造参数的值类型


# merge
不能merge不同类型的集合。
merge写在父元素中不起作用。


# 、嵌套属性名称

```xml
<bean id="something" class="things.ThingOne">   <property name="fred.bob.sammy" value="123" /> </bean>
```

# 、<parent/>标签
## 1）类父子关系

若类B继承于类A（B extends A），

## 2）容器父子关系

```xml
<!-- in the parent context --> 
<bean id="accountService" class="com.something.SimpleAccountService">
	<!-- insert dependencies as required here -->
</bean>
<!-- in the child (descendant) context --> 
<!-- bean name is the same as the parent bean -->   
<bean id="accountService" class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target">
		<ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
	</property>
	<!-- insert other configuration and dependencies as required here -->
</bean>
```
## 3）BeanDefinition层级关系

