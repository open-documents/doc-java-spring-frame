
参考文档：https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/pointcuts.html

# 通过接口声明一个切点（Pointcut、ComposablePointcut）

下面是 PointCut接口 的定义。
```java
public interface Pointcut {
	ClassFilter getClassFilter();
	MethodMatcher getMethodMatcher();
}
```
通过 <font color=44cf57>ClassFilter 和 MethodMatcher 共同确定切入点</font>。

1） <font color="FFDD00">ClassFilter接口确定感兴趣的类。</font>下面是 ClassFilter接口 的定义。
```java
public interface ClassFilter {
	boolean matches(Class clazz);
}
```
2）<font color="FFDD00">MethodMatcher接口确定感兴趣的方法。</font>下面是 MethodMatcher接口 的定义。
```java
public interface MethodMatcher {
	boolean matches(Method m, Class<?> targetClass);
	boolean isRuntime();
	boolean matches(Method m, Class<?> targetClass, Object... args);
}
```





一个切点的声明包含2部分：
- 由名称和任意数量参数组成的签名。
- 切点表达式。

# 通过注解声明一个切点（@PointCut）

下面是 @PointCut注解 的定义。

下面来


在方法上标注@PointCut注解来声明一个切点。
```java
@Pointcut("execution(* transfer(..))") // the pointcut expression 
private void anyOldTransfer() {} // the pointcut signature
```

```xml
<aop:config>   
	<aop:aspect id="myAspect" ref="aBean">   
		<aop:pointcut id="businessService" expression="execution(* com.xyz.myapp.service.*.*(..))"/>
		...   
	</aop:aspect>
</aop:config>
```

xml中引用注解定义的切点
```xml
<aop:config>   
	<aop:pointcut id="businessService" expression="com.xyz.myapp.CommonPointcuts.businessService()"/> </aop:config>
```

# 切点类型

## 静态切点

静态切点是通过抽象类 `org.springframework.aop.support.StaticMethodMatcher` 的子类提供的，主要有两个子类：
- JdkRegexpMethodPointcut
- NameMatchMethodPointcut

静态切点的优点在于它们可以在代理对象创建时确定，因此性能较高。然而，由于它们是在代理对象创建时确定的，所以无法在运行时改变。


1）JdkRegexpMethodPointcut

通过正则表达式匹配方法签名来定义切点。可以通过正则表达式拦截具有特定签名的所有方法。例如，如果你希望拦截所有以"get"开头，以"Info"结尾的方法，可以使用JdkRegexpMethodPointcut设置为`get.*Info(..)`。
`org.springframework.aop.support.JdkRegexpMethodPointcut` 通过java.util.regex中的正则来匹配 patterns属性（String类型数组），从而确定切点。
```java
/* -------------------------- AbstractRegexpMethodPointcut --------------------------
 * Description: 该类是JdkRegexpMethodPointcut的父类
 */
private String[] patterns = new String[0];

public void setPattern(String pattern){  setPatterns(pattern);  }
public void setPatterns(String... patterns){  ...  }
```

2）NameMatchMethodPointcut

通过提供简单字符串匹配方法签名来定义切点。例如，如果你希望拦截所有名为"add"的方法，你可以将NameMatchMethodPointcut设置为add(..)。




## 动态切点

动态切点是在运行时根据参数决定是否满足切点条件。它们通常使用MethodMatcher实现类来定义切点。MethodMatcher TRUE = TrueMethodMatcher.INSTANCE;这个类提供了matches方法，该方法在运行时根据目标类和方法参数决定是否应该通知（织入）。

动态切点的优点在于它们可以根据运行时的参数动态决定是否通知，因此更加灵活。然而，由于它们需要在运行时进行判断，所以性能低于静态切点。



AspectJExpressionPointcut

从2.0开始，内置的<font color=44cf57>最重要的实现类</font> `org.springframework.aop.aspectj.AspectJExpressionPointcut` ，通过AspectJ提供的库来解析切点表达式字符串来确定切点。




