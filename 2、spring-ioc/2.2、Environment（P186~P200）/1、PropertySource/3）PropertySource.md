
参考文档：P195~P200。

Environment提供了可配置的property sources层面的<font color=44cf57>查询</font>操作。

Environment的实现类：StandardEnvironment

# 向Environment中添加PropertySource

## 使用MutablePropertySourcesAPI
```java
ConfigurableApplicationContext ctx = new GenericApplicationContext(); 
MutablePropertySources sources = ctx.getEnvironment().getPropertySources(); 
sources.addFirst(new MyPropertySource());
```
-- --
## 使用@PropertySource注解（P197~P199）

使用@PropertySource注解可以快速向Environment中添加properties。

```java
@Configuration 
@PropertySource("classpath:/com/myco/app.properties") 
public class AppConfig {   
	@Autowired   
	Environment env;   
	@Bean   
	public TestBean testBean() {   
		TestBean testBean = new TestBean();   
		testBean.setName(env.getProperty("testbean.name"));   
		return testBean; 
	}
}
```

@PropertySource注解也支持placeholder解析，只要placeholder内的值已经被注册。
```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties") 
public class AppConfig {   
	@Autowired   
	Environment env;   
	@Bean   
	public TestBean testBean() {   
		TestBean testBean = new TestBean();   
		testBean.setName(env.getProperty("testbean.name"));
		return testBean;
	}
}
```


# Environment的实现类

## 1）StandardEnvironment

StandardEnvironment包含两个PropertySource对象：JVM系统属性（System.getProperties()）、系统环境变量（System.getenv()）。

StandardEnvironment会按如下顺序查找（后者覆盖前者）：
1）系统环境变量。
2）JVM系统属性。

## 2）StandardServletEnvironment

StandardServletEnvironment除了包含StandardEnvironment包含的两个PropertySource对象，还另外包含额外的PropertySource对象：servlet config、servlet context parameters、JndiPropertySource（如果有的话）。

StandardServletEnvironment会按如下顺序查找（后者覆盖前者）：
1）JVM system environment (operating system environment variables)
2）JVM system properties (-D command-line arguments)
3）JNDI environment variables (java:comp/env/ entries)
4）ServletContext parameters (web.xml context-param entries)
5）ServletConfig parameters (if applicable — for example, in case of a DispatcherServlet context)
