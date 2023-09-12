
该抽象类直接实现了BeanDefinition。
```java
public abstract class AbstractBeanDefinition 
		extends BeanMetadataAttributeAccessor  
        implements BeanDefinition, Cloneable
```

# 属性

下面的属性是BeanDefinition接口中没有的，由该抽象类提供。

## 1）额外属性

下面的属性只在AbstractBeanDefinition提供，其实现的接口BeanDefinition中没有提供。

| 属性| 描述| 类型| 相关方法|
| ---------------------------- | ------------------------------- | --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| beanClass                    |                                 | Class`<?>`                              | - getBeanClass():</br>- hasBeanClass():</br>- setBeanClass(@Nullable Class`<?>`)|
| nonPublicAccessAllowed       | 默认为true                      | boolean                                 | - isNonPublicAccessAllowed() </br>- setNonPublicAccessAllowed(boolean)                                                                                                       |
| lenientConstructorResolution | 默认为true                      | boolean                                 | - isLenientConstructorResolution()</br>- setLenientConstructorResolution(boolean)                                                                                            |
| methodOverrides              |                                 | MethodOverrides                         | - getMethodOverrides()</br>- hasMethodOverrides()</br>- setMethodOverrides(MethodOverrides)</br>- prepareMethodOverrides()                                                                                  |
| synthetic                    | 默认为false                     | boolean                                 | - isSynthetic()</br>- setSynthetic(boolean)                                                                                                                                  |

实例化

| 属性 | 描述 | 类型 | 相关方法 |
| ---- | ---- | :----: | -------- |
| instanceSupplier             | 创建实例的函数,用来代替工厂方法 | Supplier`<?>`                           | - getInstanceSupplier() </br>- setInstanceSupplier(@Nullable Supplier`<?>` instanceSupplier)                                                                                 |
| enforceInitMethod            | 是否使用局部初始化方法(initMethodName),true使用局部初始化方法,false使用全局初始化方法,默认为true,可通过applyDefaults(BeanDefinitionDefaults)设置(默认值为false)| boolean                                 | - isEnforceInitMethod()</br>- setEnforceInitMethod(boolean)                                                                                                                  |
| enforceDestroyMethod         | 是否使用局部销毁方法(destroyMethodName),true使用局部初始化方法,false使用全局初始化方法,默认为true,可通过applyDefaults(BeanDefinitionDefaults)设置(默认值为false)| boolean                                 | - isEnforceDestroyMethod()</br>- setEnforceDestroyMethod(boolean)                                                                                                            |
| dependencyCheck              |可通过applyDefaults(BeanDefinitionDefaults)设置(默认值为AbstractBeanDefinition.DEPENDENCY_CHECK_NONE=0)| int                                     | getDependencyCheck(), setDependencyCheck()                                                                                                                                   |



自动装配

| 属性 | 描述 | 类型 | 相关方法 |
| ---- | ---- | :----: | -------- |
| autowireMode                 | 装配模式,默认值为0,可通过applyDefaults()设置. | int(AutowireCapableBeanFactory)                                     | - getAutowireMode()</br>- getResolvedAutowireMode()</br>- setAutowireMode()                                                                                                              |
| qualifiers                   |                                 | Map<String, AutowireCandidateQualifier> | - getQualifier(String):</br>- hasQualifier(String):</br>- getQualifiers()</br>- addQualifier(AutowireCandidateQualifier): </br>- copyQualifiersFrom(AbstractBeanDefinition): |


## 2）BeanDefinition中已有的属性

下面的属性根据BeanDefinition的定义需要在实现类中定义，但是AbstractBeanDefinition补充了与该属性的简单方法。

| 属性                      | 描述                                                                    | 类型                      | 补充的方法                                                |
| ------------------------- | ----------------------------------------------------------------------- | ------------------------- | --------------------------------------------------------- |
|resource|该BeanDefinition来自的Resource|Resource(@Nullable)|- getResource()</br>- setResource(@Nullable Resource)</br>- setResourceDescription(@Nullable String)</br>- setOriginatingBeanDefinition(BeanDefinition)|
| abstractFlag              | 决定该BeanDefinition能否实例化bean,即只能作为一个子BeanDefinition的父亲 | boolean                   | - setAbstract()                                           |
| constructorArgumentValues |                                                                         | ConstructorArgumentValues | - setConstructorArgumentValues(ConstructorArgumentValues) |
| propertyValues            |                                                                         | MutablePropertyValues     | - setPropertyValues(MutablePropertyValues)                |

# 为属性赋默认值--applyDefaults()

设置的属性如下：
- lazyInit（实例化相关属性，BeanDefinition提供的属性）
- dependsOn（实例化相关属性，BeanDefinition提供的属性）
- initMethodName（实例化相关属性，BeanDefinition提供的属性）
- enforceInitMethod（实例化相关属性，该类提供的属性）
- destroyMethodName（实例化相关属性，BeanDefinition提供的属性）
- enforceDestroyMethod（实例化相关属性，该类提供的属性）
- autowireMode（自动装配相关属性，该类提供的属性）

代码如下（简单）：
```java
public void applyDefaults(BeanDefinitionDefaults defaults) {  
    Boolean lazyInit = defaults.getLazyInit();  
    if (lazyInit != null) {  
        setLazyInit(lazyInit);  
    }  
    setAutowireMode(defaults.getAutowireMode());  
    setDependencyCheck(defaults.getDependencyCheck());  
    setInitMethodName(defaults.getInitMethodName());  
    setEnforceInitMethod(false);  
    setDestroyMethodName(defaults.getDestroyMethodName());  
    setEnforceDestroyMethod(false);   
}
```



