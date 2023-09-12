	参考文档:P400

`org.springframework.aop.Pointcut`是核心接口。

PointCut分为static pointcut和dynamic pointcut。
- static pointcut只考虑方法和target class，而不考虑方法参数。Spring只在当方法被调用时计算static pointcut一次，后续都不再计算。
- dynamic pointcut：每次方法调用都计算一次。


# 定义

## PointCut
```java
public interface Pointcut {   
	ClassFilter getClassFilter();   
	MethodMatcher getMethodMatcher();
}
```
-- --
## ClassFilter

ClassFilter接口 将PointCut限制在一个target class集。
```java
public interface ClassFilter {   
	boolean matches(Class clazz);
}
```
-- --
## MethodMatcher

```java
public interface MethodMatcher {   
	boolean matches(Method m, Class<?> targetClass);  
	boolean isRuntime();   
	boolean matches(Method m, Class<?> targetClass, Object... args);
}
```
- `matches(Method m, Class<?> targetClass)`：判断PointCut是否匹配给定target class中的给定方法。
- `matches(Method m, Class<?> targetClass, Object... args)`：当两个参数的matches方法返回true并且isRuntime()返回true时，在每个方法调用时被调用。这使得PointCut可以在目标通知开始之前查看传递给方法调用的参数。

大多数MethodMatcher实现类是静态的，意味着isRuntime()返回false，进而意味着三个参数的matches方法不会被调用。


# 操作PointCut

对PointCut操作主要是union和intersection。

union：匹配任意一个。
intersection：需要同时匹配。

使用`org.springframework.aop.support.Pointcuts`或`org.springframework.aop.support.ComposablePointcut`的静态方法来操作PointCut。

# 内置实现PointCut

## AspectJExpressionPointcut

从Spring2.0开始，AspectJExpressionPointcut是最重要的实现类。它使用AspecJ库提供的功能来解析AspectJ风格的切点表达式。

## JdkRegexpMethodPointcut

	参考文档:P402~P403

JdkRegexpMethodPointcut是static pointcut。使用union规则。通过正则表达式来匹配符合的目标。

```xml
<bean id="settersAndAbsquatulatePointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut"> 
	<property name="patterns">
		<list>
			<value>.*set.*</value>
			<value>.*absquatulate</value>
		</list> 
	</property>
</bean>
```
Spring提供一个方便的RegexpMethodPointcutAdvisor。内部使用JdkRegexpMethodPointcut。
```xml
<bean id="settersAndAbsquatulateAdvisor"   class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
	<property name="advice">
		<ref bean="beanNameOfAopAllianceInterceptor"/>
	</property>
	<property name="patterns"> 
		<list> 
			<value>.*set.*</value> 
			<value>.*absquatulate</value> 
		</list>
	</property>
</bean>
```

## ControlFlowPointcut

# 内置帮助子类

## StaticMethodMatcherPointcut

```java
class TestStaticPointcut extends StaticMethodMatcherPointcut { 
	public boolean matches(Method m, Class targetClass) { 
		// return true if custom criteria match 
	}
}
```