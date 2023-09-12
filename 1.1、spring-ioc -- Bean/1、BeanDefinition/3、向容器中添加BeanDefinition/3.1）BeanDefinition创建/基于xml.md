一般，通过<bean/>标签的class属性指定将要实例化对象的类型（class属性是BeanDefinition实例的Class属性）。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
       
       <bean>...</bean>
</beans>
```




# 一、通过构造函数创建

```java
<bean id="exampleBean" class="examples.ExampleBean"/>
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```
-- --
# 二、通过静态工厂方法创建

通过这种方法，class属性不再指定返回的对象的类型，而是静态工厂方法所属的类。举个例子：
```java
public class Clazz1 {
	private static Clazz1 clazz1 = new Clazz1();
	private Clazz1() {}
	public static Clazz1 createInstance() {
	   return clazz1;
	}
}
```
```xml
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
```
```java
public class Clazz2{}
public class Clazz3{
	public static Clazz2 createClazz2(){
		return new Clazz2();
	}
}
```
```xml
<bean id="clazz2" class="Clazz3" factory-method="createClazz2"/>
```
-- --
# 三、通过实例工厂方法创建

通过这种方法，不再需要class属性，而是替换成factory-bean属性。举个栗子：
```java
public class Clazz2{}
public class Clazz3{
	public Clazz2 createClazz2(){
		return new Clazz2();
	}
}
```
```xml
<bean id="clazz2" factory-bean="clazz3" factory-method="createClazz2"/>
<bean id="clazz3" class="Clazz3"/>
```


## depends-on

当两个Bean不存在依赖注入关系，但一个类的正常工作又依托于另一个类的初始化完成时，能够使用depends-on来定义这种加载顺序关系。

`depend- on` 属性既可以指定初始化-时间依赖，也可以指定相应的销毁-时间依赖(如果只有singleton bean )。定义与给定bean依赖关系的依赖bean在给定bean自身被破坏之前，首先被破坏。因此，依赖也可以控制停机顺序。
```java
public class Clazz3 {  
    public Clazz3() {  
        System.out.println("Clazz3...");  
    }  
}
public class Clazz4 {  
    public Clazz4() {  
        System.out.println("Clazz4...");  
    }  
}
```

当没有这种顺序要求时：
```xml
<bean id="clazz3" class="org.pojo.Clazz3"/>  
<bean id="clazz4" class="org.pojo.Clazz4"/>
```
```java
// 输出
Clazz3...
Clazz4...
```

定义这种顺序要求后：
```xml
<bean id="clazz3" class="org.pojo.Clazz3" depends-on="clazz4"/>  
<bean id="clazz4" class="org.pojo.Clazz4"/>
```
```java
// 输出
Clazz4...
Clazz3...
```

## Lazy-initialized Beans

默认情况下，ApplicationContext实现创建和配置所有单例bean作为初始化过程的一部分。一般来说，这种 ` pre-instantiation `是可取的，因为配置或周围环境中的错误是立即发现的，而不是几小时甚至几天后才发现。当不希望这种行为时，可以通过将bean定义标记为 `lazy-init` 来防止对单个bean进行 `pre-instantiation `。标记为 `lazy-init` 的bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时。
```java
public class Clazz5 {  
    public Clazz5(Clazz6 clazz6) {  
        System.out.println("Clazz5初始化...");  
    }  
}
public class Clazz6 {  
    public Clazz6() {  
        System.out.println("Clazz6初始化...");  
    }  
}
```
```xml
<bean id="clazz5" class="Clazz5"/>
<bean name="clazz6" class="Clazz6" lazy-init="true"/>
```
```java
// 输出
Clazz5初始化...
```

但是，当一个 `lazy-init` 的bean被另一个非 `lazy-init` 的singleton bean依赖时，ApplicationContext在启动时还是会创建`lazy-init` 的bean，因为它必须满足依赖关系（将 `lazy-init` 的bean注入到非 `lazy-init` 的的singleton bean中。

```java
public class Clazz5 {  
    private Clazz6 clazz6;  
    public Clazz5(Clazz6 clazz6) {  
        this.clazz6 = clazz6;  
        System.out.println("Clazz5初始化...");  
    }  
}
public class Clazz6 {  
    public Clazz6() {  
        System.out.println("Clazz6初始化...");  
    }  
}
```
```xml
<bean id="clazz5" class="org.pojo.Clazz5">  
    <constructor-arg name="clazz6" ref="clazz6"/>  
</bean>  
<bean id="clazz6" class="org.pojo.Clazz6" lazy-init="true"/>
```
```java
// 输出
Clazz6初始化...
Clazz5初始化...
```

如果要改变所有bean的默认 `lazy-init` 行为，可以通过设置<beans/>标签的 `default-lazy-init`。
```xml
<beans default-lazy-init="true">
	<!-- no beans will be pre-instantiated... -->
</beans>
```


3）scope属性


# 指定scope-proxy

