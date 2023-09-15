Lookup方法注入是容器覆盖容器管理bean中的方法并返回容器中另一个命名bean的查找结果的能力。
Lookup方法通常用于prototype bean。

因为Spring通过cglib实现这个功能，因此有几点需要注意：
- lookup所在的类不能是final。
- lookup方法不能是final。
- 如果lookup方法是abstract，Spring会实现这个方法；如果lookup方法是具体的，Spring会重写这个方法。
- A further key limitation is that lookup methods do not work with factory methods and in particular not with @Bean methods in configuration classes, since, in that case, the container is not in charge of creating the instance and therefore cannot create a runtime-generated subclass on the fly.

下面是lookup方法所在的类：
```java
public abstract class CommandManager {   
	public Object process(Object commandState) {   
		// grab a new instance of the appropriate Command interface   
		Command command = createCommand();   
		// set the state on the (hopefully brand new) Command instance   
		command.setState(commandState);   
		return command.execute();
	}
	// okay... but where is the implementation of this method?   
	protected abstract Command createCommand();
}
```
-- --
# 基于xml的lookup注入
```xml
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
<!-- inject dependencies here as required --> 
</bean> 
<bean id="commandManager" class="fiona.apple.CommandManager">   
	<lookup-method name="createCommand" bean="myCommand"/>
</bean>
```
-- --
# 基于注解的Lookup注入(47~53)
```java
public abstract class CommandManager {   
	public Object process(Object commandState) {   
		Command command = createCommand();   
		command.setState(commandState);   
		return command.execute();  
	}   
	@Lookup("myCommand") 
	protected abstract Command createCommand();
}
```


当一个singleton类的方法内部需要使用prototype类实例的时候，需要用到这个特性。


通过使用Java配置，可以创建CommandManager的一个子类，在该子类中，抽象的createCommand ( )方法被重写，以便查找新的(prototype)command对象。下面的例子说明如何做到这一点：

-- --
# 基于@Bean

```java
@Configuration
public class AppConfig{
	@Bean 
	@Scope("prototype")
	 public AsyncCommand asyncCommand() {
		AsyncCommand command = new AsyncCommand();
		// inject dependencies here as required   
		return command;
	}
	@Bean 
	public CommandManager commandManager() {   
	// return new anonymous implementation of CommandManager with createCommand()   
	// overridden to return a new prototype Command object   
		return new CommandManager() {
			protected Command createCommand() {
				return asyncCommand();  
		    }
		}
	}
}
```
