
# 默认的

```java
public Object getEarlyBeanReference(Object bean, String beanName) {  
   Object cacheKey = getCacheKey(bean.getClass(), beanName);  
   this.earlyProxyReferences.put(cacheKey, bean);  
   return wrapIfNecessary(bean, beanName, cacheKey);  
}


```

```java
protected Object getCacheKey(Class<?> beanClass, @Nullable String beanName) {  
   if (StringUtils.hasLength(beanName)) {  
      return (FactoryBean.class.isAssignableFrom(beanClass) ?  
            BeanFactory.FACTORY_BEAN_PREFIX + beanName : beanName);  
   }  
   else {  
      return beanClass;  
   }  
}
```

```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {  
   if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {  
      return bean;  
   }  
   if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {  
      return bean;  
   }  
   if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {  
      this.advisedBeans.put(cacheKey, Boolean.FALSE);  
      return bean;  
   }  
  
   // Create proxy if we have advice.  
   // 抽象方法
   Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);  
   if (specificInterceptors != DO_NOT_PROXY) {  
      this.advisedBeans.put(cacheKey, Boolean.TRUE);  
      Object proxy = createProxy(  
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));  
      this.proxyTypes.put(cacheKey, proxy.getClass());  
      return proxy;  
   }  
  
   this.advisedBeans.put(cacheKey, Boolean.FALSE);  
   return bean;  
}
```
