
参考文档：P186~P

BeanDefinition Profiles使得容器能在不同的环境下注册不同的bean。

两个不同环境下的数据源：开发环境下使用内存数据源，QA或生产环境下使用JDNI数据源。

内存数据源：
```java
@Bean 
public DataSource dataSource() {   
	return new EmbeddedDatabaseBuilder()
	.setType(EmbeddedDatabaseType.HSQL)
	.addScript("my-schema.sql")
	.addScript("my-test-data.sql")
	.build();
}
```
JDNI数据源：
```java
@Bean(destroyMethod = "") 
public DataSource dataSource() throws Exception {   
	Context ctx = new InitialContext();   
	return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

# 一、基于注解@Proflie

参考文档：P188~P192.
-- --
## 1.1）标注在@Configuration类上

当@Profile注解标注在@Configuration类上时，与这个类有关的@Import和所有@Bean方法受到影响。

开发环境下使用内存数据源：
```java
@Configuration 
@Profile("development") 
public class StandaloneDataConfig {   
	@Bean   
	public DataSource dataSource() {   
		return new EmbeddedDatabaseBuilder()   
		.setType(EmbeddedDatabaseType.HSQL)   
		.addScript("classpath:com/bank/config/sql/schema.sql")   
		.addScript("classpath:com/bank/config/sql/test-data.sql")   
		.build();
	}
}
```
QA或生产环境下使用JDNI数据源：
```java
@Configuration 
@Profile("production") 
public class JndiDataConfig {   
	@Bean(destroyMethod = "")
	public DataSource dataSource() throws Exception {   
		Context ctx = new InitialContext();   
		return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource"); 
	}
}
```
## 1.2）标注在@Bean方法上

@Profile注解也能标注在@Bean方法上。--1--

当方法重写时，父类方法与子类方法的@Profile必须保持一致，如果不一致，则只有父类方法上的@Profile生效。

举个--1--栗子：
```java
@Configuration 
public class AppConfig {   
	@Bean("dataSource")   
	@Profile("development")
	public DataSource standaloneDataSource() {   
		return new EmbeddedDatabaseBuilder()   
		.setType(EmbeddedDatabaseType.HSQL)   
		.addScript("classpath:com/bank/config/sql/schema.sql")   
		.addScript("classpath:com/bank/config/sql/test-data.sql")   
		.build(); 
	}  
	@Bean("dataSource")
	@Profile("production")
	public DataSource jndiDataSource() throws Exception { 
		Context ctx = new InitialContext(); 
		return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource"); 
	}
}
```
-- --
## 2）@Profile的逻辑运算符

基于@Profile注解的值支持稍微复杂的逻辑运算符：&、|、！。

混合使用 & 和 | 时，需要使用括号，如  env1 & (env2 | env3)。
-- --
## 3）合成注解

@Profile能被作为元注解去自定义合成注解。举个栗子：
```java
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME) 
@Profile("production") 
public @interface Production {}
```



# 二、基于<beans/>标签的profile属性

参考文档：P192~P193。
-- --
## 1.1）在不同xml文件中定义不同的profile。

之前的栗子可以写成下面这样：
```xml
<!--file1-->
<beans profile="development"   
	   xmlns="http://www.springframework.org/schema/beans"   
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
	   xmlns:jdbc="http://www.springframework.org/schema/jdbc"   
	   xsi:schemaLocation="...">   
	
	<jdbc:embedded-database id="dataSource">   
		<jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>  
		<jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>   
		</jdbc:embedded-database>
</beans> 
<!--file2-->
<beans profile="production"   
	   xmlns="http://www.springframework.org/schema/beans"  
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
	   xmlns:jee="http://www.springframework.org/schema/jee"   
	   xsi:schemaLocation="..."> 

	<jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

## 1.2）在同一个xml中定义不同的profile

使用嵌套的<beans/>标签，但是嵌套的<beans/>标签只能是<beans/>标签的最后一个元素，像下面这样：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"   
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
	   xmlns:jdbc="http://www.springframework.org/schema/jdbc"   
	   xmlns:jee="http://www.springframework.org/schema/jee"   
	   xsi:schemaLocation="...">   
	   
	<!-- other bean definitions -->
	<beans profile="development">
		<jdbc:embedded-database id="dataSource">
			<jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
			<jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/> 
		</jdbc:embedded-database>
	</beans> 
	<beans profile="production">
		<jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
	</beans>
</beans>
```
-- --
## 2）profile属性的逻辑运算符

不同于@Profile，在xml中只支持实现 &和！，不能实现 |。

逻辑&的实现：
```xml
<beans xmlns="http://www.springframework.org/schema/beans"   
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
	   xmlns:jdbc="http://www.springframework.org/schema/jdbc"   
	   xmlns:jee="http://www.springframework.org/schema/jee"   
	   xsi:schemaLocation="...">   
	   
	<!-- other bean definitions -->  
	<beans profile="production"> 
		<beans profile="us-east"> 
			<jee:jndi-lookup id="dataSource" jndiname="java:comp/env/jdbc/datasource"/>
		</beans> 
	</beans>
</beans>
```



# 三、激活Profile

参考文档：P194~P195

## 1）使用Environment API

最简单的方式就是使用Environment API。举个栗子：
```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(); ctx.getEnvironment().setActiveProfiles("development"); 
// ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```
-- --
## 2）spring.profiles.active属性

可以通过系统环境变量、JVM系统变量、web.xml中servlet环境参数、JDNI中的条目来指定spring.profiles.active值。

```shell
-Dspring.profiles.active="profile1,profile2"
```

# 四、默认的Profile（P195）

Profile名称为"default"，代表默认启用的Profile，当任意的Profile指定后，就不会启用默认的Profile。
```java
@Configuration 
@Profile("default") 
public class DefaultDataConfig {   
	@Bean   
	public DataSource dataSource() {   
		return new EmbeddedDatabaseBuilder()   
		.setType(EmbeddedDatabaseType.HSQL)   
		.addScript("classpath:com/bank/config/sql/schema.sql")   
		.build(); 
	}
}
```

