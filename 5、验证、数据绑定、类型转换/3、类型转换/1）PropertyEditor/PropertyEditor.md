	参考文档：246

Spring提供大量的内置PropertyEditor，所有都位于 org.springframework.beans.propertyeditors包。

它们绝大多数被 BeanWrapperImpl 注册，其中包括：


在完成对非基础类型（java8种）对象属性注入时，可以通过ref引用的方式完成，但是有时候我们想通过仅指定一个String类型的value值，希望Spring能够将String类型的值转换成相应对象，此时可以用到PropertyEditor。比如，可读性更强的 `String:'2007-14-09'`（或者其他可读性强的方式）转换成 `Date`实例对象。

# PropertyEditor实现类

内建的PropertyEditor实现都位于 `org.springframework.beans.propertyeditors` 包中。绝大多数内建的实现都在 BeanWrapperImpl 中注册。

下面是内建的PropertyEditor实现：

| 实现类                  | 描述                                                                                                                                                |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| ByteArrayPropertyEditor | 把String类型字符串数组数组转换成对应的字节数组。默认被BeanWrapperImpl注册。                                                                         |
| ClassEditor             | 把String字符串转换成对应的Class，或者相反，当Class无法找到时，抛出IllegalArgumentException。默认被BeanWrapperImpl注册。                             |
| CustomBooleanEditor     | 把String类型字符串转换成bool值。默认被BeanWrapperImpl注册，但是可以通过自定义一个Editor来覆盖。                                                     |
| CustomNumberEditor      | 把String类型字符串转换成Number子类实例。默认被BeanWrapperImpl注册，但是可以通过自定义一个Editor来覆盖。                                             |
| FileEditor              | 把String类型字符串转换成File对象实例。默认被BeanWrapperImpl注册。                                                                                   |
| LocaleEditor            | 把String类型字符串转换成Locale对象实例，或者相反。字符串格式为 `[language]_[country]_[variant]`(也能使用空格作为分隔符).默认被BeanWrapperImpl注册。 |
| PropertiesEditor        | 把String类型字符串转换成`java.util.Properties`对象实例。默认被BeanWrapperImpl注册。                                                                 |
| URLEditor               | 把String类型字符串转换成URL对象实例。默认被BeanWrapperImpl注册。                                                                                    |
| InputStreamEditor       | 把String类型字符串转换成InputStream对象实例，通过间接使用ResourceEditor和Resource。默认被BeanWrapperImpl注册。                                      |
| CustomCollectionEditor  | 把String类型字符串转换成Collection对象实例。                                                                                                        |
| CustomDateEditor        | 把String类型字符串转换成Date对象实例。没有默认注册，StringTrimmerEditor                                                                              |
| StringTrimmerEditor| 去除String类型字符串的空白，可以将空字符串转换成null值。没有默认注册，必须使用者手动注册。                                                                                              |

# 设置搜索PropertyEditor的路径（未理解248）

Spring使用java.beans.PropertyEditorManager来设置所需要的PropertyEditor路径。搜索路径包括 `sun.bean.editors`（包含诸如Font，Color的PropertyEditor实现类）。



# 自定义PropertyEditor

Spring大部分默认属性编辑器都直接扩展(extends)于 `java.beans.PropertyEditorSupport` 类，用户也可以通过扩展PropertyEditorSupport实现自己的属性编辑器。举个栗子：

如果一个类中有Date类型的属性，不通过ref实例的方式，而是仅给定一个value字符串值："2023-1-20"，我们需要将其解析成Date对象，并注入到对象实例中。
```java
// 将字符串解析成Date对象
public class DateEditor extends PropertyEditorSupport {  
    @Override  
    public void setAsText(String text) throws IllegalArgumentException {  
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-mm-dd");  
        try {  
            Date date = simpleDateFormat.parse(text);  
            this.setValue(date);  
        } catch (ParseException e) {  
            e.printStackTrace();  
        }  
    }  
}
```

# 注册自定义的PropertyEditor

	参考文档：P249~P252

如果使用BeanFactory，用户需要手动调用 `registerCustomEditor(Class requiredType, PropertyEditor propertyEditor)` 方法注册自定义属性编辑器。（不知道怎么做）

如果使用ApplicationContext，用户需要在配置文件通过 `CustomEditorConfigurer` 注册。

另外，如果需要将String字符串转换成的目标类和自定义的PropertyEditor在一个包，并且有相同的名字，那么不用显示注册。
举个栗子：
```java
// 包结构
// com
//   chank
//     pop
//       Something  
//       SomethingEditor
```

通过后者注册前面提到的DateEditor：

1）第一种方式：
```xml
<!--test.xml-->
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">  
    <property name="customEditors">  
        <map>            
	        <entry key="java.util.Date" value="propertyEditor.DateEditor"/>  
        </map>
	</property>
</bean>
```
2）第二种方式：
注册 `PropertyEditorRegistrar` 接口的实现类间接注册自定义的PropertyEditor，这个接口只有一个方法registerCustomEditors，用以注册多个自定义的PropertyEditor。
```java
public class CustomPropertyEditorRegistrar implements PropertyEditorRegistrar {  
    @Override  
    public void registerCustomEditors(PropertyEditorRegistry registry) {  
        registry.registerCustomEditor(Date.class,new DateEditor());  
    }  
    
    public CustomPropertyEditorRegistrar() {}  
}
```
```xml
<!--test.xml-->
<bean id="customPropertyEditorRegistrar" class="propertyEditor.CustomPropertyEditorRegistrar"/>
<bean class="org.springframework.beans.factory.config.CustomEditorConfigurer">  
    <property name="propertyEditorRegistrars">  
        <list>            
	        <ref bean="customPropertyEditorRegistrar"/>  
        </list>
	</property>
</bean>  
```

# 实验

有Date属性的类：
```java
public class Bean {  
    private Date date;  
  
    public void setDate(Date date) {  
        this.date = date;  
    }  
  
    public Bean() {  
    }  
}
```
配置文件：
```xml
<!--test.xml-->
<beans>
	<!-- 加上前面的CustomEditorConfigurer的一种配置-->
	<bean id="bean" class="Bean">
		<property name="date" value="2023-1-20"/>
	</bean> 
</beans>
```
测试：
```java
public void test(){  
    ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("test.xml");
    Bean bean = applicationContext.getBean("bean", Bean.class);  
}
```


