
`AnnotationScanner` 是 Spring 注解包使用的一个内部类，它根据 `SearchStrategy` 指定的搜索策略对 `AnnotatedElement` 的层级结构进行搜索，并且使用 `AnnotationsProcessor` 在获得注解后进行回调。

-- --
`AnnotationScanner` 只提供了唯一允许包内调用的静态方法 `scan`。
# scan()方法

scan()方法在TypeMappedAnnotations.scan()方法中被调用。
```java
// class: TypeMappedAnnotations
AnnotationsScanner.scan(criteria, this.element, this.searchStrategy, processor);
```

```java

static <C, R> R scan(C context, AnnotatedElement source, SearchStrategy searchStrategy,  
      AnnotationsProcessor<C, R> processor) {  
    R result = process(context, source, searchStrategy, processor);  
    return processor.finish(result);  
}
```
根据 `AnnotatedElement` 类型的不同，又区分 `Method`、`Class` 以及其他 `AnnotatedElement` ，这三种对象会有不同的扫描方式。 `AnnotationScanner` 对这三者的扫描方式大同小异，基本都是没层级结构就直接返回，有层级结构就通过反射遍历按深度优先扫描层级结构。
```java

@Nullable  
private static <C, R> R process(C context, AnnotatedElement source,  
      SearchStrategy searchStrategy, AnnotationsProcessor<C, R> processor) {  
    if (source instanceof Class) {  
        return processClass(context, (Class<?>) source, searchStrategy, processor);  
    }  
    if (source instanceof Method) {  
        return processMethod(context, (Method) source, searchStrategy, processor);  
    }  
    return processElement(context, source, processor);  
}
```

```java
@Nullable  
private static <C, R> R processClass(C context, Class<?> source,  
      SearchStrategy searchStrategy, AnnotationsProcessor<C, R> processor) {  
    switch (searchStrategy) {  
        case DIRECT:  
	        return processElement(context, source, processor);  
	    case INHERITED_ANNOTATIONS:  
	        return processClassInheritedAnnotations(context, source, searchStrategy, processor);  
        case SUPERCLASS:  
            return processClassHierarchy(context, source, processor, false, false);  
		case TYPE_HIERARCHY:  
            return processClassHierarchy(context, source, processor, true, false);  
	    case TYPE_HIERARCHY_AND_ENCLOSING_CLASSES:  
            return processClassHierarchy(context, source, processor, true, true);  
    }  
    throw new IllegalStateException("Unsupported search strategy " + searchStrategy);  
}
```
Spring 针对方法的扫描制定了比较严格的标准，假设扫描的原始方法称为 `A`，则被扫描的方法 `B`，要允许获得 `B` 上的注解 ，则必须满足如下规则：
-   `B` 不能是桥接方法；
-   `A` 不能是私有方法；
-   `A` 和 `B` 的名称、参数类型、数量皆必须相等 ==> `hasSameParameterTypes` / `isOverride`；
-   若 `A` 和 `B` 参数有泛型，则要求泛型也一致 ==> `hasSameGenericTypeParameters()`；
```java
private static <C, R> R processMethod(C context, Method source,  
      SearchStrategy searchStrategy, AnnotationsProcessor<C, R> processor) {  
    switch (searchStrategy) {  
        case DIRECT:  
        case INHERITED_ANNOTATIONS:  
		    return processMethodInheritedAnnotations(context, source, processor);  
        case SUPERCLASS:  
	        return processMethodHierarchy(context, new int[] {0}, source.getDeclaringClass(),      processor, source, false);  
        case TYPE_HIERARCHY:  
        case TYPE_HIERARCHY_AND_ENCLOSING_CLASSES:  
	        return processMethodHierarchy(context, new int[] {0}, source.getDeclaringClass(),  
processor, source, true);  
     }  
     throw new IllegalStateException("Unsupported search strategy " + searchStrategy);  
}
```

-- --
# processElement()

根据SearchStrategy的不同，主要方法逻辑也不同。下面依次介绍不同的处理逻辑。

1）如果SearchStrategy是DIRECT或者SearchStrategy是INHERITED_ANNOTATIONS但是source不存在层级结构，都会执行下面的方法。
```java
// context: org.springframework.context.annotation.Configuration(String)
// AnnotatedElement: example.MyConfiguration(Class)
// processor: TypeMappedAnnotations$MergedAnnotationFinder
@Nullable  
private static <C, R> R processElement(C context, AnnotatedElement source,  
      AnnotationsProcessor<C, R> processor) {  
    try {  
	    // return TypeMappedAnnotations.result(属性)
        R result = processor.doWithAggregate(context, 0);  
        return (result != null ? result : processor.doWithAnnotations(  
            context, 0, source, getDeclaredAnnotations(source, false)));  
    }  
    catch (Throwable ex) {...}  
    return null;  
}
```

```java
@Nullable  
private static <C, R> R processClassInheritedAnnotations(C context, Class<?> source,  
      SearchStrategy searchStrategy, AnnotationsProcessor<C, R> processor) {  
    try {  
	    // 判断是否不存在层级关系
	    // 对于类的判断:
	    // 如果SearchStrategy不是TYPE_HIERARCHY_AND_ENCLOSING_CLASSES,那么没有父类并且没有实现任何接口,即不存在层级关系。
	    // 如果SearchStrategy是TYPE_HIERARCHY_AND_ENCLOSING_CLASSES,那么除了没有父类并且没有实现任何接口,还需要不是内部类,才不存在层级关系。
	    // 对于方法的判断:
	    // 待补充
        if (isWithoutHierarchy(source, searchStrategy)) {  
            return processElement(context, source, processor);  
        }  
        Annotation[] relevant = null;  
        int remaining = Integer.MAX_VALUE;  
        int aggregateIndex = 0;  
        Class<?> root = source;  
        while (source != null && source != Object.class && remaining > 0 &&  
            !hasPlainJavaAnnotationsOnly(source)) {  
         R result = processor.doWithAggregate(context, aggregateIndex);  
         if (result != null) {  
            return result;  
         }  
         Annotation[] declaredAnnotations = getDeclaredAnnotations(source, true);  
         if (relevant == null && declaredAnnotations.length > 0) {  
            relevant = root.getAnnotations();  
            remaining = relevant.length;  
         }  
         for (int i = 0; i < declaredAnnotations.length; i++) {  
            if (declaredAnnotations[i] != null) {  
               boolean isRelevant = false;  
               for (int relevantIndex = 0; relevantIndex < relevant.length; relevantIndex++) {  
                  if (relevant[relevantIndex] != null &&  
                        declaredAnnotations[i].annotationType() == relevant[relevantIndex].annotationType()) {  
                     isRelevant = true;  
                     relevant[relevantIndex] = null;  
                     remaining--;  
                     break;                  }  
               }  
               if (!isRelevant) {  
                  declaredAnnotations[i] = null;  
               }  
            }  
         }  
         result = processor.doWithAnnotations(context, aggregateIndex, source, declaredAnnotations);  
         if (result != null) {  
            return result;  
         }  
         source = source.getSuperclass();  
         aggregateIndex++;  
      }  
   }  
   catch (Throwable ex) {...}  
   return null;  
}
```

# 获取注解

```java
@Nullable  
static <A extends Annotation> A getDeclaredAnnotation(AnnotatedElement source, Class<A> annotationType) {  
   Annotation[] annotations = getDeclaredAnnotations(source, false);  
   for (Annotation annotation : annotations) {  
      if (annotation != null && annotationType == annotation.annotationType()) {  
         return (A) annotation;  
      }  
   }  
   return null;  
}


static Annotation[] getDeclaredAnnotations(AnnotatedElement source, boolean defensive) {  
    boolean cached = false;  
    Annotation[] annotations = declaredAnnotationCache.get(source);  
    if (annotations != null) {  
        cached = true;  
    }  
    else {  
        annotations = source.getDeclaredAnnotations();  
        if (annotations.length != 0) {  
            boolean allIgnored = true;  
	        for (int i = 0; i < annotations.length; i++) {  
                Annotation annotation = annotations[i];
				// 用AnnotationFilter.PLAIN过滤器来过滤
				// 
				if (isIgnorable(annotation.annotationType()) ||  
!AttributeMethods.forAnnotationType(annotation.annotationType()).isValid(annotation)) {  
	                annotations[i] = null;  
                }  
		        else {  
		            allIgnored = false;  
                }  
            }   
            annotations = (allIgnored ? NO_ANNOTATIONS : annotations);  
            if (source instanceof Class || source instanceof Member) {  
                declaredAnnotationCache.put(source, annotations);  
                cached = true;  
            }  
        }   
    }  
    if (!defensive || annotations.length == 0 || !cached) {  
	    return annotations;  
    }  
    return annotations.clone();  
}
```