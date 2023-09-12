	参考文档:P344

任何通知都能在方法参数声明一个JoinPoint类型参数。

# 一、JoinPoint方法

• getArgs(): Returns the method arguments. 
• getThis(): Returns the proxy object. 
• getTarget(): Returns the target object. 
• getSignature(): Returns a description of the method that is being advised. 
• toString(): Prints a useful description of the method being advised.

# 二、`proceed(Object[])`的兼容性

	参考文档:P343、P350

在传统的AspectJ语言中，传递给proceed()方法的参数数量必须与传递通知的参数数量一致而不是与原始方法参数数量一致。

有一种在AspectJ和SpringAOP之间保证100%兼容性的写法。
确保通知签名按顺序绑定每个方法参数。
```java
@Around("execution(List<Account> find*(..)) && " + "com.xyz.myapp.CommonPointcuts.inDataAccessLayer() && " + "args(accountHolderNamePattern)") 
public Object preProcessQueryPattern(ProceedingJoinPoint pjp, String accountHolderNamePattern) throws Throwable {   
	String newPattern = preProcess(accountHolderNamePattern);
	return pjp.proceed(new Object[] {newPattern});
}
```