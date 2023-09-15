	参考文档：P257~P260

ConversionService接口也定义了执行类型转换的API，一般Converter是在这里面被使用，可以将ConversionService视作对Converter的代理。

大多数ConversionService的实现类实现的是ConfigurableConversionService接口。

# 一、接口定义

ConversionService
```java
package org.springframework.core.convert; 
public interface ConversionService {   
	boolean canConvert(Class<?> sourceType, Class<?> targetType);   
	<T> T convert(Object source, Class<T> targetType);   
	boolean canConvert(TypeDescriptor sourceType, TypeDescriptor targetType);   
	Object convert(Object source, TypeDescriptor sourceType, TypeDescriptor targetType);
}
```
ConverterRegistry
```java
public interface ConverterRegistry {
	void addConverter(Converter<?, ?> converter);
	void addConverter(GenericConverter converter);
	<S, T> void addConverter(Class<S> sourceType, Class<T> targetType, Converter<? super S, ? extends T> converter);
	void addConverterFactory(ConverterFactory<?, ?> factory);
	void removeConvertible(Class<?> sourceType, Class<?> targetType);
}
```
ConfigurableConversionService
```java
public interface ConfigurableConversionService extends ConversionService, ConverterRegistry
```

# DefaultConversionService

对于大多数转换，可以直接指定target类型完成，但是不能完成复杂的转换，如Integer的List转换成String类型的List。

```java
DefaultConversionService cs = new DefaultConversionService(); 
List<Integer> input = ... 
cs.convert(input, TypeDescriptor.forObject(input), // List<Integer> type descriptor   
		   TypeDescriptor.collection(List.class, TypeDescriptor.valueOf(String.class)));
```

DefaultConversionService默认注册了许多Converter。


# 注册DefaultConversionService

为了在Spring中注册默认的ConversionService，需要添加一个id为conversionService的工厂bean。
```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean"/>
```
默认的ConversionService能完成String，Number，Enum，Collection，Map以及其他类型之间的转换。

通过设置ConversionServiceFactoryBean的converters属性，可以来提供自定义的Converter或用自定义的Converter来覆盖默认的Converter。
自定义的Converter可以是实现了Converter、ConverterFactory或GenericConverter接口的任何类。
```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean"> 
	<property name="converters">
		<set>
			<bean class="example.MyCustomConverter"/>
		</set>
	</property>
</bean>
```
