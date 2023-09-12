
BeanDefinition是一个接口，下面是它的定义。
```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement
```

# 属性

根据BeanDefinition本身提供的方法，可以看出BeanDefinition实现类需要提供下面的属性。

| 属性| 描述|类型| 支持的bean方法|
| ------------------------- | ----------------------------------------------------------------------- |:-------------------------:| ---------------------------------------------------------- |
| description               | 该BeanDefinition的描述                                                  |          String           | - getDescription()</br>- setDescription(@Nullable String)|
| role                      |                                                                         |                           |- getRole()</br>- setRole(int) |
| beanClassName             | 该BeanDefinition对应的类的名称                                          |          String           | - getBeanClassName()</br>- setBeanClassName(@Nullable String)|
| resolvableType            |                                                                         |      ResolvableType       |- getResolvableType()|
| resource                  | 该BeanDefinition来自的Resource                                          |         Resource          | - getResourceDescription()</br>- getOriginatingBeanDefinition()|
| parentName                | 父BeanDefinition的名称                                                  |          String           | - getParentName()</br> - setParentName(@Nullable String)|
| abstractFlag              | 决定该BeanDefinition能否实例化bean,即只能作为一个子BeanDefinition的父亲 | boolean                   | - isAbstract()                                           |


| 属性                      | 描述                                                              | 类型                      | 支持的bean方法                                                                                                                                         |
| ------------------------- | ----------------------------------------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| factoryBeanName           |                                                                   | String                    | - getFactoryBeanName()</br>- setFactoryBeanName(@Nullable String)                                                                                      |
| factoryMethodName         |                                                                   | String                    | - getFactoryMethodName()</br>- setFactoryMethodName(@Nullable String)                                                                                  |
| constructorArgumentValues | 构造函数参数的值                                                  | ConstructorArgumentValues | - getConstructorArgumentValues()</br>- hasConstructorArgumentValues()</br>- set方法在抽象类AbstractBeanDefinition中提供                                |
| propertyValues            |                                                                   | MutablePropertyValues     | - getPropertyValues()</br>- hasPropertyValues(): 默认方法</br>- set方法在子抽象类AbstractBeanDefinition中提供                                          |
| scope                     | 该bean所属的target scope                                          | String                    | - get(): 可返回null.</br>- isSingleton(): 判断该BeanDefinition是否以单例实例化bean.</br>- isPrototype(): 判断该BeanDefinition是否以原型模式实例化bean. |
| lazyInit                  | 决定bean实例是否懒加载                                            | boolean                   | - isLazyInit() </br>- setLazyInit(boolean)                                                                                                             |
| dependsOn                 | 在该BeanDefinition实例化bean前,先实例化该属性指定的BeanDefinition | `String[]`                | - getDependsOn()</br>- setDependsOn(@Nullable String...)                                                                                               |
| initMethodName            | 调用的初始化方法名称,可通过applyDefaults(BeanDefinitionDefaults)设置(默认值为null)| String                    | - getInitMethodName()</br>- setInitMethodName(@Nullable String)|
| destroyMethodName         | 调用的销毁方法名称,可通过applyDefaults(BeanDefinitionDefaults)设置(默认值为null)| String                    |- getDestroyMethodName()</br>- setDestroyMethodName(@Nullable String)|


| 属性 | 描述 | 类型 | 支持的bean方法 |
| ---- | ---- | ---- | -------------- |
| primary                   | 决定该BeanDefinition实例化的bean在参与其它bean的自动装配时是否是primary |          Boolean          | - isPrimary()</br>- setPrimary(boolean)                                                                                                                |
| autowireCandidate| 决定该BeanDefinition实例化的bean是否参与其它bean的自动装配|boolean| - isAutowireCandidate()</br> - setAutowireCandidate(boolean)|

# beanName

```java
// ------------------------ AnnotationBeanNameGenerator -----------------------------------------
public String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry) {  
    if (definition instanceof AnnotatedBeanDefinition) {  
        String beanName = determineBeanNameFromAnnotation((AnnotatedBeanDefinition) definition);  
        if (StringUtils.hasText(beanName)) {  
            // Explicit bean name found.  
            return beanName;  
        }  
    }  
    // Fallback: generate a unique default bean name.  
    return buildDefaultBeanName(definition, registry);  
}
```

```java
// ------------------------ AnnotationBeanNameGenerator -----------------------------------------
protected String determineBeanNameFromAnnotation(AnnotatedBeanDefinition annotatedDef) {  
    AnnotationMetadata amd = annotatedDef.getMetadata();  
    Set<String> types = amd.getAnnotationTypes();  
    String beanName = null;  
    for (String type : types) {  
        AnnotationAttributes attributes = AnnotationConfigUtils.attributesFor(amd, type);  
        if (attributes != null) {  
            Set<String> metaTypes = this.metaAnnotationTypesCache.computeIfAbsent(type, key -> {  
                Set<String> result = amd.getMetaAnnotationTypes(key);  
            return (result.isEmpty() ? Collections.emptySet() : result);  
            });  
	        if (isStereotypeWithNameValue(type, metaTypes, attributes)) {  
	            Object value = attributes.get("value");  
	            if (value instanceof String) {  
	                String strVal = (String) value;  
                    if (StringUtils.hasLength(strVal)) {  
			            if (beanName != null && !strVal.equals(beanName)) {
				            ...
                        }  
                    beanName = strVal;  
	                }  
	            }  
	        }  
	    }  
    }  
    return beanName;  
}
```

```java
protected String buildDefaultBeanName(BeanDefinition definition,BeanDefinitionRegistry registry) {      return buildDefaultBeanName(definition);  
}
protected String buildDefaultBeanName(BeanDefinition definition) {  
    String beanClassName = definition.getBeanClassName();  
    ... 
    String shortClassName = ClassUtils.getShortName(beanClassName);  
    return Introspector.decapitalize(shortClassName);  
}
```
