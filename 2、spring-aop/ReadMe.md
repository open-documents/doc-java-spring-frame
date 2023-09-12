
spring参考文档：
- https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/
- https://docs.spring.io/spring-framework/reference/
spring-aop参考文档：
- 基础 -- `https://docs.spring.io/spring-framework/reference/core/aop-api.html`
- 详尽 -- `https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/aop.html`


参考文档：https://docs.spring.io/spring-framework/reference/core/aop/introduction-defn.html

首先介绍AOP中的相关术语（注意，不是Spring AOP特有的术语）：

1）连接点（Join point）

连接点描述程序执行的某个特定位置。如一个类的初始化前、初始化后，或者类的某个方法调用前、调用后、方法抛出异常后等等。一个类或一段程序代码拥有一些具有边界性质的特定点，这些特定点就称为连接点。连接点用来定义在目标程序的哪里通过AOP加入新的逻辑。

>Spring AOP仅支持方法的连接点。即仅能在方法调用前、方法调用后、方法抛出异常时以及方法调用前后这些程序执行点织入增强。

连接点由两个信息确定：第一是用方法表示的程序执行点；第二是用相对点表示的方位。

例如 ：UserService#add() 方法执行之前，这个连接点。执行点为 UserService#add()方法本身； 方位为该方法执行前的位置。


2）切入点（Pointcut）

切入点是一个连接点的过滤条件，AOP 通过切点定位到特定的连接点。

每个类都拥有多个连接点：例如 UserService类中的所有方法实际上都是连接点，即连接点是程序类中客观存在的事物。类比：连接点相当于数据库中的记录，切入点相当于查询条件。切入点和连接点不是一对一的关系，一个切入点匹配多个连接点，切点通过 org.springframework.aop.Pointcut 接口进行描述，它使用类和方法作为连接点的查询条件。

xml配置方式：
```xml
<aop:config>
    <aop:pointcut id="myPointcut" expression="execution(* com.kaka.spring.aop.aspect.service.UserService.*(..))"/>
</aop:config>
```

3）通知（Advice）

切面在某个具体的连接点采取的行为或行动，称为通知。切面的核心逻辑代码都写在通知中。通知是切面功能的具体实现，通常是业务代码以外的需求，如日志、验证等，这些被模块化的特殊对象。

常用的通知接口有：
- 前置通知：org.springframework.aop.MethodBeforeAdvice
- 后置通知：org.springframework.aop.AfterReturningAdvice
- 异常通知：org.springframework.aop.ThrowsAdvice。该接口没有要实现的方法，需要自定义一个afterThrowing()方法。
- 环绕通知：org.aopalliance.intercept.MethodInterceptor

4）通知器（Advisor）

通知器由一个切入点（pointcut）和一个通知（Advice）组成。通知就是增强的那部分功能代码，如记录日志代码、控制权限代码。
```xml
<aop:advisor advice-ref="myMethodBeforeAdvice" pointcut-ref="myPointcut" />
```

5）切面（Aspect）

与通知器（advisor）类似，都是通知+切入点。区别在于，切面中的类无需实现通知接口，但需要在配置文件中指定类中的方法名，而通知器仅需指定类名即可，因为通知器中的类都实现了通知接口，很明确的知道通知方法是哪个。  
xml配置方式
```xml
<bean id="myXmlAspect" class="com.kaka.spring.aop.aspect.MyXmlAspect"/>
<aop:config>
    <!--①. 切入点(可以有多个) -->
    <aop:pointcut id="myPointcut" expression="execution(* com.kaka.spring.aop.aspect.service.UserService.*(..))"/>
    <!--②. 切面(可以有多个) -->
    <aop:aspect ref="myXmlAspect">
        <!--切点与切面中的方法关联 -->
        <aop:around method="aroundMethod" pointcut-ref="myPointcut"/>
    </aop:aspect>
</aop:config>
```



spring-aop支持xml风格和注解风格的AOP。

# xml风格aop

- 如果要使用xml风格的AOP，需要导入spring-aop schema。
- 所有切面、切点、通知都必须放在`<aop:config>`元素内。可以有多个`<aop:config>`元素
- `<aop:config>`元素内，必须按照切点、通知、切面的顺序依次声明。
- 不能混合使用`<aop:config>`风格和`AutoProxyCreator`风格。

# 准备工作

```java
interface UserDao {
    public void get();
    public void set();
}
public class UserDaoImpl implenments UserDao {
    public void get() {
        System.out.println("UserDao的get()方法...");
    }
    public void set(){
	    System.out.println("UserDao的set()方法...");
    }
}
```

# 需求：对类的所有方法进行增强

通过xml方式。

```xml
<bean id="userDao" class="UserDaoImpl"/> 
<bean id="beforeAdvice" class="UserDaoBeforeAdvice"/>

<bean id="userDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target" ref="userDao"/>
	<property name="proxyInterfaces" value="UserDao"/>
	<property name="interceptorNames" value="beforeAdvice"/>
</bean>
```

# 需求：对类的部分特定方法进行增强

主要通过定义切面进行实现。

xml配置文件
```xml
<bean id="userDao" class="UserDao"/>
<bean id="aroundAdvice" class="OrderDaoAroundAdvice"/>

<!--  切面Advisor  -->
<bean id="pointCutAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
	<property name="patterns" value="UserDao.add.*,UserDao.delete.*"/>
	<property name="advice" ref="aroundAdvice"/>
</bean>

<bean id="userDaoProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
	<property name="target" value="userDao"/>
	<property name="proxyTargetClass" value="true"/>
	<property name="interceptorNames" value="pointCutAdvisor"/>
</bean>
```

