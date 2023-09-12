
参考文档：
- https://docs.spring.io/spring-framework/reference/core/aop-api/pfb.html
- https://docs.spring.io/spring-framework/reference/core/aop-api/prog.html







```java
public class ProxyFactoryBean extends ProxyCreatorSupport  
      implements FactoryBean<Object>, BeanClassLoaderAware, BeanFactoryAware {
```

```java
public Object getObject() throws BeansException {  
    initializeAdvisorChain();  
    if (isSingleton()) {  
        return getSingletonInstance();  
    }  
    else {  
        if (this.targetName == null) {  
            logger.info(...);  
        }  
        return newPrototypeInstance();  
    }  
}
```

# 初始化Advisor链

```java
/* --------------------------------------- ProxyFactoryBean --------------------------------------- */
private synchronized void initializeAdvisorChain() throws AopConfigException, BeansException {  
	// 1. ProxyFactoryBean.this.advisorChainInitialized: Advisor链是否已经初始化, 默认初值 false
	// 2. ProxyFactoryBean.this.interceptorNames
	//  - 类型 String[]  
	//  - 仅支持set注入
    if (!this.advisorChainInitialized && !ObjectUtils.isEmpty(this.interceptorNames)) {  
        if (this.beanFactory == null) {  
            throw new IllegalStateException(...);  
        }  
  
        // Globals can't be last unless we specified a targetSource using the property...  
        // GLOBAL_SUFFIX = "*"
        if (this.interceptorNames[this.interceptorNames.length - 1].endsWith(GLOBAL_SUFFIX) 
	        && this.targetName == null
            && this.targetSource == EMPTY_TARGET_SOURCE) {  
            throw new AopConfigException("Target required after globals");  
        }  
  
        // Materialize interceptor chain from bean names.  
        for (String name : this.interceptorNames) {  
            if (name.endsWith(GLOBAL_SUFFIX)) {  
	            if (!(this.beanFactory instanceof ListableBeanFactory lbf)) {  
	                throw new AopConfigException(...);  
	            }  
	            addGlobalAdvisors(lbf, name.substring(0, name.length() - GLOBAL_SUFFIX.length()));  
            }  
            else {  
	            // If we get here, we need to add a named interceptor.  
	            // We must check if it's a singleton or prototype.            
	            Object advice;  
	            if (this.singleton || this.beanFactory.isSingleton(name)) {  
	                // Add the real Advisor/Advice to the chain.  
	                advice = this.beanFactory.getBean(name);  
                }  
	            else {  
	                // It's a prototype Advice or Advisor: replace with a prototype.  
	                // Avoid unnecessary creation of prototype bean just for advisor chain initialization. 
	                advice = new PrototypePlaceholderAdvisor(name);  
                }  
                addAdvisorOnChainCreation(advice);  
            }  
        }  
        this.advisorChainInitialized = true;  
    }  
}
```

# 创建代理对象

```java
private synchronized Object newPrototypeInstance() {  
    // In the case of a prototype, we need to give the proxy  
    // an independent instance of the configuration.   
    // In this case, no proxy will have an instance of this object's configuration,   
    // but will have an independent copy.   
    ProxyCreatorSupport copy = new ProxyCreatorSupport(getAopProxyFactory());  
  
    // The copy needs a fresh advisor chain, and a fresh TargetSource.  
    TargetSource targetSource = freshTargetSource();  
    copy.copyConfigurationFrom(this, targetSource, freshAdvisorChain());  
   if (this.autodetectInterfaces && getProxiedInterfaces().length == 0 && !isProxyTargetClass()) {  
      // Rely on AOP infrastructure to tell us what interfaces to proxy.  
      Class<?> targetClass = targetSource.getTargetClass();  
      if (targetClass != null) {  
         copy.setInterfaces(ClassUtils.getAllInterfacesForClass(targetClass, this.proxyClassLoader));  
      }  
   }  
   copy.setFrozen(this.freezeProxy);  
  
   return getProxy(copy.createAopProxy());  
}
```