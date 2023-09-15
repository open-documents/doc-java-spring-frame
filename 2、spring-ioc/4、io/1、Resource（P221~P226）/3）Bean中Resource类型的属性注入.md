参考文档：P229~P231

当Bean内部动态根据path动态获得Resource时，可能用到ResourceLoader或者ResourcePatternResolver，但是如果只是简单地静态获取Resource时，完全可以不用到ResourceLoader或ResourcePatternResolver接口，而是直接传入一个String类型的值就能获得Resource。

传入的String会被PropertyEditor处理，返回一个Resource，并进行注入。

举个栗子：
```java
public class MyBean {   
	private Resource template;   
	public setTemplate(Resource template) {   
		this.template = template;  
	}
}
```
## 基于xml

传入一个String给template就能得到Resource类型的实例。
```xml
<bean id="myBean" class="example.MyBean">
	<property name="template" value="some/resource/path/myTemplate.txt"/>
</bean>
```
因为上述的value没有前缀，所以template的具体实现类依据ApplicationContext的类型而定，如果希望指定返回的Resource的类型，可以为value加上前缀。
```xml
<property name="template" value="classpath:some/resource/path/myTemplate.txt">
<property name="template" value="file:///some/resource/path/myTemplate.txt"/>
```
## 基于注解

```java
@Component 
public class MyBean {   
	private final Resource template;   
	public MyBean(@Value("${template.path}") Resource template) {   
		this.template = template;  
	}
}
```

# Bean中的`Resource[]`依赖（P229~P231）

```java
@Component 
public class MyBean {   
	private final Resource[] templates;   
	public MyBean(@Value("${templates.path}") Resource[] templates) {   
		this.templates = templates;
	}
}
```
