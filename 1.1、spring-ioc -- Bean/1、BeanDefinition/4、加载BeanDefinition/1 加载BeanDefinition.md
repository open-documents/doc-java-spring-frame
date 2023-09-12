
# 1、loadBeanDefinitions()
```java
/* -------------------------- ConfigurationClassBeanDefinitionReader --------------------------
 */
public void loadBeanDefinitions(Set<ConfigurationClass> configurationModel) {  
	TrackedConditionEvaluator trackedConditionEvaluator = new TrackedConditionEvaluator();  
    for (ConfigurationClass configClass : configurationModel) {  
        loadBeanDefinitionsForConfigurationClass(configClass, trackedConditionEvaluator);  
    }  
}
private void loadBeanDefinitionsForConfigurationClass(ConfigurationClass configClass, TrackedConditionEvaluator trackedConditionEvaluator) {  
   if (trackedConditionEvaluator.shouldSkip(configClass)) {  
      String beanName = configClass.getBeanName();  
      if (StringUtils.hasLength(beanName) && this.registry.containsBeanDefinition(beanName)) {  
         this.registry.removeBeanDefinition(beanName);  
      }  
      this.importRegistry.removeImportingClass(configClass.getMetadata().getClassName());  
      return;   
      }  
  
    if (configClass.isImported()) {  
	    // 将配置类自身注册为BeanDefinition
        registerBeanDefinitionForImportedConfigurationClass(configClass);  
    }  
    // BeanMethod中封装了MethodMetadata和@Bean方法所属的配置类
    for (BeanMethod beanMethod : configClass.getBeanMethods()) {  
	    // 将配置类中的@Bean方法注册为BeanDefitinion
        loadBeanDefinitionsForBeanMethod(beanMethod);  
    }  
   
    loadBeanDefinitionsFromImportedResources(configClass.getImportedResources());  
    // 调用@Import导入的ImportBeanDefinitionRegistrar实现类来注册
    loadBeanDefinitionsFromRegistrars(configClass.getImportBeanDefinitionRegistrars());  
}

```

# 相关--registerBeanDefinitionForImportedConfigurationClass
```java
/* class: ConfigurationClassBeanDefinitionReader
 * description: 将配置类自身注册为BeanDefinition
 */
private void registerBeanDefinitionForImportedConfigurationClass(ConfigurationClass configClass) {  
   AnnotationMetadata metadata = configClass.getMetadata();  
   AnnotatedGenericBeanDefinition configBeanDef = new AnnotatedGenericBeanDefinition(metadata);  
  
   ScopeMetadata scopeMetadata = scopeMetadataResolver.resolveScopeMetadata(configBeanDef);  
   configBeanDef.setScope(scopeMetadata.getScopeName());  
   String configBeanName = this.importBeanNameGenerator.generateBeanName(configBeanDef, this.registry);  
   AnnotationConfigUtils.processCommonDefinitionAnnotations(configBeanDef, metadata);  
  
   BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(configBeanDef, configBeanName);  
   definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);  
   this.registry.registerBeanDefinition(definitionHolder.getBeanName(), definitionHolder.getBeanDefinition());  
   configClass.setBeanName(configBeanName);  
  
   if (logger.isTraceEnabled()) {  
      logger.trace("Registered bean definition for imported class '" + configBeanName + "'");  
   }  
}
```


# 相关
```java
private void loadBeanDefinitionsFromImportedResources(Map<String, Class<? extends BeanDefinitionReader>> importedResources) {  
  
   Map<Class<?>, BeanDefinitionReader> readerInstanceCache = new HashMap<>();  
  
   importedResources.forEach((resource, readerClass) -> {  
      // Default reader selection necessary?  
      if (BeanDefinitionReader.class == readerClass) {  
         if (StringUtils.endsWithIgnoreCase(resource, ".groovy")) {  
            // When clearly asking for Groovy, that's what they'll get...  
            readerClass = GroovyBeanDefinitionReader.class;  
         }  
         else if (shouldIgnoreXml) {  
            throw new UnsupportedOperationException("XML support disabled");  
         }  
         else {  
            // Primarily ".xml" files but for any other extension as well  
            readerClass = XmlBeanDefinitionReader.class;  
         }  
      }  
  
      BeanDefinitionReader reader = readerInstanceCache.get(readerClass);  
      if (reader == null) {  
         try {  
            // Instantiate the specified BeanDefinitionReader  
            reader = readerClass.getConstructor(BeanDefinitionRegistry.class).newInstance(this.registry);  
            // Delegate the current ResourceLoader to it if possible  
            if (reader instanceof AbstractBeanDefinitionReader) {  
               AbstractBeanDefinitionReader abdr = ((AbstractBeanDefinitionReader) reader);  
               abdr.setResourceLoader(this.resourceLoader);  
               abdr.setEnvironment(this.environment);  
            }  
            readerInstanceCache.put(readerClass, reader);  
         }  
         catch (Throwable ex) {  
            throw new IllegalStateException(  
                  "Could not instantiate BeanDefinitionReader class [" + readerClass.getName() + "]");  
         }  
      }  
  
      // TODO SPR-6310: qualify relative path locations as done in AbstractContextLoader.modifyLocations  
      reader.loadBeanDefinitions(resource);  
   });  
}
```
# 相关类--ConfigurationClassBeanDefinition
```java
public ConfigurationClassBeanDefinition(  
      ConfigurationClass configClass, MethodMetadata beanMethodMetadata, String derivedBeanName) {  
  
   this.annotationMetadata = configClass.getMetadata();  
   this.factoryMethodMetadata = beanMethodMetadata;  
   this.derivedBeanName = derivedBeanName;  
   setResource(configClass.getResource());  
   setLenientConstructorResolution(false);  
}
```