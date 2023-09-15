
# applyScopedProxyMode()

```java
static BeanDefinitionHolder applyScopedProxyMode(  
      ScopeMetadata metadata, BeanDefinitionHolder definition, BeanDefinitionRegistry registry) {  
  
    ScopedProxyMode scopedProxyMode = metadata.getScopedProxyMode();  
    if (scopedProxyMode.equals(ScopedProxyMode.NO)) {  
        return definition;  
    }  
    boolean proxyTargetClass = scopedProxyMode.equals(ScopedProxyMode.TARGET_CLASS);  
    return ScopedProxyCreator.createScopedProxy(definition, registry, proxyTargetClass);  
}
```

# attributesFor()

```java
static AnnotationAttributes attributesFor(AnnotatedTypeMetadata metadata, Class<?> annotationClass) {  
    return attributesFor(metadata, annotationClass.getName());  
}
static AnnotationAttributes attributesFor(AnnotatedTypeMetadata metadata, String annotationClassName) {  
    return AnnotationAttributes.fromMap(metadata.getAnnotationAttributes(annotationClassName));  
}
```


# attributesForRepeatable()

下面是Spring使用到该方法的地方。
```java
Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(  
	sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
```
下面是该方法的定义。
```java

static Set<AnnotationAttributes> attributesForRepeatable(AnnotationMetadata metadata,  
      Class<?> containerClass, Class<?> annotationClass) {  

	return attributesForRepeatable(metadata, containerClass.getName(), annotationClass.getName());  
}
```
下面是更加具体的逻辑。
```java
static Set<AnnotationAttributes> attributesForRepeatable(  
      AnnotationMetadata metadata, String containerClassName, String annotationClassName) {  
    Set<AnnotationAttributes> result = new LinkedHashSet<>();  
  
    // Direct annotation present?  
    // if (attributes != null) {  
    //     result.add(AnnotationAttributes.fromMap(attributes));  
	// }
    addAttributesIfNotNull(result, metadata.getAnnotationAttributes(annotationClassName));  
  
    // Container annotation present?  
    Map<String, Object> container = metadata.getAnnotationAttributes(containerClassName);  
    if (container != null && container.containsKey("value")) {  
        for (Map<String, Object> containedAttributes : (Map<String, Object>[]) container.get("value")) {  
            addAttributesIfNotNull(result, containedAttributes);  
        }  
    }  
    // Return merged result  
    return Collections.unmodifiableSet(result);  
}
```

# processCommonDefinitionAnnotations()

```java
public static void processCommonDefinitionAnnotations(AnnotatedBeanDefinition abd) {  
    processCommonDefinitionAnnotations(abd, abd.getMetadata());  
}
static void processCommonDefinitionAnnotations(AnnotatedBeanDefinition abd, AnnotatedTypeMetadata metadata) {  
	// 获取@Lazy注解的属性
    AnnotationAttributes lazy = attributesFor(metadata, Lazy.class);  
    if (lazy != null) {  
        abd.setLazyInit(lazy.getBoolean("value"));  
    }  
    else if (abd.getMetadata() != metadata) {  
        lazy = attributesFor(abd.getMetadata(), Lazy.class);  
        if (lazy != null) {  
            abd.setLazyInit(lazy.getBoolean("value"));  
        }  
    }  
    // 查看注解元数据是否标注了@Primary注解
    if (metadata.isAnnotated(Primary.class.getName())) {  
        abd.setPrimary(true);  
    }  
    // 获取注解元数据的@DependsOn注解的属性
    AnnotationAttributes dependsOn = attributesFor(metadata, DependsOn.class);  
    if (dependsOn != null) {  
        abd.setDependsOn(dependsOn.getStringArray("value"));  
    }  
    // 获取注解元数据的@Role注解的属性
    AnnotationAttributes role = attributesFor(metadata, Role.class);  
    if (role != null) {  
        abd.setRole(role.getNumber("value").intValue());  
    }  
    // 获取注解元数据的@Description注解的属性
    AnnotationAttributes description = attributesFor(metadata, Description.class);  
    if (description != null) {  
        abd.setDescription(description.getString("value"));  
    }  
}
```