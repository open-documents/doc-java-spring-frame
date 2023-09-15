	参考文档：P263~P266

注解驱动的Formatter

属性的格式化能通过注解配置。

要将一个注解绑定到一个Formatter，需要实现 `AnnotationFormatterFactory`接口。

# 1）接口定义

AnnotationFormatterFactory
```java
public interface AnnotationFormatterFactory<A extends Annotation> {
	Set<Class<?>> getFieldTypes(); 
	Printer<?> getPrinter(A annotation, Class<?> fieldType);  
	Parser<?> getParser(A annotation, Class<?> fieldType);
}
```


# 2）基本使用

```java
public class MyModel { 
	@NumberFormat(style=Style.CURRENCY) 
	private BigDecimal decimal;
}
```

```java
public class MyModel { 
	@DateTimeFormat(iso=ISO.DATE) 
	private Date date;
}
```

# x）内置AnnotationFormatterFactory

