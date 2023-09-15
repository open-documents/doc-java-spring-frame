
默认情况下，在一个application context中，aspect是单实例，这在AspectJ中被称为单实例初始化模型。

aspect能够有自己的声明周期。

使用xml定义的切面只支持单例初始化模型。
使用注解定义的切面，支持AspectJ的perthis和pertarget初始化模型，不支持percflow、percflowbelow、pertypewithin初始化模型。

# perthis初始化模型

通过在@Aspect注解中声明perthis初始化模型。

```java
@Aspect("perthis(com.xyz.myapp.CommonPointcuts.businessService())") 
public class MyAspect {   
	private int someState;   
	@Before("com.xyz.myapp.CommonPointcuts.businessService()")   
	public void recordServiceUsage() {   
		// ...
	}
}
```

# pertarget初始化模型

```java
@Aspect("pertarget(com.xyz.myapp.CommonPointcuts.businessService())") 
public class MyAspect {   
	private int someState;   
	@Before("com.xyz.myapp.CommonPointcuts.businessService()")   
	public void recordServiceUsage() {   
		// ...
	}
}
```

