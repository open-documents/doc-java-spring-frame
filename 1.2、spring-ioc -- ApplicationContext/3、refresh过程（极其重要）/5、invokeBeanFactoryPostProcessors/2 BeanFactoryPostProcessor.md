
BeanFactoryPostProcessor与BeanPostProcessor有点类似，不同的是，BeanFactoryPostProcessor处理的是Configuration metadata。

也就是容器使用BeanFactoryPostProcessor读取configuration metadata，并在容器实例化非BeanFactoryPostProcessor实例bean之前对与这些bean相关的bean definition（也就是这个bean的配置）进行修改。

# 注意事项

## 1）lazy-init

因为将一个bean设置成lazy-init后，如果没有bean引用它，那么这个设置成lazy-init的bean将永远不会得到初始化，因此BeanFactoryPostProcessor的lazy-init属性将会被忽略，即使设置了<beans/>标签的default-lazy-init属性为true，BeanFactoryPostProcessor还是会被早早地初始化。举个栗子：

使用到的类：
```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {  
    public MyBeanFactoryPostProcessor() {  
        System.out.println("MyBeanFactoryPostProcessor()...");  
    }  
    @Override  
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {}  
}
```
```xml
<bean class="example.MyBeanFactoryPostProcessor" lazy-init="true"/>
```
测试：
```java
public class MyTest {  
    @Test  
    public void test(){  
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("application.xml");  
    }  
}
```
输出：
```java
// MyBeanFactoryPostProcessor()...
```
-- --
## 2）BeanFactoryPostProcessor内部使用bean的问题

虽然可以在BeanFactoryPostProcessor内部使用bean实例，例如通过BeanFactory.getBean()获取后使用得到的bean实例，但是这会导致通过这种途径得到的bean实例被提前初始化，这会违背了标准的容器生命周期，从而导致一些问题，比如这些被提前初始化的bean不会被BeanPostProcessor处理。举个栗子：

使用到的类：
```java
// package example;
public class Clazz1 {  
    public Clazz1() {}  
}
public class Clazz2 {  
    public Clazz2() {}  
}
// BeanPostProcessor
public class MyBeanPostProcessor implements BeanPostProcessor {  
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {  
        System.out.println("BeanPostProcessor.postProcessBeforeInitialization().....");  
        return bean;  
    }  
   public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
       System.out.println("BeanPostProcessor.postProcessAfterInitialization().....");  
        return bean;  
    }  
}
// BeanFactoryPostProcessor
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {  
    @Override  
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {  
        Clazz1 clazz1 = beanFactory.getBean("clazz1", Clazz1.class);  
    }  
}
```
```xml
<bean id="clazz1" class="example.Clazz1FactoryBean"/>  
<bean id="clazz2" class="example.Clazz2"/>  <!--1-->
<bean class="example.MyBeanPostProcessor"/>  
<bean class="example.MyBeanFactoryPostProcessor"/>
```
测试：
```java
public class MyTest {  
    @Test  
    public void test(){  
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("application.xml");  
        Clazz1 clazz1 = applicationContext.getBean("clazz1", Clazz1.class);  // 2
        Clazz2 clazz2 = applicationContext.getBean("clazz2", Clazz2.class);  // 3
    }   
}
```
当1、、2、3被注释时，没有任何输出。
当1、3被注释时，没有任何输出。
没有代码被注释时，有如下输出，这是由于"clazz2"的初始化带来的，说明"clazz1" bean没有被BeanPostProcessor处理。
```java
// BeanPostProcessor.postProcessBeforeInitialization().....
// BeanPostProcessor.postProcessAfterInitialization().....
```




# 注册BeanFactoryPostProcessor

# 自动注册

ApplicationContext自动注册在configuration metadata中定义了的BeanFactoryPostProcessor。















