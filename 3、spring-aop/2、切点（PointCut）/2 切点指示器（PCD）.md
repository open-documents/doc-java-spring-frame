
参考文档：https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/pointcuts.html#aop-pointcuts-designators


## execution（使用最普遍）

execution切点指示器的格式如下：
```text
execution(
	modifiers-pattern? 
	ret-type-pattern 
	declaring-type-pattern?namepattern(param-pattern)   
	throws-pattern?
)
```
除了 `ret-type-pattern`，其他片段都是可选的。
- `ret-type-pattern`：匹配返回类型。`*` 代表返回任何类型，是最常用的。
- `namepattern`：匹配方法名称。能够使用 `*` 作为全部或部分方法名称模式串。
- declaring-type-pattern：使用 `.` 去加入到namepattern中。
- param-pattern：`()`代表无参数；`..`代表任意数量参数（零个或以上）；`*` 代表任意类型的一个参数。

```java
// 任意public方法
execution(public * *(..))
// 任何以set开头的名称的方法
execution(* set*(..))
// 任意AccountService接口中定义的方法
execution(* com.xyz.service.AccountService.*(..))
// service包中的方法
execution(* com.xyz.service.*.*(..))
// service包或其子包中的方法
execution(* com.xyz.service..*.*(..))
```


## within

用于匹配指定类型内的方法执行。

```java
// Any join point (method execution only in Spring AOP) within the service package
within(com.xyz.service.*)
// Any join point (method execution only in Spring AOP) within the service package or one of its sub-packages
within(com.xyz.service..*)
```







## bean

bean PCD用于匹配特定名称的Bean对象的执行方法。

bean PCD是SpringAOP特有的，不被原生的AspectJ weaving支持，因此不能在@AspectJ中声明。

```java
// Any join point (method execution only in Spring AOP) on a Spring bean named tradeService
bean(tradeService)
// Any join point (method execution only in Spring AOP) on Spring beans having names that match the wildcard expression *Service
bean(*Service)
```



## @within

用于匹配所有持有指定注解类型内的方法

```java
// Any join point (method execution only in Spring AOP) where the declared type of the target object has an @Transactional annotation
@within(org.springframework.transaction.annotation.Transactional)
```

## this

用于匹配当前AOP代理对象类型的执行方法；注意是AOP代理对象的类型匹配，这样就可能包括引入接口也类型匹配；

```java
// Any join point (method execution only in Spring AOP) within the service package or one of its sub-packages
this(com.xyz.service.AccountService)
```

## target

用于匹配当前目标对象类型的执行方法；注意是目标对象的类型匹配，这样就不包括引入接口也类型匹配。

```java
// Any join point (method execution only in Spring AOP) where the target object implements the AccountService interface
target(com.xyz.service.AccountService)
```

## @target

用于匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解。
```java
// Any join point (method execution only in Spring AOP) where the target object has a @Transactional annotation
@target(org.springframework.transaction.annotation.Transactional)
```


## args

用于匹配当前执行的方法传入的参数为指定类型的执行方法

```java
// Any join point (method execution only in Spring AOP) that takes a single parameter and where the argument passed at runtime is Serializable
args(java.io.Serializable)
```

## @args

用于匹配当前执行的方法传入的参数持有指定注解的执行。
```java
// Any join point (method execution only in Spring AOP) which takes a single parameter, and where the runtime type of the argument passed has the @Classified annotation
@args(com.xyz.security.Classified)
```

## @annotation

用于匹配当前执行方法持有指定注解的方法。
```java
// Any join point (method execution only in Spring AOP) where the executing method has an @Transactional annotation
@annotation(org.springframework.transaction.annotation.Transactional)
```

# 组合切点表达式

参考文档：https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/pointcuts.html#aop-pointcuts-combining




xml中：与（and或&amp;&amp;）、或（||）、非（  ！）。






注解中：&& ，|| ，！来合并切点表达式。

```java
@Pointcut("execution(public * *(..))") 
private void anyPublicOperation() {} 
@Pointcut("within(com.xyz.myapp.trading..*)") 
private void inTrading() {} 
@Pointcut("anyPublicOperation() && inTrading()") 
private void tradingOperation() {}
```



- `*`：匹配任何数量字符。
- `..`：匹配任何数量字符的重复。如在类型模式中匹配任何数量子包；而在方法参数模式中匹配任何数量参数。
- `+`：匹配指定类型的子类型。仅能作为后缀放在类型模式后边。

`java.lang.String`：匹配String类型。

`java.*.String`：匹配java包下的任何“一级子包”下的String类型。 如匹配java.lang.String，但不匹配java.lang.ss.String。

`java..*`：匹配java包及任何子包下的任何类型。如匹配java.lang.String、java.lang.annotation.Annotation。

`java.lang.*ing`：匹配任何java.lang包下的以ing结尾的类型。

`java.lang.Number+`：匹配java.lang包下的任何Number的子类型。如匹配java.lang.Integer，也匹配java.math.BigInteger。