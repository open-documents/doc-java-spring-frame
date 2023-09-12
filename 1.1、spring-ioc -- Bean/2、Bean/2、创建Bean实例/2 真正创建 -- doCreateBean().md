
总体而言，逻辑分为部分：
- 


```java
/* --------------------------------- AbstractAutowireCapableBeanFactory ---------------------------------

*/
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {  
  
    // 实例化Bean
    BeanWrapper instanceWrapper = null;  
    if (mbd.isSingleton()) {  
	    // this.factoryBeanInstanceCache:
	    // - 类型为 ConcurrentMap<String, BeanWrapper>
	    // - 默认(初)值为 new ConcurrentHashMap<String, BeanWrapper>()
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);  
    }  
    if (instanceWrapper == null) {  
        instanceWrapper = createBeanInstance(beanName, mbd, args);  // 见 2.1节 创建实例
    }  
    Object bean = instanceWrapper.getWrappedInstance();  
    Class<?> beanType = instanceWrapper.getWrappedClass();  
    if (beanType != NullBean.class) {  
        mbd.resolvedTargetType = beanType;  
    }

    // Allow post-processors to modify the merged bean definition.  
    synchronized (mbd.postProcessingLock) {  
	    if (!mbd.postProcessed) {  
	        try {
	            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);  
            }  
		    catch (Throwable ex) {  
	            throw new BeanCreationException("Post-processing of merged bean definition failed");  
            }  
            mbd.markAsPostProcessed();  
        }  
    }  
  

	// 如果当前bean是单例并且支持循环依赖，且当前bean正在创建
	// 就通过往singletonFactories(三级缓存)添加一个objectFactory
	// 这样后期如果有其他bean依赖该bean,可以从singletonFactories获取到bean,解决循环依赖
    boolean earlySingletonExposure = (
	    mbd.isSingleton()                             // 
	    && this.allowCircularReferences               // 默认为true
	    && isSingletonCurrentlyInCreation(beanName)   // 需要创建的Bean正在创建中, 说明该需要创建的Bean被其他单实例Bean依赖
	);  
    if (earlySingletonExposure) {  
	    ... // log: "Eagerly caching bean beanName.. to allow for resolving potential circular references"
	    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));  // 方法 1 
    }  
  
    // 初始化Bean实例, 包含两个部分
    Object exposedObject = bean;  
    try {  
        populateBean(beanName, mbd, instanceWrapper);  
        exposedObject = initializeBean(beanName, exposedObject, mbd);  
    }  
    catch (Throwable ex) {  
		if (ex instanceof BeanCreationException bce && beanName.equals(bce.getBeanName())) {  
            throw bce;  
        }  
		else {  
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, ex.getMessage(), ex);  
        }  
    }  

	// 
    if (earlySingletonExposure) {  
	    // 从一级缓存(即已经完全创建好的缓存)和二级缓存中获取单实例Bean
	    Object earlySingletonReference = getSingleton(beanName, false);  
	    if (earlySingletonReference != null) {  
			// 提前暴露的对象(bean)和经过了完整的生命周期后的对象相等(exposedObject)
		    if (exposedObject == bean) { 
			    exposedObject = earlySingletonReference;  
            }  
            // 检测该bean的dependon的bean是否都已经初始化好了
		    else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {  
	            String[] dependentBeans = getDependentBeans(beanName);  
	            Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);  
	            for (String dependentBean : dependentBeans) {  
	                if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {  
	                    actualDependentBeans.add(dependentBean);  
	                }  
	            }  
		        // 因为bean创建后,其所依赖的 bean一定是创建了的。
				// actualDependentBeans 不为空表示当 bean 创建后依赖的 bean 没有全部创建完，也就是说存在循环依赖
	            if (!actualDependentBeans.isEmpty()) {  
	                throw new BeanCurrentlyInCreationException(beanName,  
	                     "Bean with name beanName.. has been injected into other beans [" +  
	                     StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +  
	                     "] in its raw version as part of a circular reference, but has eventually been " +  
	                     "wrapped. This means that said other beans do not use the final version of the " +  
	                     "bean. This is often the result of over-eager type matching - consider using " +  
	                     "'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");  
	            }  
	        }  
        }  
    }  
  
    // Register bean as disposable.  
    try {  
        registerDisposableBeanIfNecessary(beanName, bean, mbd);  
    }  
    catch (BeanDefinitionValidationException ex) {  
        throw new BeanCreationException("Invalid destruction signature");  
    }  
  
    return exposedObject;  
}                                      
```

## 方法 1 -- 
```java
/* --------------------------------- AbstractAutowireCapableBeanFactory ---------------------------------
 * 参数:
 *   -  beanName: 去掉&后的beanName
 *   -  mbd: 
 *   -  bean: 
 */
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {  
    Object exposedObject = bean;  
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {  
        for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {  
            exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);  
        }  
    }  
    return exposedObject;  
}
```

## addSingletonFactory()
```java
/* ------------------------------- DefaultSingletonBeanRegistry -------------------------------
 */
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {  
    ... // Assert
    
    synchronized (this.singletonObjects) {  
        if (!this.singletonObjects.containsKey(beanName)) {  
            this.singletonFactories.put(beanName, singletonFactory);  
            this.earlySingletonObjects.remove(beanName);  
            this.registeredSingletons.add(beanName);  
        }  
    }  
}
```

