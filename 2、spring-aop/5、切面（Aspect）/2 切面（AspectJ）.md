
# 声明一个切面


参考文档：
- `https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/aspectj-support.html`
- `https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/at-aspectj.html`

先说结论，@AspectJ有2个作用：
- 让spring组件变为


为了在spring配置类中使用 @AspectJ注解 的方式来声明一个切面，需要开启对于 @AspectJ的支持。有通过注解的方式 和 通过xml配置的方式 2种方式开启对@AspectJ的支持。

1）通过注解的方式开启对@AspectJ的支持需要 在Spring配置类上标注@EnableAspectJAutoProxy注解。
```java
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {}
```
2）通过xml配置的方式开启对@AspectJ的支持需要在xml配置文件中添加`<aop:aspectj-autoproxy>`标签。
```xml
<aop:aspectj-autoproxy>
```


开启对于@AspectJ的支持后，任何定义在ApplicationContext中并且标注了@AspectJ注解的bean都会被自动检测并被代理。

>需要注意的是，仅仅添加@AspectJ注解是不足以让spring完成自动检测并将其通过aop进行配置，spring完成自动检测是通过对于容器组件的扫描完成的，即需要将其注册为容器中的组件才可以。简而言之，@AspectJ必须添加在组件上才能生效。

下面是2个简单的例子。

1）通过xml方式定义一个标注了@AspectJ注解的bean。
```xml
<bean id="myAspect" class="com.xyz.NotVeryUsefulAspect">
	<!-- configure properties of the aspect here -->
</bean>
```
2）通过注解方式定义一个@AspectJ注解的bean。
```java
@Aspect 
@Component
public class NotVeryUsefulAspect {}
```


>注意，@AspectJ类不会成为来自其他切面中的通知的的target，标注@AspectJ后，会将其从自动代理中排除。


