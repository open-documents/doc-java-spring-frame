

对@Bean方法的影响：
决定配置类的@Bean方法解析后是否继续解析为BeanDefinition。
```java
/* Class: ConfigurationClassBeanDefinitionReader
 * Parameter:
 *   - beanMethod: BeanMethod类型封装了@Bean方法的MethodMetadata和所属的配置类
 */
private void loadBeanDefinitionsForBeanMethod(BeanMethod beanMethod) {  
	...
    // Do we need to mark the bean as skipped by its condition?  
    // metadata: @Bean方法的MethodMetadata
    if (this.conditionEvaluator.shouldSkip(metadata, ConfigurationPhase.REGISTER_BEAN)) {  
        configClass.skippedBeanMethods.add(methodName);  
        return;   
    }  
    if (configClass.skippedBeanMethods.contains(methodName)) {  
        return;  
    }  
}
```


- @ComponentScan：扫描到的不符合条件的




## 1、涉及的类--TrackedConditionEvaluator

下面是该类的全部定义。
```java
private final Map<ConfigurationClass, Boolean> skipped = new HashMap<>();  
  
public boolean shouldSkip(ConfigurationClass configClass) {  
    Boolean skip = this.skipped.get(configClass);  
    if (skip == null) {  
	    // 这个类是由其它类使用@Import导入
        if (configClass.isImported()) {  
            boolean allSkipped = true;  
            for (ConfigurationClass importedBy : configClass.getImportedBy()) {  
	            if (!shouldSkip(importedBy)) {  
	                allSkipped = false;  
	                break;
				}  
            }  
            if (allSkipped) {  
            // The config classes that imported this one were all skipped, therefore we are skipped...  
	            skip = true;  
            }  
        }  
		if (skip == null) {  
            skip = conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN);  
        }  
        this.skipped.put(configClass, skip);  
    }  
    return skip;  
}
```




# ConditionEvaluator#shouldSkip()
```java
public boolean shouldSkip(@Nullable AnnotatedTypeMetadata metadata, @Nullable ConfigurationPhase phase) {  
    if (metadata == null || !metadata.isAnnotated(Conditional.class.getName())) {  
        return false;  
    }  
  
   if (phase == null) {  
      if (metadata instanceof AnnotationMetadata &&  
            ConfigurationClassUtils.isConfigurationCandidate((AnnotationMetadata) metadata)) {  
         return shouldSkip(metadata, ConfigurationPhase.PARSE_CONFIGURATION);  
      }  
      return shouldSkip(metadata, ConfigurationPhase.REGISTER_BEAN);  
   }  
  
   List<Condition> conditions = new ArrayList<>();  
   for (String[] conditionClasses : getConditionClasses(metadata)) {  
      for (String conditionClass : conditionClasses) {  
         Condition condition = getCondition(conditionClass, this.context.getClassLoader());  
         conditions.add(condition);  
      }  
   }  
  
   AnnotationAwareOrderComparator.sort(conditions);  
  
   for (Condition condition : conditions) {  
      ConfigurationPhase requiredPhase = null;  
      if (condition instanceof ConfigurationCondition) {  
         requiredPhase = ((ConfigurationCondition) condition).getConfigurationPhase();  
      }  
      if ((requiredPhase == null || requiredPhase == phase) && !condition.matches(this.context, metadata)) {  
         return true;  
      }  
   }  
  
   return false;  
}
```

## 相关方法--getConditionClasses()
```java
private List<String[]> getConditionClasses(AnnotatedTypeMetadata metadata) {  
   MultiValueMap<String, Object> attributes = metadata.getAllAnnotationAttributes(Conditional.class.getName(), true);  
   Object values = (attributes != null ? attributes.get("value") : null);  
   return (List<String[]>) (values != null ? values : Collections.emptyList());  
}
```
## 相关方法--getCondition()