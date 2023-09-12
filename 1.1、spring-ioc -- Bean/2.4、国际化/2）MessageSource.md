
	参考文档:P201~

ApplicationContext继承了MessageSource接口，因而提供了i18n功能。

Spring提供了MessageSource和HierarchicalMessageSource接口。

MessageSource接口的定义：
```java
public interface MessageSource {
	@Nullable  
	String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);
	String getMessage(String code, @Nullable Object[] args, Locale locale) throws NoSuchMessageException;
	String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException;
}
```

# 理论知识

当ApplicationContext被加载时，它自动寻找定义在这个context中的名字叫messageSource的MessageSource bean。
它将相关工作交给这个bean来完成，如果找不到这样的bean，就到父context中去寻找同样的bean，如果一直找不到，就会实例化一个空的DelegatingMessageSource，只是为了能够接受对上述方法的调用。

# 准备材料

```properties
# format.properties 
message=Alligators rock!
```
```properties
# exceptions.properties 
argument.required=The {0} argument is required.
```
# 通过application直接使用MessageSource接口中的方法

```java
MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");   
String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);   System.out.println(message);
// Alligators rock!
```

# 、通过MessageSource对象使用MessageSource接口中的方法

除了通过ApplicationContext间接地调用MessageSource接口，还可以单独使用MessageSource对象来直接调用MessageSource接口。

## 获取MessageSource接口


```java
// MessageSourceAware接口
public class ExtendBean implements MessageSourceAware {
	@Override
	public void setMessageSource(MessageSource messageSource) {
		this.setterMs = messageSource;
	}
}

// 从容器直接注入
public class ExtendBean implements MessageSourceAware {
	@Autowired
	private MessageSource autowiredMs;
}
```

需要注意的是，使用@Autowired等方式直接获取MessageSource类型的数据得到的是添加到容器的那个Bean，而其他方式获取到的是ApplicationContext。



```java
@Configuration
public class I18nApp {
	@Bean("messageSource")
	ResourceBundleMessageSource resourceBundleMessageSource() {
		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
		messageSource.setBasenames(new String[] { "i18n", "extend" });//添加资源名称
		return messageSource;
	}
}
```
或
```javascript
<beans>
    <bean id="messageSource"      class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>i18n</value>
                <value>extend</value>
            </list>
        </property>
    </bean>
</beans>
```

basenames这个Setter用于指定*.properties资源文件的名称，规则和前面介绍的ResourceBundle一样。然后就可以通过ApplicationContext::getMessage方法获取对应的资源了：

```java
@Configuration
public class I18nApp {
	@Bean("messageSource")
	ResourceBundleMessageSource resourceBundleMessageSource() {
		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
		messageSource.setBasenames(new String[] { "i18n", "extend" });
		return messageSource;
	}

	public static void main(String[] args) {
		ApplicationContext context = new AnnotationConfigApplicationContext(I18nApp.class);
		System.out.println("Spring Default 1:" + context.getMessage("say", null, Locale.getDefault()));
		System.out.println("Spring Default 2:" + context.getMessage("say", null, null));
		System.out.println("Spring Chinese:" + context.getMessage("say", null, Locale.SIMPLIFIED_CHINESE));
		System.out.println("Spring Us English:" + context.getMessage("say", null, Locale.US));
		System.out.println("Spring Custom:" + context.getMessage("say", null, new Locale("web", "BASE64")));
		System.out.println("Spring Argument:" + context.getMessage("info", new String[] {"chkui"},null));
		System.out.println("Spring Info:" + context.getMessage("say", null, null));
	}
}
```

# 占位符替换

注意上面的示例代码的这一行：_context.getMessage("info", new String[] {"chkui"},null))，_这里的_getMessage_向方法传递了一个数组，他用于替换资源文件中的占位符号。在例子中我们除了i18n还加载了一个_extend.properties_文件，文件内容如下：

```javascript
info={0}\u5E05\u7684\u8BA9\u4EBA\u6CA1\u813E\u6C14\u3002
```

文件中的_{0}_表示这个位置用数组中的[0]位置的元素替换。

还有一点需要注意的是，*.properties文件输入中文等UTF-8的符号时需要保留上面这种ACS的格式，现在大部分IDE都会自动处理的，切记不要为了方便看内容将*.properties的编码格式切换为UTF-8。



# MessageSource的内置实现类

Spring提供了3个实现类：ResourceBundleMessageSource、ReloadableResourceBundleMessageSource、StaticMessageSource。
3个实现类都实现了HierarchicalMessageSource接口，用以能够处理嵌套的消息。

- ResourceBundleMessageSource：基础实现类，基于jdk的ResourceBundle。不合并相同的base name文件，而是只使用第一个。
- ReloadableResourceBundleMessageSource：比ResourceBundleMessageSource更加灵活，支持Spring Resource Location（不仅仅从classpath）、支持bundle property文件热重加载。
- StaticMessageSource：几乎不使用，但是它提供了一种添加message到source的方法。
