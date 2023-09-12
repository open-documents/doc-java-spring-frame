
```java
/* ---------------------------------- ConditionEvaluator ---------------------------------- */
public boolean shouldSkip(@Nullable AnnotatedTypeMetadata metadata, @Nullable ConfigurationPhase phase) {  
    if (metadata == null || !metadata.isAnnotated(Conditional.class.getName())) {  
        return false;  
    }  
  
    if (phase == null) {  
        if (
	        metadata instanceof AnnotationMetadata annotationMetadata &&  
            ConfigurationClassUtils.isConfigurationCandidate(annotationMetadata)) {  
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
        if (condition instanceof ConfigurationCondition configurationCondition) {  
            requiredPhase = configurationCondition.getConfigurationPhase();  
        }  
        if ((requiredPhase == null || requiredPhase == phase) && !condition.matches(this.context, metadata)) {  
            return true;  
        }  
    }  
  
    return false;  
}
```