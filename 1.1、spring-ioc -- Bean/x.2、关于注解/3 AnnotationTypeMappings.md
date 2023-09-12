
`AnnotationTypeMappings` 用于表示某一个注解类上全部元注解，对应的还有一个 `AnnotationTypeMapping` ，它表示一个具体的元注解对象。

`AnnotationTypeMappings` 与 `MergedAnnotations` 的设计思路一样，它表示一组 `AnnotationTypeMapping` 对象的聚合状态，同时用于提供对 `AnnotationTypeMapping` 的创建和搜索等功能。

某种程度上来说，`AnnotationTypeMappings` 其实就是一个注解类的元注解结合体。

# 1、AnnotationTypeMappings的创建

`AnnotationTypeMappings` 提供3个静态方法forAnnotationType()，用于根据一个注解类型创建 `AnnotationTypeMappings` 对象。
```java
static AnnotationTypeMappings forAnnotationType(Class<? extends Annotation> annotationType) {  
    return forAnnotationType(annotationType, new HashSet<>());  
}
static AnnotationTypeMappings forAnnotationType(Class<? extends Annotation> annotationType,  
      Set<Class<? extends Annotation>> visitedAnnotationTypes) {  
    return forAnnotationType(annotationType, RepeatableContainers.standardRepeatables(),  
						     AnnotationFilter.PLAIN, visitedAnnotationTypes);  
}
// annotationType: interface org.springframework.context.annotation.Configuration
// repeatableContainers: RepeatableContainers$NoRepeatableContainers
// annotationFilter: Packages annotation filter: java.lang.,org.springframework.lang.
static AnnotationTypeMappings forAnnotationType(Class<? extends Annotation> annotationType,  
      RepeatableContainers repeatableContainers, AnnotationFilter annotationFilter) {  
    return forAnnotationType(annotationType, repeatableContainers, annotationFilter,
						     new HashSet<>());  
}
```
具体的创建过程如下：
```java
private static AnnotationTypeMappings forAnnotationType(
	Class<? extends Annotation> annotationType,  
    RepeatableContainers repeatableContainers, 
    AnnotationFilter annotationFilter,  
    Set<Class<? extends Annotation>> visitedAnnotationTypes) {  
  
    if (repeatableContainers == RepeatableContainers.standardRepeatables()) {  
	    // static ConcurrentReferenceHashMap<AnnotationFilter, Cache> standardRepeatablesCache
        return standardRepeatablesCache.computeIfAbsent(
        annotationFilter,  
        key -> new Cache(repeatableContainers, key)).get(annotationType, visitedAnnotationTypes);  
    }  
    // 如果注解类型是@Configuration这类不可重复注解,那么通过其它方法调用这个方法时传入的RepeatableContainers是RepeatableContainers.none()
    if (repeatableContainers == RepeatableContainers.none()) {  
	    // static ConcurrentReferenceHashMap<AnnotationFilter, Cache> noRepeatablesCache
        return noRepeatablesCache.computeIfAbsent(
        annotationFilter,  
        key -> new Cache(repeatableContainers, key)).get(annotationType, visitedAnnotationTypes);  
    }  
    return new AnnotationTypeMappings(repeatableContainers, annotationFilter, annotationType, visitedAnnotationTypes);  
}
```
# 2、构造函数

AnnotationTypeMappings的构造函数是私有的，创建AnnotationTypeMappings实例是通过forAnnotationType静态方法创建。
这个构造函数在Cache实例中不存在注解的Class对象对应的key时被使用。
```java
private AnnotationTypeMappings(RepeatableContainers repeatableContainers,  
      AnnotationFilter filter, Class<? extends Annotation> annotationType,  
      Set<Class<? extends Annotation>> visitedAnnotationTypes) {  
  
    this.repeatableContainers = repeatableContainers;  
    this.filter = filter;  
    this.mappings = new ArrayList<>();  // List<AnnotationTypeMapping> 
    addAllMappings(annotationType, visitedAnnotationTypes);  
    this.mappings.forEach(AnnotationTypeMapping::afterAllMappingsSet);  
}
// ----------------------------------------------------------------------------------------
private void addAllMappings(Class<? extends Annotation> annotationType,  
      Set<Class<? extends Annotation>> visitedAnnotationTypes) {  
    // 使用广度优先搜索遍历annotationType的元注解
    Deque<AnnotationTypeMapping> queue = new ArrayDeque<>();  
    addIfPossible(queue, null, annotationType, null, visitedAnnotationTypes);  
    while (!queue.isEmpty()) {  
        AnnotationTypeMapping mapping = queue.removeFirst();  
        this.mappings.add(mapping);  
        addMetaAnnotationsToQueue(queue, mapping);  
    }  
}
// ---------------------------addIfPossible()--------------------------------------------------
private void addIfPossible(
	Deque<AnnotationTypeMapping> queue,
	@Nullable AnnotationTypeMapping source, Class<? extends Annotation> annotationType, @Nullable Annotation ann, Set<Class<? extends Annotation>> visitedAnnotationTypes) {  
    try {  
        queue.addLast(new AnnotationTypeMapping(source, annotationType, ann, visitedAnnotationTypes));  
    }  
    catch (Exception ex) {...}  
    }  
}
// ---------------------------addMetaAnnotationsToQueue()---------------------------------------
private void addMetaAnnotationsToQueue(Deque<AnnotationTypeMapping> queue, AnnotationTypeMapping source) {  
   Annotation[] metaAnnotations = AnnotationsScanner.getDeclaredAnnotations(source.getAnnotationType(), false);  
   for (Annotation metaAnnotation : metaAnnotations) {  
      if (!isMappable(source, metaAnnotation)) {
         continue;  
      }  
      Annotation[] repeatedAnnotations = this.repeatableContainers.findRepeatedAnnotations(metaAnnotation);  
      if (repeatedAnnotations != null) {  
         for (Annotation repeatedAnnotation : repeatedAnnotations) {  
            if (!isMappable(source, repeatedAnnotation)) {  
               continue;  
            }  
            addIfPossible(queue, source, repeatedAnnotation);  
         }  
      }  
      else {  
         addIfPossible(queue, source, metaAnnotation);  
      }  
   }  
}
```


# 3、Cache

属性
```java
private final RepeatableContainers repeatableContainers;  
  
private final AnnotationFilter filter;  
  
private final Map<Class<? extends Annotation>, AnnotationTypeMappings> mappings;
```

```java

AnnotationTypeMappings get(Class<? extends Annotation> annotationType,  
      Set<Class<? extends Annotation>> visitedAnnotationTypes) {  
    return this.mappings.computeIfAbsent(
    annotationType, 
    key -> createMappings(key, visitedAnnotationTypes));  
}
// ----------------------------------------------------------------------------------------
// 重点逻辑在AnnotationTypeMappings的构造函数
private AnnotationTypeMappings createMappings(Class<? extends Annotation> annotationType,  
      Set<Class<? extends Annotation>> visitedAnnotationTypes) {  
    return new AnnotationTypeMappings(
	    this.repeatableContainers, 
	    this.filter, 
	    annotationType, 
	    visitedAnnotationTypes);  
}
```