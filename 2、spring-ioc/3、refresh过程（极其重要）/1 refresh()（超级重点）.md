

与之有关的属性：
```java
private ApplicationStartup applicationStartup = ApplicationStartup.DEFAULT;  // 属性2
```

# 一、refresh()（超级重要）

下面是该方法的定义。
```java
@Override  
public void refresh()..{  
	// private final Object startupShutdownMonitor = new Object();
    synchronized (this.startupShutdownMonitor) {  // 属性1
	    // ...: "spring.context.refresh"
        StartupStep contextRefresh = this.applicationStartup.start(...);  

		// Prepare this context for refreshing.  
		// 1.完成了与Environment有关的部分工作
		// 2.完成了与ApplicationEventPublisher有关的部分工作
		prepareRefresh();  // 1

		// 1.refreshBeanFactory()  抽象方法,需要子类来实现
		// 特别注意,该方法会对配置文件或者配置类(取决于实现类)进行读取加载
		// 2.getBeanFactory()  抽象方法,需要子类来实现
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory(); // 2 
		// Prepare the bean factory for use in this context.
		// 1.
		prepareBeanFactory(beanFactory);  // 3
  
        try {  
            // 默认protected NoOp方法,只有在Web应用下的子类才会重写该方法
            postProcessBeanFactory(beanFactory);  // 4

			// ...: 
            StartupStep beanPostProcess = this.applicationStartup.start(...)

			// 
            // 除了调用this.beanFactoryPostProcessors外,
			// 还会在此方法内部调用定义在配置类或配置文件中的beanFactoryPostProcessor
            invokeBeanFactoryPostProcessors(beanFactory);  // 5
  
            // Register bean processors that intercept bean creation.  
            registerBeanPostProcessors(beanFactory);  // 6
            
            ...  // beanPostProcess.end();  
  
            // Initialize message source for this context.  
            initMessageSource();  // 7

            // Initialize event multicaster for this context.  
            initApplicationEventMulticaster();  // 8

            // Initialize other special beans in specific context subclasses.  
            // protected的NoOp方法
            onRefresh();  

            // Check for listener beans and register them.  
            registerListeners(); // 9 
  
            // Instantiate all remaining (non-lazy-init) singletons.  
            finishBeanFactoryInitialization(beanFactory);  // 10

			// Last step: publish corresponding event.  
            finishRefresh();  // 11
        }  
        catch (BeansException ex) {  
	        if (logger.isWarnEnabled()) {...}  
	        // Destroy already created singletons to avoid dangling resources.  
	        destroyBeans();  
            // Reset 'active' flag.  
            cancelRefresh(ex);  
	        // Propagate exception to caller.  
            throw ex;  
        }  
		finally {  
			// Reset common introspection caches in Spring's core, since we  
			// might not ever need metadata for singleton beans anymore...         resetCommonCaches(); 
			contextRefresh.end();  
        }  
    }  
}
```

# 1、相关方法--prepareRefresh()

```java
// -------------------- AbstractApplicationContext --------------------
protected void prepareRefresh() {  
   // Switch to active.  
    this.startupDate = System.currentTimeMillis();  
    this.closed.set(false);  
    this.active.set(true);  
  
    if (logger.isDebugEnabled()) {  
        if (logger.isTraceEnabled()) {  
            logger.trace("Refreshing " + this);  
        }  
	    else {  
            logger.debug("Refreshing " + getDisplayName());  
        }  
    }  
  
    // Initialize any placeholder property sources in the context environment. 
    // 用实际实例取代stub property sources
	// 默认是protected NoOp方法
    initPropertySources();  
  
    // Validate that all properties marked as required are resolvable:  
    // see ConfigurablePropertyResolver#setRequiredProperties   
    getEnvironment().validateRequiredProperties();  
  
    // Store pre-refresh ApplicationListeners...  
    // private Set<ApplicationListener<?>> earlyApplicationListeners;
    // private final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet<>();
    if (this.earlyApplicationListeners == null) {  
		this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);  
    }  
    else {  
	    // Reset local application listeners to pre-refresh state.  
        this.applicationListeners.clear();  
        this.applicationListeners.addAll(this.earlyApplicationListeners);  
    }  
  
    // Allow for the collection of early ApplicationEvents,  
    // to be published once the multicaster is available...   
    this.earlyApplicationEvents = new LinkedHashSet<>();  
}
```

# 2、相关方法--obtainFreshBeanFactory()
```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {  
    refreshBeanFactory();  // 抽象方法,需要子类来实现 
    return getBeanFactory();  // 抽象方法,需要子类来实现
}
```
# 3、涉及的方法--prepareBeanFactory()
```java
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
	// Tell the internal bean factory to use the context's class loader etc.  
	beanFactory.setBeanClassLoader(getClassLoader());  
	// private static final boolean shouldIgnoreSpel = SpringProperties.getFlag("spring.spel.ignore");
    if (!shouldIgnoreSpel) {  
		beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader())); 
    }  
    
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));  
  
   // Configure the bean factory with context callbacks.  
   beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));  
   
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);  
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);  
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);  
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);  
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);  
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);  
    beanFactory.ignoreDependencyInterface(ApplicationStartupAware.class);  
  
   // BeanFactory interface not registered as resolvable type in a plain factory.  
   // MessageSource registered (and found for autowiring) as a bean.  
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);  
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);  
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);  
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);  
  
   // Register early post-processor for detecting inner beans as ApplicationListeners.  
   beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));  
  
   // Detect a LoadTimeWeaver and prepare for weaving, if found.  
   if (!NativeDetector.inNativeImage() && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {  
      beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));  
      // Set a temporary ClassLoader for type matching.  
      beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));  
   }  
  
   // Register default environment beans.  
   if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {  
      beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());  
   }  
   if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {  
      beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());  
   }  
   if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {  
      beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());  
   }  
   if (!beanFactory.containsLocalBean(APPLICATION_STARTUP_BEAN_NAME)) {  
      beanFactory.registerSingleton(APPLICATION_STARTUP_BEAN_NAME, getApplicationStartup());  
   }  
}
```


# 6、

```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {  
   PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);  
}
// --------------- PostProcessorRegistrationDelegate ----------------------------
public static void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {  
   // WARNING: Although it may appear that the body of this method can be easily  
   // refactored to avoid the use of multiple loops and multiple lists, the use   
   // of multiple lists and multiple passes over the names of processors is   
   // intentional. We must ensure that we honor the contracts for PriorityOrdered   
   // and Ordered processors. Specifically, we must NOT cause processors to be   
   // instantiated (via getBean() invocations) or registered in the ApplicationContext   
   // in the wrong order.   
   //   
   // Before submitting a pull request (PR) to change this method, please review the  
   // list of all declined PRs involving changes to PostProcessorRegistrationDelegate   
   // to ensure that your proposal does not result in a breaking change:   
   // https://github.com/spring-projects/spring-framework/issues?q=PostProcessorRegistrationDelegate+is%3Aclosed+label%3A%22status%3A+declined%22  

	// spring默认有2个元素:
	//   - org.springframework.context.annotation.internalAutowiredAnnotationProcessor
	//   - org.springframework.context.annotation.internalCommonAnnotationProcessor
	// springboot中有6个元素(在spring已有的2个基础上多了额外4个):
	//   - org.springframework.boot.context.properties.ConfigurationPropertiesBindingPostProcessor
	//   - webServerFactoryCustomizerBeanPostProcessor
	//   - errorPageRegistrarBeanPostProcessor
	//   - org.springframework.aop.config.internalAutoProxyCreator
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);  
  
    // Register BeanPostProcessorChecker that logs an info message when  
    // a bean is created during BeanPostProcessor instantiation, i.e. when
	// a bean is not eligible for getting processed by all BeanPostProcessors.   
	int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;  
    beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));  
    
    // Separate between BeanPostProcessors that implement PriorityOrdered,  
    // Ordered, and the rest.   
    List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();  
    List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();  
    List<String> orderedPostProcessorNames = new ArrayList<>();  
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();  

	// 
   for (String ppName : postProcessorNames) {  
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {  
            BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);  
            priorityOrderedPostProcessors.add(pp);  
            if (pp instanceof MergedBeanDefinitionPostProcessor) {  
	            internalPostProcessors.add(pp);  
            }  
        }  
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {  
            orderedPostProcessorNames.add(ppName);  
        }  
        else {  
            nonOrderedPostProcessorNames.add(ppName);  
        }  
    }  
  
   // First, register the BeanPostProcessors that implement PriorityOrdered.  
   sortPostProcessors(priorityOrderedPostProcessors, beanFactory);  
   // 
   registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);  
  
    // Next, register the BeanPostProcessors that implement Ordered.  
    List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());  
    for (String ppName : orderedPostProcessorNames) {  
        BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);  
        orderedPostProcessors.add(pp);  
        if (pp instanceof MergedBeanDefinitionPostProcessor) {  
            internalPostProcessors.add(pp);  
        }  
    }  
    sortPostProcessors(orderedPostProcessors, beanFactory);  
    registerBeanPostProcessors(beanFactory, orderedPostProcessors);  
  
   // Now, register all regular BeanPostProcessors.  
   List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());  
   for (String ppName : nonOrderedPostProcessorNames) {  
      BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);  
      nonOrderedPostProcessors.add(pp);  
      if (pp instanceof MergedBeanDefinitionPostProcessor) {  
         internalPostProcessors.add(pp);  
      }  
   }  
   registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);  
  
   // Finally, re-register all internal BeanPostProcessors.  
   sortPostProcessors(internalPostProcessors, beanFactory);  
   registerBeanPostProcessors(beanFactory, internalPostProcessors);  
  
   // Re-register post-processor for detecting inner beans as ApplicationListeners,  
   // moving it to the end of the processor chain (for picking up proxies etc).   beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));  
}
```

