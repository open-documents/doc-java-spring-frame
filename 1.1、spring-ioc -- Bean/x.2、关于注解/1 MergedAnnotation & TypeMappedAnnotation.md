
MergedAnnotation接口顾名思义，表示一个合并而成的注解，MergedAnnotation 通常与一个注解对象一对一，但是它的属性可能来自于子注解或者元注解，甚至是同一个注解中通过 @AliasFor 绑定其他属性。


TypeMappedAnnotation是MergedAnnotation接口的一个实现类。

# 1、创建TypeMappedAnnotation

首先需要明确创建TypeMappedAnnotation实例需要AnnotationTypeMapping实例。
```java
@Nullable  
static <A extends Annotation> TypeMappedAnnotation<A> createIfPossible(  
      AnnotationTypeMapping mapping, @Nullable Object source, Annotation annotation,  
      int aggregateIndex, IntrospectionFailureLogger logger) {  
  
    return createIfPossible(mapping, 
	    source, 
	    annotation,  
        AnnotationUtils::invokeAnnotationMethod, 
        aggregateIndex, 
        logger);  
}
@Nullable  
private static <A extends Annotation> TypeMappedAnnotation<A> createIfPossible(  
      AnnotationTypeMapping mapping, @Nullable Object source, @Nullable Object rootAttribute,  
      ValueExtractor valueExtractor, int aggregateIndex, IntrospectionFailureLogger logger) {  
    try {  
        return new TypeMappedAnnotation<>(mapping, null, source, rootAttribute,  
            valueExtractor, aggregateIndex);  
    }  
    catch (Exception ex) {
	    ...
		return null;
    }  
}
```

# 2、构造函数

可以看得出，`TypeMappedAnnotation` 基本可以认为是 `AnnotationTypeMapping` 的包装类，它以一个 `AnnotationTypeMapping` 实例作为数据源，从而提供一些关于映射后的属性的相关功能。

它本身并没有很多特殊的逻辑，我们仅需要关注通过它合成注解的代理对象，以及后续调用代理对象时，是如何从映射过的属性获取值的。

```java
private TypeMappedAnnotation(AnnotationTypeMapping mapping, @Nullable ClassLoader classLoader,  
      @Nullable Object source, @Nullable Object rootAttributes, ValueExtractor valueExtractor,  
      int aggregateIndex, @Nullable int[] resolvedRootMirrors) {  
    // 当前合并注解对应的AnnotationTypeMapping
    this.mapping = mapping;  
    this.classLoader = classLoader;  
    this.source = source;  
    this.rootAttributes = rootAttributes;  
    // 通过属性方法对象获得属性值的方法，一般是ReflectionUtils::invokeMethod
    this.valueExtractor = valueExtractor;  
    this.aggregateIndex = aggregateIndex;  
    this.useMergedValues = true;  
    this.attributeFilter = null;
    this.resolvedRootMirrors = (resolvedRootMirrors != null ? resolvedRootMirrors :  
	    mapping.getRoot().getMirrorSets().resolve(source, rootAttributes, this.valueExtractor));  
    this.resolvedMirrors = (getDistance() == 0 ? this.resolvedRootMirrors :  
	    mapping.getMirrorSets().resolve(source, this, this::getValueForMirrorResolution));  
}
```


```java
@Override  
public <T extends Map<String, Object>> T asMap(Function<MergedAnnotation<?>, T> factory, Adapt... adaptations) {  
    T map = factory.apply(this);  
    // assert
    AttributeMethods attributes = this.mapping.getAttributes();  
    for (int i = 0; i < attributes.size(); i++) {  
        Method attribute = attributes.get(i);  
        // isFiltered():
        // if (this.attributeFilter != null) {  
        //   return !this.attributeFilter.test(attributeName);  
        // }  
        // return false;
	    Object value = (isFiltered(attribute.getName()) ? null :  
            getValue(i, getTypeForMapOptions(attribute, adaptations)));  
        if (value != null) {  
		    map.put(attribute.getName(),  
               adaptValueForMapOptions(attribute, value, map.getClass(), factory, adaptations));  
      }  
   }  
   return map;  
}
// -----------------------------getTypeForMapOptions()----------------------------------------
private Class<?> getTypeForMapOptions(Method attribute, Adapt[] adaptations) {  
    Class<?> attributeType = attribute.getReturnType();  
    Class<?> componentType = (attributeType.isArray() ? attributeType.getComponentType() : attributeType);  
    if (Adapt.CLASS_TO_STRING.isIn(adaptations) && componentType == Class.class) {  
        return (attributeType.isArray() ? String[].class : String.class);  
    }  
    return Object.class;  
}
// -----------------------------getValue()----------------------------------------------------
private <T> T getValue(int attributeIndex, Class<T> type) {  
    Method attribute = this.mapping.getAttributes().get(attributeIndex);  
    Object value = getValue(attributeIndex, true, false);  
    if (value == null) {  
        value = attribute.getDefaultValue();  
    }  
    return adapt(attribute, value, type);  
}  
private Object getValue(int attributeIndex, boolean useConventionMapping, boolean forMirrorResolution) {  
    AnnotationTypeMapping mapping = this.mapping;  
    if (this.useMergedValues) {  
        int mappedIndex = this.mapping.getAliasMapping(attributeIndex);  
        if (mappedIndex == -1 && useConventionMapping) {  
		    mappedIndex = this.mapping.getConventionMapping(attributeIndex);  
        }  
        if (mappedIndex != -1) {  
		    mapping = mapping.getRoot();  
            attributeIndex = mappedIndex;  
        }  
    }  
    if (!forMirrorResolution) {  
        attributeIndex =  
            (mapping.getDistance() != 0 ? this.resolvedMirrors : this.resolvedRootMirrors)[attributeIndex];  
    }  
    if (attributeIndex == -1) {  
        return null;  
    }  
    if (mapping.getDistance() == 0) {  
        Method attribute = mapping.getAttributes().get(attributeIndex);  
        Object result = this.valueExtractor.extract(attribute, this.rootAttributes);  
        return (result != null ? result : attribute.getDefaultValue());  
    }  
    return getValueFromMetaAnnotation(attributeIndex, forMirrorResolution);  
}
```