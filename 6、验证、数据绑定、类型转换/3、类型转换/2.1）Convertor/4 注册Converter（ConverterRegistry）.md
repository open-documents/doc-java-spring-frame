下面是接口ConverterRegistry的定义：
```java
public interface ConverterRegistry {
	void addConverter(Converter<?, ?> converter);
	<S, T> void addConverter(Class<S> sourceType, Class<T> targetType, Converter<? super S, ? extends T> converter);
	void addConverter(GenericConverter converter);
	void addConverterFactory(ConverterFactory<?, ?> factory);

	void removeConvertible(Class<?> sourceType, Class<?> targetType);
}
```
