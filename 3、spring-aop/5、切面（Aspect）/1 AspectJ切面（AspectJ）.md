
# 声明一个切面


参考文档：
- `https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/aspectj-support.html`
- `https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/at-aspectj.html`


开启对于@AspectJ的支持后（如何开启见2），任何定义在ApplicationContext中的bean（各种方式均可，只要让bean的BeanDefinition添加到容器中）同时标注了@AspectJ注解都会被自动检测并被解析以据此创建代理。

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


