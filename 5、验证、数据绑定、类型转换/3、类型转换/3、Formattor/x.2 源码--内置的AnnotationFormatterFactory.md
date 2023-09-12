```java
public final class NumberFormatAnnotationFormatterFactory implements AnnotationFormatterFactory<NumberFormat> { 
	public Set<Class<?>> getFieldTypes() {
		return new HashSet<Class<?>>(asList(new Class<?>[] {
			Short.class, Integer.class, Long.class, Float.class, Double.class,
			BigDecimal.class, BigInteger.class }));
	} 
	public Printer<Number> getPrinter(NumberFormat annotation, Class<?> fieldType) { 
		return configureFormatterFrom(annotation, fieldType); 
	} 
	public Parser<Number> getParser(NumberFormat annotation, Class<?> fieldType) { 
		return configureFormatterFrom(annotation, fieldType); 
	} 
	
	private Formatter<Number> configureFormatterFrom(NumberFormat annotation, Class<?> fieldType) { 
		if (!annotation.pattern().isEmpty()) {  
			return new NumberStyleFormatter(annotation.pattern()); 
		} else { 
			Style style = annotation.style(); 
			if (style == Style.PERCENT) { 
				return new PercentStyleFormatter(); 
			} else if (style == Style.CURRENCY) {
				return new CurrencyStyleFormatter();  
			} else {
				return new NumberStyleFormatter(); 
			} 
		} 
	}
}
```
