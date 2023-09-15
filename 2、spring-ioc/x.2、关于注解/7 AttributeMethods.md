AttributeMethods代表一个注解的属性集合。

# 1、获取AttributeMethods

AttributeMethods提供了forAnnotationType()静态方法来根据一个注解类型创建该注解类型的属性集合。
```java
static AttributeMethods forAnnotationType(@Nullable Class<? extends Annotation> annotationType) {  
    if (annotationType == null) {  
        return NONE;  
    }  
    return cache.computeIfAbsent(annotationType, AttributeMethods::compute);  
}

private static AttributeMethods compute(Class<? extends Annotation> annotationType) {  
    Method[] methods = annotationType.getDeclaredMethods();  
    int size = methods.length;
    for (int i = 0; i < methods.length; i++) {  
	    // return (method.getParameterCount() == 0 && method.getReturnType() != void.class);
        if (!isAttributeMethod(methods[i])) {  
		    methods[i] = null;  
            size--;  
        }  
     }  
    if (size == 0) {  
        return NONE;  
    }  
    ... // assert
    Method[] attributeMethods = Arrays.copyOf(methods, size);  
    return new AttributeMethods(annotationType, attributeMethods);  
}
```

# 2、构造函数

提供的静态方法来创建实例，最终还是会调用构造函数来创建实例。
```java
private AttributeMethods(@Nullable Class<? extends Annotation> annotationType, Method[] attributeMethods) {  
    this.annotationType = annotationType;  
    this.attributeMethods = attributeMethods;  
    this.canThrowTypeNotPresentException = new boolean[attributeMethods.length];  
    boolean foundDefaultValueMethod = false;  
    boolean foundNestedAnnotation = false;  
    for (int i = 0; i < attributeMethods.length; i++) {  
        Method method = this.attributeMethods[i];  
        Class<?> type = method.getReturnType();  
        if (!foundDefaultValueMethod && (method.getDefaultValue() != null)) {  
            foundDefaultValueMethod = true;
        }  
        if (!foundNestedAnnotation && (type.isAnnotation() || (type.isArray() && type.getComponentType().isAnnotation()))) {  
            foundNestedAnnotation = true;  
        }  
        ReflectionUtils.makeAccessible(method);  
        this.canThrowTypeNotPresentException[i] = (type == Class.class || type == Class[].class || type.isEnum());  
    }  
    this.hasDefaultValueMethod = foundDefaultValueMethod;  
    this.hasNestedAnnotation = foundNestedAnnotation;  
}
```