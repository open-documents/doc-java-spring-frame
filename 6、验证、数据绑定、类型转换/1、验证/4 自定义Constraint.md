自定义一个Constraint涉及两个方面内容：
- 一个@Constraint注解。
- 一个 `jakarta.validation.ConstraintValidator` 接口实现类。

当在一个domain model上检测到@Contraint注解时，ConstraintValidatorFactory会实例化一个与该注解相关的ConstraintValidator。

LocalValidatorFactoryBean配置一个SpringConstraintValidatorFactory。


# 一、@Constraint和ConstraintValidator定义



# 二、基本使用



```java
@Target({ElementType.METHOD, ElementType.FIELD}) 
@Retention(RetentionPolicy.RUNTIME) 
@Constraint(validatedBy=MyConstraintValidator.class) 
public @interface MyConstraint { }
```


默认情况下，LocalValidatorFactoryBean配置了SpringConstraintValidatorFactory属性，它能使用Spring的特性来创建ConstraintValidator实例，这就使得ConstraintValidator的实现类中的属性能够使用依赖注入等特性。举个栗子：
```java
import jakarta.validation.ConstraintValidator; 
public class MyConstraintValidator implements ConstraintValidator {   
	@Autowired;   
	private Foo aDependency;   
}
```


