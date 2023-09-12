# 环绕通知

	参考文档:P405

```java
public interface MethodInterceptor extends Interceptor { 
	Object invoke(MethodInvocation invocation) throws Throwable;
}
```

```java
public class DebugInterceptor implements MethodInterceptor { 
	public Object invoke(MethodInvocation invocation) throws Throwable { 
		System.out.println("Before: invocation=[" + invocation + "]"); 
		Object rval = invocation.proceed(); 
		System.out.println("Invocation returned");
		return rval; 
	}
}
```

# Before通知

	参考文档:P407~P408

```java
public interface MethodBeforeAdvice extends BeforeAdvice {
	void before(Method m, Object[] args, Object target) throws Throwable;
}
```

```java
public class CountingBeforeAdvice implements MethodBeforeAdvice { 
	private int count; 
	public void before(Method m, Object[] args, Object target) throws Throwable {
		++count; 
	} 
	public int getCount() { 
		return count;
	}
}
```

# 抛出异常通知

Spring支持抛出各种类型异常，这意味着`org.springframework.aop.ThrowsAdvice`是标记接口，不包含任何方法。

格式：
`afterThrowing([Method, args, target], subclassOfThrowable)`

例子：
```java
public class RemoteThrowsAdvice implements ThrowsAdvice {
	public void afterThrowing(RemoteException ex) throws Throwable { 
		// Do something with remote exception
	}
}
```
```java
public class ServletThrowsAdviceWithArguments implements ThrowsAdvice {
	public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) {
		// Do something with all arguments 
	}
}
```

```java
public static class CombinedThrowsAdvice implements ThrowsAdvice {
	public void afterThrowing(RemoteException ex) throws Throwable { 
		// Do something with remote exception 
	} 
	public void afterThrowing(Method m, Object[] args, Object target, ServletException ex) { 
		// Do something with all arguments 
	}
}
```

# 返回结果通知

	参考文档:P410

定义返回结果通知必须实现`org.springframework.aop.AfterReturningAdvice接口`。

```java
public interface AfterReturningAdvice extends Advice {
	void afterReturning(Object returnValue, Method m, Object[] args, Object target) throws Throwable;
}
```
例子：
```java
public class CountingAfterReturningAdvice implements AfterReturningAdvice {   
	private int count;   
	public void afterReturning(Object returnValue, Method m, Object[] args, Object target) throws Throwable {  
		++count;
	}  
	public int getCount() {
		return count;  
	}
}
```

# introduction通知

	参考文档:P411

定义introduction通知需要实现IntroductionAdvisor接口。
```java
public interface IntroductionInterceptor extends MethodInterceptor {
	boolean implementsInterface(Class intf);
}
```