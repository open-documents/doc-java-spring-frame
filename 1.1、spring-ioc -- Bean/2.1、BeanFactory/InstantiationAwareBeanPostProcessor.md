`InstantiationAwareBeanPostProcessor` 接口主要用于框架内部使用，平时尽可能使用普通的BeanPostProcessor接口。
`InstantiationAwareBeanPostProcessor` 接口继承了 `BeanPostProcessor` 接口。接口定义如下：

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
	Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException;
	boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException;
	PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)  
      throws BeansException;
}
```

