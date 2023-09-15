
检查一个BeanDefinition是否是配置类，由ConfigurationClassUtils.checkConfigurationClassCandidate()完成。

```java
static boolean checkConfigurationClassCandidate(BeanDefinition beanDef, MetadataReaderFactory metadataReaderFactory) {  

    String className = beanDef.getBeanClassName();  
    if (className == null || beanDef.getFactoryMethodName() != null) {  
        return false;  
    }  

    AnnotationMetadata metadata;  
    if (beanDef instanceof AnnotatedBeanDefinition annotatedBd &&  
         className.equals(annotatedBd.getMetadata().getClassName())) {  
      // Can reuse the pre-parsed metadata from the given BeanDefinition...  
      metadata = annotatedBd.getMetadata();  
   }  
   else if (beanDef instanceof AbstractBeanDefinition abstractBd && abstractBd.hasBeanClass()) {  
      // Check already loaded Class if present...  
      // since we possibly can't even load the class file for this Class.      Class<?> beanClass = abstractBd.getBeanClass();  
      if (BeanFactoryPostProcessor.class.isAssignableFrom(beanClass) ||  
            BeanPostProcessor.class.isAssignableFrom(beanClass) ||  
            AopInfrastructureBean.class.isAssignableFrom(beanClass) ||  
            EventListenerFactory.class.isAssignableFrom(beanClass)) {  
         return false;  
      }  
      metadata = AnnotationMetadata.introspect(beanClass);  
   }  
   else {  
      try {  
         MetadataReader metadataReader = metadataReaderFactory.getMetadataReader(className);  
         metadata = metadataReader.getAnnotationMetadata();  
      }  
      catch (IOException ex) {  
         if (logger.isDebugEnabled()) {  
            logger.debug("Could not find class file for introspecting configuration annotations: " +  
                  className, ex);  
         }  
         return false;  
      }  
   }  
  
   Map<String, Object> config = metadata.getAnnotationAttributes(Configuration.class.getName());  
   if (config != null && !Boolean.FALSE.equals(config.get("proxyBeanMethods"))) {  
      beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_FULL);  
   }  
   else if (config != null || isConfigurationCandidate(metadata)) {  
      beanDef.setAttribute(CONFIGURATION_CLASS_ATTRIBUTE, CONFIGURATION_CLASS_LITE);  
   }  
   else {  
      return false;  
   }  
  
   // It's a full or lite configuration candidate... Let's determine the order value, if any.  
   Integer order = getOrder(metadata);  
   if (order != null) {  
      beanDef.setAttribute(ORDER_ATTRIBUTE, order);  
   }  
  
   return true;  
}
```

```java
static boolean isConfigurationCandidate(AnnotationMetadata metadata) {  
   // Do not consider an interface or an annotation...  
   if (metadata.isInterface()) {  
      return false;  
   }  
  
   // Any of the typical annotations found?  
   for (String indicator : candidateIndicators) {  
      if (metadata.isAnnotated(indicator)) {  
         return true;  
      }  
   }  
  
   // Finally, let's look for @Bean methods...  
   return hasBeanMethods(metadata);  
}
```

```java
static boolean hasBeanMethods(AnnotationMetadata metadata) {  
   try {  
      return metadata.hasAnnotatedMethods(Bean.class.getName());  
   }  
   catch (Throwable ex) {  
      if (logger.isDebugEnabled()) {  
         logger.debug("Failed to introspect @Bean methods on class [" + metadata.getClassName() + "]: " + ex);  
      }  
      return false;  
   }  
}
```