

# （重要）preInstantiateSingletons()

```java
// ------------------------------------------ DefaultListableBeanFactory ------------------------------------------

public void preInstantiateSingletons() throws BeansException {  

	... // log: 在this中进行提前实例化单实例对象
  
    // Iterate over a copy to allow for init methods which in turn register new bean definitions.  
    // While this may not be part of the regular factory bootstrap, it does otherwise work fine. 
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);  
  
    // Trigger initialization of all non-lazy singleton beans...  
    for (String beanName : beanNames) {  
	    // 合并父BeanDefinition的属性, 得到新的RootBeanDefinition
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);  // 该方法由父类AbstractBeanFactory提供
        // bd.isAbstract(): 默认为false
        // bd.isSingleton(): BeanDefinition的scope属性为"singleton"或""
        // bd.isLazyInit(): 是否懒加载
        // 综上,预先初始化的单实例
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {  
	        // beanName对应的类型是FactoryBean
	        if (isFactoryBean(beanName)) {  // isFactory()由父类AbstractBeanFactory提供
		        // 获取FactoryBean实例
		        // 在getBean()执行过程中,
	            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);  // FACTORY_BEAN_PREFIX = "&"
	            
	            if (bean instanceof SmartFactoryBean<?> smartFactoryBean && smartFactoryBean.isEagerInit()) {  
		            // 
		            getBean(beanName);  
                }  
            }  
            // // beanName对应的类型是普通Bean
            else {  
	            // 
	            getBean(beanName);  
            }   
        }  
    }  
  
    // Trigger post-initialization callback for all applicable beans...  
    for (String beanName : beanNames) {  
        Object singletonInstance = getSingleton(beanName);  
        if (singletonInstance instanceof SmartInitializingSingleton smartSingleton) {  
            ... // StartupStep.start()
            smartSingleton.afterSingletonsInstantiated();  
            ... // StartupStep.end()
        }  
    }  
}
```


## 判断是否是FactoryBean -- isFactoryBean()
```java
// ------------------------------------------ AbstractBeanFactory ------------------------------------------
public boolean isFactoryBean(String name) throws NoSuchBeanDefinitionException {  
    String beanName = transformedBeanName(name);  
    Object beanInstance = getSingleton(beanName, false);  
    if (beanInstance != null) {  
        return (beanInstance instanceof FactoryBean);  
    }  
    // No singleton instance found -> check bean definition.  
    if (!containsBeanDefinition(beanName) && getParentBeanFactory() instanceof ConfigurableBeanFactory cbf) {  
        // No bean definition found in this factory -> delegate to parent.  
        return cbf.isFactoryBean(name);  
    }  
    return isFactoryBean(beanName, getMergedLocalBeanDefinition(beanName));  
}

protected boolean isFactoryBean(String beanName, RootBeanDefinition mbd) {  
    Boolean result = mbd.isFactoryBean;  
    if (result == null) {  
        Class<?> beanType = predictBeanType(beanName, mbd, FactoryBean.class);  
        result = (beanType != null && FactoryBean.class.isAssignableFrom(beanType));  
        mbd.isFactoryBean = result;  
    }  
    return result;  
}
```



