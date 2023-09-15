
# AbstractApplicationContext中的方法
```java
/* ------------------------------ AbstractApplicationContext ------------------------------
 * 
 */
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
	// getBeanFactoryPostProcessors返回this.beanFactoryPostProcessors
	// AnnotationConfigApplicationContext的this.beanFactoryPostProcessors为空
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());  
  
    // Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime  
    // (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)   
    if (!NativeDetector.inNativeImage() && beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {  
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));  
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));  
    }  
}
```
# 1、（重要）主要逻辑

在此之前需要弄清BeanFactoryPostProcessor接口和BeanDefinitionRegistryPostProcessor接口的关系，后者是前者的子接口。
```java
/* --------------------- PostProcessorRegistrationDelegate --------------------------
 *  参数:
 *    - beanFactory: 
 *    - beanFactoryPostProcessors: AbstractApplicationContext.beanFactoryPostProcessors
 */
public static void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {  
  
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
   
   // Invoke BeanDefinitionRegistryPostProcessors first, if any.   
    Set<String> processedBeans = new HashSet<>();  

	// 只有AnnotationConfigApplicationContext符合该条件，即只有注解方式的ApplicationContext才会符合
    if (beanFactory instanceof BeanDefinitionRegistry) {  
        BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;  
        // regularPostProcessors只来自于AbstractApplicationContext的beanFactoryPostProcessors属性中的BeanFactoryPostProcessor实例
        List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();  
        // registryProcessors来自于:
        //   - AbstractApplicationContext的beanFactoryPostProcessors属性中的BeanDefinitionRegistryPostProcessor实例
        //   - 在AbstractApplicationContext#refresh()方法前注册的BeanDefinitionRegistryPostProcessor类型的BeanDefinition
        List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();  
        // beanFactoryPostProcessors: 来自方法参数,默认为空
        for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {  
            if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {  
		        BeanDefinitionRegistryPostProcessor registryProcessor =  
                       (BeanDefinitionRegistryPostProcessor) postProcessor;  
                registryProcessor.postProcessBeanDefinitionRegistry(registry); 
                registryProcessors.add(registryProcessor);  
            }  
            else {  
                regularPostProcessors.add(postProcessor);  
            }  
        } 
		// Do not initialize FactoryBeans here: We need to leave all regular beans  
	    // uninitialized to let the bean factory post-processors apply to them!      
	    // Separate between BeanDefinitionRegistryPostProcessors that implement      
	    // PriorityOrdered, Ordered, and the rest.      
	    List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();  
  
		// 从ApplicationContext的beanFactory中获取类型为BeanDefinitionRegistryPostProcessor的
		// BeanNames 
	    String[] postProcessorNames =  
	    beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);  

	    for (String ppName : postProcessorNames) {  
		    // 判断是否实现了PriorityOrdered接口,满足条件的:
		    //   - internalConfigurationAnnotationProcessor
	        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {  
	            currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));  
	            processedBeans.add(ppName);  
	        }  
	    }  
	    sortPostProcessors(currentRegistryProcessors, beanFactory);  
	    registryProcessors.addAll(currentRegistryProcessors);
	    // 简单地遍历，挨个调用BeanDefinitionRegistryPostProcessor
	    invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());  // 1
	    currentRegistryProcessors.clear();  
  
	    // 其次，调用另外还实现了Ordered接口的BeanDefinitionRegistryPostProcessors
	    postProcessorNames = 
	    beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);  
	    for (String ppName : postProcessorNames) {  
	        if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {  
				currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));  
				processedBeans.add(ppName);  
	        }  
	    }  
	    sortPostProcessors(currentRegistryProcessors, beanFactory);  
	    registryProcessors.addAll(currentRegistryProcessors);  
	    invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());  
	    currentRegistryProcessors.clear();  
  
		// 最后，调用普通的BeanDefinitionRegistryPostProcessors
	    boolean reiterate = true;  
	    while (reiterate) {  
		    reiterate = false;  
	        postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);  
		    for (String ppName : postProcessorNames) {  
		        if (!processedBeans.contains(ppName)) {  
		            currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));  
	                processedBeans.add(ppName);  
	                reiterate = true;  
	            }  
	        }  
	        sortPostProcessors(currentRegistryProcessors, beanFactory);  
	        registryProcessors.addAll(currentRegistryProcessors);  
	        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry, beanFactory.getApplicationStartup());  
	        currentRegistryProcessors.clear();  
	    }  

		// 下面调用BeanFactoryPostProcessor的postProcessBeanFactory()方法.
	    // 调用BeanFactoryPostProcessor接口子接口BeanDefinitionRegistryPostProcessor的postProcessBeanFactory()部分
		invokeBeanFactoryPostProcessors(registryProcessors, beanFactory); 
		// 调用BeanFactoryPostProcessor接口的postProcessBeanFactory()方法
	    invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);  
    }  
  
    else {  //  符合该条件的是AbstractApplicationContext的另一分支，即非注解方式的ApplicationContext
        invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);  
    }  
  
	// Do not initialize FactoryBeans here: We need to leave all regular beans  
    // uninitialized to let the bean factory post-processors apply to them!   
    // AnnotationConfigApplicationContext中有两个:
    // 1) EventListenerMethodProcessor
    // 2) ConfigurationClassPostProcessor
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);  
    // Separate between BeanFactoryPostProcessors that implement PriorityOrdered,  
    // Ordered, and the rest.
   List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();  
   List<String> orderedPostProcessorNames = new ArrayList<>();  
   List<String> nonOrderedPostProcessorNames = new ArrayList<>();  
   for (String ppName : postProcessorNames) {  
        if (processedBeans.contains(ppName)) {  
            // skip - already processed in first phase above  
            // ConfigurationClassPostProcessor因为实现的是BeanDefinitionRegistryPostProcess接口，所以在上面的逻辑中已经被处理过了
        }  
        else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {  
            priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));  
        }  
        else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {  
            orderedPostProcessorNames.add(ppName);  
        }  
        else {  
            nonOrderedPostProcessorNames.add(ppName);  
        }  
    }  

    // 首先，调用另外实现了PriorityOrdered接口的BeanFactoryPostProcessor接口实例
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);  
    invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);  
  
    // 其次，调用另外实现了Ordered接口的BeanFactoryPostProcessor接口实例
    List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());  
    for (String postProcessorName : orderedPostProcessorNames) {  
        orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));  
    }  
    sortPostProcessors(orderedPostProcessors, beanFactory);  
    invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);  
  
    // 最后，调用普通的BeanFactoryPostProcessor接口实例
    List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());  
    for (String postProcessorName : nonOrderedPostProcessorNames) {  
        nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));  
    }  
    invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);  
  
    // Clear cached merged bean definitions since the post-processors might have  
    // modified the original metadata, e.g. replacing placeholders in values...   
    beanFactory.clearMetadataCache();  
}
```
## 5.1、相关方法--invokeBeanDefinitionRegistryPostProcessors()
```java
private static void invokeBeanDefinitionRegistryPostProcessors(Collection<? extends BeanDefinitionRegistryPostProcessor> postProcessors, BeanDefinitionRegistry registry, ApplicationStartup applicationStartup) {  
  
    for (BeanDefinitionRegistryPostProcessor postProcessor : postProcessors) {  
		// ...: "spring.context.beandef-registry.post-process"
        StartupStep postProcessBeanDefRegistry = applicationStartup.start(...)  
            .tag("postProcessor", postProcessor::toString);  
        postProcessor.postProcessBeanDefinitionRegistry(registry);  
        postProcessBeanDefRegistry.end();  
    }  
}
```



首先明确该方法的功能，该方法是refresh()过程中invokeBeanFactoryPostProcessors(beanFactory)方法的具体实现，那么从名字可以看出是对该ApplicationContext中的<font color="FF5500">BeanFactoryPostProcessor</font>接口实例进行调用。

<font color="FF5500">BeanFactoryPostProcessor</font>接口实例有两个来源：
- AbstractApplicationContext的`List<BeanFactoryPostProcessor>`类型属性beanFactoryPostProcessors，记为1。
- 可以根据已注册的BeanDefinition来创建实例，记为2。

总体而言，流程是这样的。

<font color=44cf57>一、如果ConfigurableListableBeanFactory实例是BeanDefinitionRegistry类型，</font>除了需要对其中的BeanFactoryPostProcessors做postProcessBeanFactory()调用，需要先对<font color="00E0FF">BeanDefinitionRegistryPostProcessors</font>做postProcessBeanDefinitionRegistry()调用。
而<font color="00E0FF">BeanDefinitionRegistryPostProcessors</font>有两个来源：
- AbstractApplicationContext的`List<BeanFactoryPostProcessor>`类型属性beanFactoryPostProcessors（可以根据instanceof关键字对BeanFactoryPostProcessor实例做进一步BeanDefinitionRegistryPostProcessor类型判断），记为3。
- 可以根据已注册的BeanDefinition来创建实例，记为4。

然后：
1）对3中实例做<font color="00E0FF">postProcessBeanDefinitionRegistry()</font>调用。
2）对4中实例做<font color="00E0FF">postProcessBeanDefinitionRegistry()</font>调用。
3）对1中实例做<font color="FF5500">postProcessBeanFactory()</font>调用。
4）对2中实例做<font color="FF5500">postProcessBeanFactory()</font>调用。

<font color=44cf57>二、如果ConfigurableListableBeanFactory实例不是BeanDefinitionRegistry类型，</font>那么只需要对BeanFactoryPostProcessor做postProcessBeanFactory()调用。
即：
1）对1中实例做<font color="FF5500">postProcessBeanFactory()</font>调用。
2）对2中实例做<font color="FF5500">postProcessBeanFactory()</font>调用。（和前面的4一段代码）



更具体地说，按下面的流程做了几件事情。

如果ConfigurableListableBeanFactory实例是BeanDefinitionRegistry类型（即通过注解方式创建的ApplicationContext，如果AnnotationConfigApplicationContext类型实例）：
1）遍历AbstractApplicationContext中List类型beanFactoryPostProcessors属性，如果元素类型是BeanDefinitionRegistryPostProcessor，那么直接对其进行调用，即postProcessBeanDefinitionnRegistry()；如果元素类型是普通的BeanFactoryPostProcessor，那么先将其放入regularPostProcessors局部变量中。
2.1）根据已注册的BeanDefinitionRegistryPostProcessor类型的BeanDefition，获取实现了PriorityOrdered接口的BeanDefinitionRegistryPostProcessor类型实例，将其加入registryProcessors数组，并排序，然后依次调用。
2.2）根据已注册的BeanDefinitionRegistryPostProcessor类型的BeanDefition，获取实现了Ordered接口的BeanDefinitionRegistryPostProcessor类型实例，将其加入registryProcessors数组，并排序，然后依次调用。
2.3）根据已注册的BeanDefinitionRegistryPostProcessor类型的BeanDefition，获取普通的BeanDefinitionRegistryPostProcessor类型实例，将其加入registryProcessors数组，并排序，然后依次调用。
3.1）上面已经完成了BeanDefinitionRegistryPostProcessor类型对象自身的功能（即postProcessBBeanDefinitionRegistry()），然而BeanDefinitionRegistryPostProcessor接口继承了BeanFactoryPostProcessor接口，还需要完成postProcessBeanFactory()功能，所以遍历registryProcessors数组，依次调用postProcessBeanFactory()方法。
3.2）遍历regularPostProcessors数组，依次调用postProcessBeanFactory()方法。
