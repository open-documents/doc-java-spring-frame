
参考文档：P180~P181

这个功能实现主要基于@Conditional注解、Condition接口实现。

# @Conditional

@Conditional指明只有当component满足所有的Condition接口中的match方法才会将该component注册。

<font color=44cf57>如果一个类@Configuration类被@Conditional标注，所有的@Bean方法、@Import注解、@ComponentScan注解都将受到@Conditional注解的控制。</font>

## 1）定义

@Conditional注解定义如下：
```java
@Target({ElementType.TYPE, ElementType.METHOD})  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
public @interface Conditional {   
	/**  
    * All {@link Condition} classes that must {@linkplain Condition#matches match}  
    * in order for the component to be registered.
    * */   
    Class<? extends Condition>[] value();   
}
```
-- --
## 2）使用方式

@Conditional注解标注于：
- 1）直接或间接标注了@Component注解的类上。
- 2）@Bean方法上。
- 3）其他注解上，用于作为元注解。

## 3）注意

@Conditional注解不支持被继承，所有来自父类或来自被重写方法的@Conditional不会生效，为了强制实现这个语义，@Conditional注解自身不会被@Inherited注解标注，另外任何自定义的被@Conditional标注的合成注解也禁止标注@Inherited注解。

