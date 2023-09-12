
向容器中注册用于处理注解的5个BeanDefinition由AnnotationConfigUtils类的两个静态方法提供。

向BeanDefinitionRegistry中无条件注册4个RootBeanDefinition。

| beanName                                                                        | class                                       |
| ------------------------------------------------------------------------------- | ------------------------------------------ |
| org.springframework.context.annotation.internalConfigurationAnnotationProcessor | ConfigurationClassPostProcessor      |
| org.springframework.context.annotation.internalAutowiredAnnotationProcessor     | AutowiredAnnotationBeanPostProcessor |
| org.springframework.context.event.internalEventListenerProcessor                | EventListenerMethodProcessor         |
| org.springframework.context.event.internalEventListenerFactory  |  DefaultEventListenerFactory |

4）向AnnotationConfigApplicationContext中根据情况注册2个RootBeanDefinition。

| beanName                                                                 | class                             |
| ------------------------------------------------------------------------ | --------------------------------- |
| org.springframework.context.annotation.internalCommonAnnotationProcessor | CommonAnnotationBeanPostProcessor |
| org.springframework.context.annotation.internalPersistenceAnnotationProcessor |                                   |




# 第一个方法

该方法在下面的方法中被调用：
- AnnotatedBeanDefinitionReader的构造函数中。
- ClassPathBeanDefinitionScanner的scan()实例方法中。

下面是该方法的定义。
```java
/* ---------------------------------- AnnotationConfigUtils ----------------------------------
 *
 */
public static void registerAnnotationConfigProcessors(BeanDefinitionRegistry registry) {  
   registerAnnotationConfigProcessors(registry, null);  
}
```


# 第二个方法

该方法在下面的地方被调用：
- AnnotationConfigBeanDefinitionParser的parse()方法中。该类用于解析<context:annotation-config/>元素。
- ComponentScanBeanDefinitionParser的registerComponents()方法中。该类用于解析<context:component-scan/>元素。

下面是该方法的定义。
```java
/* ---------------------------------- AnnotationConfigUtils ----------------------------------
 *
 */
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(BeanDefinitionRegistry registry, @Nullable Object source) {
	// registry的类型可能是DefaultListableBeanFactory，也可能是GenericApplicationContext
	// 如果是DefaultListableBeanFactory，直接返回DefaultListableBeanFactory
	// 如果是GenericApplicationContext，通过getDefaultListableBeanFactory()获取DefaultListableBeanFactory
    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);  

	// 简单地设置DefaultListableBeanFactory的两个属性
    if (beanFactory != null) {  
        if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {  
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);  
        }  
        if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {  
            beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());  
        }  
    }  

    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);  

	// beanName: org.springframework.context.annotation.internalConfigurationAnnotationProcessor
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {  
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);  
        def.setSource(source);
        // registerPostProcessor():
        // 设置AbstractBeanDefinition的role为BeanDefinition.ROLE_INFRASTRUCTURE
        // 向registry中注册[beanName : BeanDefinition]键值对
        // 将BeanDefinition和beanName简单封装成BeanDefinitionHolder，并将其返回
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));  
    }  

	// beanName: org.springframework.context.annotation.internalAutowiredAnnotationProcessor
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {  
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);  
        def.setSource(source);  
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));  
   }  

    // Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.  
    // beanName: org.springframework.context.annotation.internalCommonAnnotationProcessor
    if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {  
        RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);  
        def.setSource(source);  
        beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));  
    }  

	// Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor. 
	// beanName: org.springframework.context.annotation.internalPersistenceAnnotationProcessor
    if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {  
        RootBeanDefinition def = new RootBeanDefinition();  
        try {  
            def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,  
                    AnnotationConfigUtils.class.getClassLoader()));  
        }  
        catch (ClassNotFoundException ex) {  
            throw new IllegalStateException(...);  
        }  
        def.setSource(source);  
        beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));  
    }  

	// beanName: org.springframework.context.event.internalEventListenerProcessor
    if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {  
        RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);  
        def.setSource(source);  
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));  
    }

	// beanName: org.springframework.context.event.internalEventListenerFactory
    if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {  
        RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);  
        def.setSource(source);  
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));  
    }  

    return beanDefs;  
}
```
