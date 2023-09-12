
```java
// ----------------------------- AbstractApplicationContext -----------------------------
protected void finishRefresh() {  
    // Clear context-level resource caches (such as ASM metadata from scanning).  
    clearResourceCaches();  // 1
  
    // Initialize lifecycle processor for this context.  
    initLifecycleProcessor();  // 2
  
    // Propagate refresh to lifecycle processor first.  
    getLifecycleProcessor().onRefresh();  // 3
  
    // Publish the final event.  
    publishEvent(new ContextRefreshedEvent(this));  
  
    // Participate in LiveBeansView MBean, if active.  
    if (!NativeDetector.inNativeImage()) {  
        LiveBeansView.registerApplicationContext(this);  
    }  
}
```

# 1 clearResourceCaches()

```java
// ----------------------------- DefaultResourceLoader -----------------------------
public void clearResourceCaches() {  
    this.resourceCaches.clear();  
}
```


# 2 initLifecycleProcessor()
```java
// ----------------------------- AbstractApplicationContext -----------------------------
protected void initLifecycleProcessor() {  
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();  
    // LIFECYCLE_PROCESSOR_BEAN_NAME = "lifecycleProcessor"
    if (beanFactory.containsLocalBean(LIFECYCLE_PROCESSOR_BEAN_NAME)) {  
        this.lifecycleProcessor = beanFactory.getBean(LIFECYCLE_PROCESSOR_BEAN_NAME, LifecycleProcessor.class);  
		... // log
    }  
    else {  
        DefaultLifecycleProcessor defaultProcessor = new DefaultLifecycleProcessor();  
        defaultProcessor.setBeanFactory(beanFactory);  
        this.lifecycleProcessor = defaultProcessor;  
        beanFactory.registerSingleton(LIFECYCLE_PROCESSOR_BEAN_NAME, this.lifecycleProcessor);  
		... // log
    }  
}
```

# 3 onRefresh()
```java
// -------------------------------------- DefaultLifecycleProcessor --------------------------------------
public void onRefresh() {  
    startBeans(true);  
    this.running = true;  
}

private void startBeans(boolean autoStartupOnly) {  
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();  // 1
    Map<Integer, LifecycleGroup> phases = new TreeMap<>();  
  
    lifecycleBeans.forEach((beanName, bean) -> {  
        if (!autoStartupOnly || (bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())) {  
            int phase = getPhase(bean);  // 2
            phases.computeIfAbsent(  
               phase,  
               p -> new LifecycleGroup(phase, this.timeoutPerShutdownPhase, lifecycleBeans, autoStartupOnly)  
            ).add(beanName, bean);  // 3
        }  
    });  
    if (!phases.isEmpty()) {  
        phases.values().forEach(LifecycleGroup::start);  // 4
    }  
}
```

## getLifecycleBeans()
```java
// -------------------------------------- DefaultLifecycleProcessor --------------------------------------
protected Map<String, Lifecycle> getLifecycleBeans() {  
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();  
    Map<String, Lifecycle> beans = new LinkedHashMap<>();  
    String[] beanNames = beanFactory.getBeanNamesForType(
	    Lifecycle.class, 
	    false,              // includeNonSingletons
	    false               // allowEagerInit
	);  
    for (String beanName : beanNames) {  
        String beanNameToRegister = BeanFactoryUtils.transformedBeanName(beanName);  
        boolean isFactoryBean = beanFactory.isFactoryBean(beanNameToRegister);  
        String beanNameToCheck = (isFactoryBean ? BeanFactory.FACTORY_BEAN_PREFIX + beanName : beanName);  
        if ((beanFactory.containsSingleton(beanNameToRegister) 
	        && (!isFactoryBean || matchesBeanType(Lifecycle.class, beanNameToCheck, beanFactory))) 
	        || matchesBeanType(SmartLifecycle.class, beanNameToCheck, beanFactory)) {  
            Object bean = beanFactory.getBean(beanNameToCheck);  
            if (bean != this && bean instanceof Lifecycle) {  
	            beans.put(beanNameToRegister, (Lifecycle) bean);  
            }  
        }  
    }  
    return beans;  
}
```

## 2 getPhase()
```java
// -------------------------------------- DefaultLifecycleProcessor --------------------------------------
protected int getPhase(Lifecycle bean) {  
    return (bean instanceof Phased ? ((Phased) bean).getPhase() : 0);  
}
```

## 3 add()
```java
// ----------------------------------------- LifecycleGroup -----------------------------------------
public void add(String name, Lifecycle bean) {  

	// 1. this.members:
	// - 类型 List<LifecycleGroupMember>
	// - 默认初值 new ArrayList<>()
	// 2. 
    this.members.add(new LifecycleGroupMember(name, bean));  

    if (bean instanceof SmartLifecycle) {  
        this.smartMemberCount++;  
    }  
}
```

## 4 start()
```java
// ----------------------------------------- LifecycleGroup -----------------------------------------
public void start() {  
    if (this.members.isEmpty()) {  
        return;  
    }  
	... // log
    Collections.sort(this.members);  
    for (LifecycleGroupMember member : this.members) {  
        doStart(this.lifecycleBeans, member.name, this.autoStartupOnly);  
    }  
}
// ------------------------------------ DefaultLifecycleProcessor ------------------------------------
private void doStart(Map<String, ? extends Lifecycle> lifecycleBeans, String beanName, boolean autoStartupOnly) {  
    Lifecycle bean = lifecycleBeans.remove(beanName);  
    if (bean != null && bean != this) {  
        String[] dependenciesForBean = getBeanFactory().getDependenciesForBean(beanName);  
        for (String dependency : dependenciesForBean) {  
            doStart(lifecycleBeans, dependency, autoStartupOnly);  
        }  
        if (!bean.isRunning() 
	        && (!autoStartupOnly || !(bean instanceof SmartLifecycle) || ((SmartLifecycle) bean).isAutoStartup())) {  
			... // log
            try {  
	            bean.start();  
            }
            catch (Throwable ex) {  throw new ApplicationContextException("启动失败");  }  
			... // log: 启动成功
        }  
    }  
}
```