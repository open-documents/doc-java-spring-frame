
ScannedGenericBeanDefinition只提供一个构造函数，这个构造函数很简单。

因为MetadataReader的构造函数中已经通过ClassReader读取Resource来获得AnnotationMetadata，因此在ScannedGenericBeanDefinition构造函数中只需要简单设置AnnotationMetadata属性即可。
```java
public ScannedGenericBeanDefinition(MetadataReader metadataReader) {  
	... 
	// MetadataReader构造函数中已经获取了AnnotationMetadata,并通过AnnotationMetadata属性来保存
    this.metadata = metadataReader.getAnnotationMetadata();  
    setBeanClassName(this.metadata.getClassName());  
    setResource(metadataReader.getResource());  
}
```
