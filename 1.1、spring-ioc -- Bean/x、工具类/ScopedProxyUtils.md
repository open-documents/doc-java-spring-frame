
# createScopedProxy()

```java
public static BeanDefinitionHolder createScopedProxy(BeanDefinitionHolder definition, BeanDefinitionRegistry registry, boolean proxyTargetClass) {  
    String originalBeanName = definition.getBeanName();  
    BeanDefinition targetDefinition = definition.getBeanDefinition();  
    String targetBeanName = getTargetBeanName(originalBeanName);  
    RootBeanDefinition proxyDefinition = new RootBeanDefinition(ScopedProxyFactoryBean.class);  
    proxyDefinition.setDecoratedDefinition(new BeanDefinitionHolder(targetDefinition, targetBeanName));  
    proxyDefinition.setOriginatingBeanDefinition(targetDefinition);  
    proxyDefinition.setSource(definition.getSource());  
    proxyDefinition.setRole(targetDefinition.getRole());  
    proxyDefinition.getPropertyValues().add("targetBeanName", targetBeanName);  
    if (proxyTargetClass) {  
        targetDefinition.setAttribute(AutoProxyUtils.PRESERVE_TARGET_CLASS_ATTRIBUTE, Boolean.TRUE);  
    } else {  
        proxyDefinition.getPropertyValues().add("proxyTargetClass", Boolean.FALSE);  
    }  
  
    proxyDefinition.setAutowireCandidate(targetDefinition.isAutowireCandidate());  
    proxyDefinition.setPrimary(targetDefinition.isPrimary());  
    if (targetDefinition instanceof AbstractBeanDefinition) {  
        proxyDefinition.copyQualifiersFrom((AbstractBeanDefinition)targetDefinition);  
    }  
  
    targetDefinition.setAutowireCandidate(false);  
    targetDefinition.setPrimary(false);  
    registry.registerBeanDefinition(targetBeanName, targetDefinition);  
    return new BeanDefinitionHolder(proxyDefinition, originalBeanName, definition.getAliases());  
}
```