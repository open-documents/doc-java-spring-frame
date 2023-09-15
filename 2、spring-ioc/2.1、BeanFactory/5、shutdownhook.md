如果在非web应用环境(例如,在富客户端的桌面环境中)中使用Spring的IoC容器，则向JVM注册一个关闭挂钩。这样做可以确保一个优雅的关闭，并在单实例bean上调用相关的销毁方法，从而释放所有资源。您仍然必须正确地配置和实现这些销毁回调。
为了注册一个关闭钩子，调用Configizable AppplicationContext接口上声明的register ShutdownHook ( )方法，如下例所示：
```java
import org.springframework.context.ConfigurableApplicationContext; 
import org.springframework.context.support.ClassPathXmlApplicationContext;
public final class Boot {   
	public static void main(final String[] args) throws Exception {   
		ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");   
		// add a shutdown hook for the above context...   
		ctx.registerShutdownHook();  
		// app runs here...   
		// main method exits, hook is called prior to the app shutting down... 
	}
}
```
