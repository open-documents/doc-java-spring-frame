
将LocalValidationFactoryBean配置为bean，这样配置后就开启了bean的验证功能。
```java
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;
@Configuration 
public class AppConfig {  
	@Bean   
	public LocalValidatorFactoryBean validator() { 
		return new LocalValidatorFactoryBean();
	}
}
```
或通过xml配置的方式：
```xml
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>
```


LocalValidatorFactoryBean配置一个SpringConstraintValidatorFactory。用于Constraint。