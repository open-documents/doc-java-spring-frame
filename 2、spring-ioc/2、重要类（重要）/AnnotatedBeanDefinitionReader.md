
# 1、被使用的地方

该类被使用在下面的地方：
1）作为Spring应用（非Springboot应用）的常见启动类AnnotationConfigApplicationContext的属性，并在其构造函数中被实例化。
```java
// --------------------------------- AnnotationConfigApplicationContext --------------------------------
private final AnnotatedBeanDefinitionReader reader;  
private final ClassPathBeanDefinitionScanner scanner;

// 4个构造函数中进行实例化的方式,即将AnnotationConfigApplicationContext作为参数传入到该类的构造函数中
this.reader = new AnnotatedBeanDefinitionReader(this);  
this.scanner = new ClassPathBeanDefinitionScanner(this);
```

# 2、构造函数（重要）

下面是该类的构造函数和相关的属性。
```java
private final BeanDefinitionRegistry registry;
private ConditionEvaluator conditionEvaluator; 

public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {  
    this(registry, getOrCreateEnvironment(registry));  // 相关方法1
}
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {  
   ... // Assert
   this.registry = registry;  
   this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);  
   // 注册5个特定的BeanDefinition到容器中
   AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```
## 1）相关方法--getOrCreateEnvironment()
```java
private static Environment getOrCreateEnvironment(BeanDefinitionRegistry registry) {  
	...
    if (registry instanceof EnvironmentCapable) {  
        return ((EnvironmentCapable) registry).getEnvironment();  
    }  
    return new StandardEnvironment();  
}
```

# 3、重要方法--register()
```java
public void register(Class<?>... componentClasses) {  
    for (Class<?> componentClass : componentClasses) {  
        registerBean(componentClass);  
    }  
}
```
# 3、重要方法--registerBean()
```java
public void registerBean(Class<?> beanClass) {  
    doRegisterBean(beanClass, null, null, null, null);  
}
public void registerBean(Class<?> beanClass, @Nullable String name) {  
    doRegisterBean(beanClass, name, null, null, null);  
}
public void registerBean(Class<?> beanClass, Class<? extends Annotation>... qualifiers) {  
    doRegisterBean(beanClass, null, qualifiers, null, null);  
}
public void registerBean(Class<?> beanClass, @Nullable String name,  
      Class<? extends Annotation>... qualifiers) {  
    doRegisterBean(beanClass, name, qualifiers, null, null);  
}
public <T> void registerBean(Class<T> beanClass, @Nullable Supplier<T> supplier) {  
    doRegisterBean(beanClass, null, null, supplier, null);  
}
public <T> void registerBean(Class<T> beanClass, @Nullable String name, @Nullable Supplier<T> supplier) {  
    doRegisterBean(beanClass, name, null, supplier, null);  
}
public <T> void registerBean(Class<T> beanClass, @Nullable String name, @Nullable Supplier<T> supplier,  
      BeanDefinitionCustomizer... customizers) {  
    doRegisterBean(beanClass, name, null, supplier, customizers);  
}
```
# 3、（重要）真正处理逻辑--doRegisterBean()
```java
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,  
      @Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,  
      @Nullable BeanDefinitionCustomizer[] customizers) {  
    
    AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);  
    // this.conditionEvaluator只在构造函数中被实例化
    // this.conditionEvaluator = new ConditionEvaluator(registry, environment, null)
    // 判断是否满足@Conditional条件
    if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {  
        return;
    }  
    
    abd.setInstanceSupplier(supplier);
    ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);  
    abd.setScope(scopeMetadata.getScopeName());  
    String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));  
  
    AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);  
    if (qualifiers != null) {
	    for (Class<? extends Annotation> qualifier : qualifiers) {  
			if (Primary.class == qualifier) {  
	            abd.setPrimary(true);  
            }  
            else if (Lazy.class == qualifier) {  
                abd.setLazyInit(true);  
            }  
	        else {  
	            abd.addQualifier(new AutowireCandidateQualifier(qualifier));  
            }  
        }  
    }  
    if (customizers != null) {  
	    for (BeanDefinitionCustomizer customizer : customizers) {  
            customizer.customize(abd);  
        }  
    }  
  
    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);  
    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);  
}
```




AnnotatedBeanDefinitionReader有如下属性：
```java
// 下面两个属性在构造函数中完成实例化和初始化。
private final BeanDefinitionRegistry registry;  
private ConditionEvaluator conditionEvaluator;

private BeanNameGenerator beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE;  
private ScopeMetadataResolver scopeMetadataResolver = new AnnotationScopeMetadataResolver();  

```
其中：
- BeanDefinitionRegistry：一般是GenericApplicationContext或其子类实例（因为它实现了BeanDefinitionRegistry接口，更具体地说就是AnnotationConfigApplicationContext实例）。
- BeanNameGenerator
- ScopeMetadataResolver：

