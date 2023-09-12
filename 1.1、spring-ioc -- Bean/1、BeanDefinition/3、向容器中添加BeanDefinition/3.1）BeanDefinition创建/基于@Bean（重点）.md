在官方文档中，称被@Bean标注的方法为工厂方法。

在被@Configuration注解标注的类、@Component注解标注的类、普通类内部，在方法上标注@Bean注解，以表示其为工厂方法。前者和后两者有所不同：
在被@Configuration注解标注的类内部使用@Bean方法，默认是full模式

# 三、@Bean在@Configuration类中与@Bean在@Component类或其他普通类中的区别

在@Component标注的类中的@Bean工厂方法与在@Configuration标注的类中的@Bean工厂方法被处理的方式略有不同：

@Configuration标注的类中，@Bean工厂方法中调用其他@Bean方法，不会按普通的java语义来调用，而是会创建对使用到的对象的bean元数据引用，通过这样的方式，经过了容器，这样能够提供声明周期管理和对bean的代理。
@Component标注的类中，因为被@Component标注的类没有被CGLIB增强，所以在@Bean中调用方法或field，会是普通java语义。

```java
@Component 
public class FactoryMethodComponent {   
	@Bean   
	@Qualifier("public")   
	public TestBean publicInstance() {   
		return new TestBean("publicInstance");  
	}   
	public void doWork() {   
		// Component method implementation omitted 
	}
}
```

# 、@Configuration中的@Bean方法的内部依赖

当一个bean依赖于另外一个bean时，表示这种依赖关系就像有一个bean方法调用另一个bean方法一样简单，这种方式只能用于声明在@Configuration的类中，而不能用于声明在@Component的类中。如下例所示：
```java
@Configuration 
public class AppConfig {   
	@Bean   
	public BeanOne beanOne() {
		return new BeanOne(beanTwo());
	}
	@Bean   
	public BeanTwo beanTwo() {
		return new BeanTwo();
	}
}
```

# 、指定作用范围（@Scope）(62)



## 基本使用

<font color=00FF00>通过@Bean注解定义的bean同样支持scope属性。</font>默认是singleton，可以指定任何标准的scope。（P160）
```java
@Configuration 
public class MyConfiguration {
	@Bean   
	@Scope("prototype")   
	public Clazz1 clazz1ByConfiguration() {}
}
```





# 、指定生命周期
## 指定initMethod

```java
public class Clazz1{  
    public Clazz1() {}  
  
    public void init(){  
        System.out.println("Clazz1.init()...");  
    }  
}
```
```java
@Configuration  
public class MyConfiguration {  
    @Bean(initMethod = "init") 
    public Clazz1 clazz1_MyConfiguration(){  
        return new Clazz1();  
    }
}    
```

## @Bean之间的相互依赖



## 指定destroyMethod

默认，通过@Bean产生的bean自动有名称为close或shutdown的destory callback。要改变，通过@Bean的destoryMethod属性指定。
```java
public class Clazz1{  
    public Clazz1() {}  
  
    public void destroy(){  
        System.out.println("Clazz1.destroy()...");  
    }  
}
```
```java
@Configuration  
public class MyConfiguration {  
    @Bean(destroyMethod = "destroy")  
    public Clazz1 clazz1_MyConfiguration(){  
        return new Clazz1();  
    }
}    
```




# 补充

## 1）关于@Bean返回类型

@Bean方法返回类型可以是接口类型，但是这会使得
-- --
## 2）关于返回类型是BeanPostProcessor和BeanFactoryPostProcessor

@Bean方法返回类型是BeanPostProcessor和BeanFactoryPostProcessor时，必须将@Bean方法声明为static，这样才不会触发包含他们的@Configuraion类的初始化，否则的话，可能会导致@Configuration类作为bean被初始化比AutowiredAnnotationBeanPostProcessor类实例化早，进而导致@Configuration类中的@Autowire和@Value注解不生效。（P174）

