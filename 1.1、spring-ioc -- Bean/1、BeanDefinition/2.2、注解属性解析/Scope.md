
Scope属性由函数式接口ScopeMetadataResolver实例解析，ScopeMetadataResolver有两个实现类：
- AnnotationScopeMetadataResolver
- Jsr330ScopeMetadataResolver

下面是ScopeMetadataResolver接口的定义。
```java
@FunctionalInterface  
public interface ScopeMetadataResolver {
	ScopeMetadata resolveScopeMetadata(BeanDefinition definition);
}
```

# AnnotationScopeMetadataResolver
```java
public ScopeMetadata resolveScopeMetadata(BeanDefinition definition) {
	// 默认单例模式
    ScopeMetadata metadata = new ScopeMetadata();  
    if (definition instanceof AnnotatedBeanDefinition annDef) {  
	    // this.scopeAnnotationType = Scope.class, 可通过setScopeAnnotationType()方法改变
	    // 获取@Scope注解的属性
        AnnotationAttributes attributes = AnnotationConfigUtils.attributesFor(annDef.getMetadata(), this.scopeAnnotationType);  
        if (attributes != null) {  
            metadata.setScopeName(attributes.getString("value"));  
            ScopedProxyMode proxyMode = attributes.getEnum("proxyMode");  
	        if (proxyMode == ScopedProxyMode.DEFAULT) {  
	            proxyMode = this.defaultProxyMode;  
            }  
            metadata.setScopedProxyMode(proxyMode);  
        }  
    }  
    return metadata;  
}
```


# Jsr330ScopeMetadataResolver

要了解该类的resolveScopeMetadata()，得先看该类的唯一属性和构造函数。

该类有唯一的属性和两个向该属性中添加元素的方法。
```java
// key: 注解的全限定名
// value: scope值
private final Map<String, String> scopeMap = new HashMap<>();

public final void registerScope(Class<?> annotationType, String scopeName) {  
    this.scopeMap.put(annotationType.getName(), scopeName);  
}
public final void registerScope(String annotationType, String scopeName) {  
    this.scopeMap.put(annotationType, scopeName);  
}
```
该类的构造函数向scopeMap属性中添加了@Singleton对应的singleton scope。
```java
public Jsr330ScopeMetadataResolver() {  
    registerScope("jakarta.inject.Singleton", BeanDefinition.SCOPE_SINGLETON);  
}
```
下面是resolveScopeMetadata()的定义。
```java
public ScopeMetadata resolveScopeMetadata(BeanDefinition definition) {  
    ScopeMetadata metadata = new ScopeMetadata();  
    // SCOPE_PROTOTYPE = "prototype"
    // BeanDefinition的Scope属性默认为prototype
    metadata.setScopeName(BeanDefinition.SCOPE_PROTOTYPE);  
    if (definition instanceof AnnotatedBeanDefinition annDef) {  
	    // 获取类上标注的所有注解的全限定名
        Set<String> annTypes = annDef.getMetadata().getAnnotationTypes();  
        String found = null;  
        // 遍历所有注解
        for (String annType : annTypes) {  
	        // 获取该注解上的所有元注解的全限定名
	        Set<String> metaAnns = annDef.getMetadata().getMetaAnnotationTypes(annType);  
            if (metaAnns.contains("jakarta.inject.Scope")) {  
	            if (found != null) {
	                throw new IllegalStateException("Found ambiguous scope annotations on bean class [" +  
                     definition.getBeanClassName() + "]: " + found + ", " + annType);  
	            }  
	            found = annType;  
	            // 从唯一属性Map<String, String>中根据注解全限定名查询对应的scope
	            String scopeName = resolveScopeName(annType);  
	            if (scopeName == null) {  
	                throw new IllegalStateException(  
                     "Unsupported scope annotation - not mapped onto Spring scope name: " + annType);  
	            }  
	            metadata.setScopeName(scopeName);  
            }  
        }  
    }  
    return metadata;  
}
```

## 总结

- 该类需要配合ClassPathBeanDefinitionScanner和AnnotatedBeanDefinitionReader一起使用。
- 使用该类解析BeanDefinition的Scope时，Scope默认为prototype，除非使用@Singleton注解修改。
- 使用该类解析时，不能标注jakarta.inject.Scope注解。

