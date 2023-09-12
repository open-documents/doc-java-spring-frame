
StandardAnnotationMetadata包含了一个类下面的内容：
- 标注在类上的所有注解的全限定名。
- 标注在类上的所有注解（全部包含在MergedAnnotations类型属性中）。

属性
```java
private final boolean nestedAnnotationsAsMap;  

@Nullable  
private Set<String> annotationTypes;
private final MergedAnnotations mergedAnnotations;  
```