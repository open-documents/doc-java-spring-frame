	参考文档：P261~P263

# 接口定义

Formatter
```java
public interface Formatter<T> extends Printer<T>, Parser<T> { }
```
Printer
```java
public interface Printer<T> {
	String print(T fieldValue, Locale locale);
}
```
Parser
```java
public interface Parser<T> {
	T parse(String clientValue, Locale locale) throws ParseException;
}
```

