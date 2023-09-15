
通过@AspectJ注解声明一个切面后，需要使用 AnnotationAwareAspectJAutoProxyCreator 来解析@AspectJ类（或者说切面）来创建对应的代理。

有2种方式来导入这个类：xml方式、注解方式，但是不管哪种方式都必须确保AspectJ的aspectjweaver.jar在类路径下。

1）xml方式
```xml
<aop:aspectj-autoproxy/>
```
2）注解方式（使用@EnableAspectJAutoProxy ）
```java
@Configuration 
@EnableAspectJAutoProxy 
public class AppConfig { }
```