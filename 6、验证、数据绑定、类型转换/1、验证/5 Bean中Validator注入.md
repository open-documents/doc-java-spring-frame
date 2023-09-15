
因为LocalValidatorFactoryBean实现了三个接口：
- `jakarta.validation.ValidatorFactory`
- `jakarta.validation.Validator`
- `org.springframework.validation.Validator`

因此可以按需注入接口类型：
```java
import jakarta.validation.Validator;
@Service 
public class MyService {   
	@Autowired   
	private Validator validator;
}
```
```java
import org.springframework.validation.Validator;
@Service 
public class MyService {   
	@Autowired   
	private Validator validator;
}
```