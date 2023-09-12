
该类的主要作用就是扫描给定的包，从中导入符合条件的类。

先看该类的定义。可以看到该类仅简单地继承了ClassPathScanningCandidateComponentProvider类。
```java
public class ClassPathBeanDefinitionScanner extends ClassPathScanningCandidateComponentProvider
```

# 父类--ClassPathScanningCandidateComponentProvider

<font color="FFDD00">第1部分属性</font>：在ClassPathBeanDefinitionScanner的构造函数中通过setEnvironment(Environment)方法设置。
```java
private Environment environment; // @Nullable
private ConditionEvaluator conditionEvaluator; // @Nullable
```
-- --
<font color="FFDD00">第2部分属性</font>：通过setResourceLoader(ResourceLoader)方法设置。
```java
// = ResourcePatternUtils.getResourcePatternResolver(resourceLoader);
private ResourcePatternResolver resourcePatternResolver; // @Nullable 
// = new CachingMetadataReaderFactory(resourceLoader);  
private MetadataReaderFactory metadataReaderFactory; // @Nullable  
// = CandidateComponentsIndexLoader.loadIndex(this.resourcePatternResolver.getClassLoader());  
private CandidateComponentsIndex componentsIndex; // @Nullable  
```
相关方法--loadIndex()
```java
// -------------------------- CandidateComponentsIndexLoader ---------------------------
public static CandidateComponentsIndex loadIndex(@Nullable ClassLoader classLoader) {  
    ClassLoader classLoaderToUse = classLoader;  
    if (classLoaderToUse == null) {  
        classLoaderToUse = CandidateComponentsIndexLoader.class.getClassLoader();  
    }  
    return cache.computeIfAbsent(classLoaderToUse, CandidateComponentsIndexLoader::doLoadIndex);  
}
```
属性方法--getResourcePatternResolver()
```java
private ResourcePatternResolver getResourcePatternResolver() {  
    if (this.resourcePatternResolver == null) {  
        this.resourcePatternResolver = new PathMatchingResourcePatternResolver();  
    }  
    return this.resourcePatternResolver;  
}
```

-- --
<font color="FFDD00">第3部分属性</font>：
```java
// 
private final List<TypeFilter> includeFilters = new ArrayList<>();
// 
private final List<TypeFilter> excludeFilters = new ArrayList<>();
```
-- --
## （重要）findCandidateComponents()

从包中找到符合条件的类，并将其封装为ScannedGenericBeanDefinition。包中的类需要符合的条件：
- 不能满足excludeFilters()属性中的任何一个。（必须满足）
- 需要满足includeFilters()属性中的任何一个，同时，满足@Conditional注解。（必须满足）
- 需要是顶层类或静态内部类，即能够单独实例化的类。（和下面的条件满足一个）
- 标注为抽象，但是内部有@Lookup方法。（和上面的条件满足一个）

```java
public Set<BeanDefinition> findCandidateComponents(String basePackage) {  
    if (this.componentsIndex != null && indexSupportsIncludeFilters()) {  
        return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);  
    }  
    else {  
        return scanCandidateComponents(basePackage);  
    }  
}

private Set<BeanDefinition> scanCandidateComponents(String basePackage) {  
   Set<BeanDefinition> candidates = new LinkedHashSet<>();  
    try {  
	    // 1. ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX = "classpath*:"
	    // 2. resolveBasePackage(basePackage)先用System.properties解析basePackage中的占位符,然后将'.'转换成'/',代码紧跟后面
		// 3. this.resourcePattern = "**/*.class",只获取class文件
        String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +  
            resolveBasePackage(basePackage) +  // 相关方法1
            '/' + 
            this.resourcePattern;  
	    // 1. getResourcePatternResolver(): resourcePatternResolver属性不为空就返回,为空就使用无参构造创建
	    // 2. 获取路径中的.class文件
        Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);  // 2
		... 
        for (Resource resource : resources) {  
	        ...
            try { 
				// ASM的方式读取.class文件
	            MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);  
	            // 通过ASM的方式判断是否满足条件
	            //   - 默认不是
	            //   - 满足属性excludeFilters中的一个,则不是
	            //   - 满足属性includeFilters中的一个&&如果标注了@Conditional并且满足,则是 
	            if (isCandidateComponent(metadataReader)) {  // 相关方法3
	                ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);  
	                sbd.setSource(resource);
	                // 继续判断是否满足条件
	                //   - 顶层类或静态内部类(和下面的条件满足一个)
	                //   - 标注为抽象,但是内部有@Lookup方法(和上面的条件满足一个)
	                if (isCandidateComponent(sbd)) {  // 注意和前面的方法区分,这是方法的重载,3
						...
	                    candidates.add(sbd);  
	                }  
	                else {...}  
	            }  
	            else {...}  
	        }  
	        ...
        }  
    }  
    ...
    return candidates;  
}
```
## 1）相关方法--resolveBasePackage()
```java
protected String resolveBasePackage(String basePackage) {  
    return ClassUtils.convertClassNameToResourcePath(getEnvironment().resolveRequiredPlaceholders(basePackage));  
}
```
## 2）相关方法--getResourcePatternResolver()
```java
private ResourcePatternResolver getResourcePatternResolver() {  
    if (this.resourcePatternResolver == null) {  
        this.resourcePatternResolver = new PathMatchingResourcePatternResolver();  
    }  
    return this.resourcePatternResolver;  
}
```
## 3）相关方法--isCandidateComponent()

isCandidateComponent()有2个重载方法。

第1个重载方法：
```java
protected boolean isCandidateComponent(MetadataReader metadataReader)..{  
    for (TypeFilter tf : this.excludeFilters) {  
	    if (tf.match(metadataReader, getMetadataReaderFactory())) {  
            return false;  
        }  
    }  
    for (TypeFilter tf : this.includeFilters) {  
        if (tf.match(metadataReader, getMetadataReaderFactory())) {  
            return isConditionMatch(metadataReader);  // 代码紧跟后面
        }  
    }  
    return false;  
}

private boolean isConditionMatch(MetadataReader metadataReader) {  
    if (this.conditionEvaluator == null) {  
	    this.conditionEvaluator =  
            new ConditionEvaluator(getRegistry(), this.environment, this.resourcePatternResolver);  
    }  
    return !this.conditionEvaluator.shouldSkip(metadataReader.getAnnotationMetadata());  
}
```
第2个重载方法：
```java
protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {  
    AnnotationMetadata metadata = beanDefinition.getMetadata();  
    
    return (metadata.isIndependent() // 类是否是顶层类或者静态内部类,即能够独立创建实例
	    && 
	    (metadata.isConcrete() // 判断不是接口或者抽象类
		 ||  
		 // 标注为抽象(接口和抽象类)          有@Lookup方法
	     (metadata.isAbstract() && metadata.hasAnnotatedMethods(Lookup.class.getName()))));  
}
```

-- --
# ClassPathBeanDefinitionScanner

ClassPathBeanDefinitionScanner继承了ClassPathScanningCandidateComponentProvider。

属性
```java
private final BeanDefinitionRegistry registry;

private BeanNameGenerator beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE;  
// 两个方法都能设置该属性:
//   - setScopeMetadataResolver(ScopeMetadataResolver): (scopeMetadataResolver != null ? scopeMetadataResolver : new AnnotationScopeMetadataResolver())
//   - setScopedProxyMode(ScopedProxyMode): new AnnotationScopeMetadataResolver(scopedProxyMode)
private ScopeMetadataResolver scopeMetadataResolver = new AnnotationScopeMetadataResolver();
// 设置了
private BeanDefinitionDefaults beanDefinitionDefaults = new BeanDefinitionDefaults();

```
构造函数
```java
// Parameters:
//   - useDefaultFilters: 默认为true

public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, 
									  boolean useDefaultFilters,  
									  Environment environment, 
									  @Nullable ResourceLoader resourceLoader) {  
	...
    this.registry = registry;  
  
    if (useDefaultFilters) {  
        registerDefaultFilters();  
    }
    // 设置 environment,conditionEvaluator 2个属性
    setEnvironment(environment);  
    // 设置 resourcePatternResolver,metadataReaderFactory,componentsIndex 3个属性
    setResourceLoader(resourceLoader);  
}
```

## 1、scan()
```java
public int scan(String... basePackages) {  
    int beanCountAtScanStart = this.registry.getBeanDefinitionCount();  
  
    doScan(basePackages);  
  
    // Register annotation config processors, if necessary.  
    if (this.includeAnnotationConfig) {  
        AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);  
    }  

    return (this.registry.getBeanDefinitionCount() - beanCountAtScanStart);  
}
```

## （重要）真正的处理逻辑--doScan()

需要注意的是：该方法是protected，即外部应用无法直接使用。
```java
// --------------------- ClassPathBeanDefinitionScanner -----------------------------------
// basePackages来自:
//   - @ComponentScan的basePackages(String[])属性
//   - @ComponentScan的basePackageClasses(Class<?>[])属性所在的包
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {  
    Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();  
    for (String basePackage : basePackages) {  
	    // 该方法由父类ClassPathScanningCandidateComponentProvider提供
	    // 从包中找到符合条件的类,并将其封装成BeanDefinition(默认ScannedGenericBeanDefinition)
	    // 包中符合条件的类:
	    //   - 不能满足@ComponentScan.excludeFilters()属性中的任何一个。(必须满足)
		//   - 需要满足@ComponentScan.includeFilters()属性中的任何一个 && 满足@Conditional注解。(必须满足)
		//   - 需要是顶层类或静态内部类，即能够单独实例化的类。(和下面的条件满足一个)
		//   - 标注为抽象，但是内部有@Lookup方法。(和上面的条件满足一个)
        Set<BeanDefinition> candidates = findCandidateComponents(basePackage);  
        for (BeanDefinition candidate : candidates) {  
	        // 解析@Scope注解
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);  
            candidate.setScope(scopeMetadata.getScopeName());  
            // this.beanNameGenerator = AnnotationBeanNameGenerator.INSTANCE,可以通过set()更改
            String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);  
            
            if (candidate instanceof AbstractBeanDefinition) {  // 默认满足
	            // 1.根据该类的BeanDefinitionDefaults类型属性设置BeanDefinition默认值
	            //   - 设置LazyInit属性(boolean,如果BeanDefinitionDefaults.LazyInit属性不为空)
	            //   - 设置自动装配模式(int,BeanDefinitionDefaults中默认为0)
	            //   - 设置依赖性检查(int,BeanDefinitionDefaults中默认为0)
	            //   - 设置初始化方法名称(String,BeanDefinitionDefaults中默认为null)
	            //   - 设置强制使用初始化方法(false,默认true)
	            //   - 设置销毁方法名称(String,BeanDefinitionDefaults中默认为null)
	            //   - 设置强制使用销毁方法(false,默认true)
	            // 2.设置BeanDefinition的AutowireCandidate(Boolean,如果autowireCandidatePatterns属性不为空)
                postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);  // 1
            }
            if (candidate instanceof AnnotatedBeanDefinition) {  // 默认满足
	            // 设置BeanDefinition的下面属性:
	            //  - LazyInit(前面是根据属性设置,这里是根据注解设置)
	            //  - Primary
	            //  - DependsOn
	            //  - Role
	            //  - Description 
                AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);  
            }  
            if (checkCandidate(beanName, candidate)) {  
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);  
                definitionHolder =  
	                AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry); 
	            beanDefinitions.add(definitionHolder);  
	            registerBeanDefinition(definitionHolder, this.registry);  
            }  
        }  
    }  
    return beanDefinitions;  
}
```
### findCandidateComponents()

该方法由父类提供。

### postProcessBeanDefinition()
```java
protected void postProcessBeanDefinition(AbstractBeanDefinition beanDefinition, String beanName) {  
    beanDefinition.applyDefaults(this.beanDefinitionDefaults);  
    // this.autowireCandidatePatterns默认为空,但是能够通过set()设置
    if (this.autowireCandidatePatterns != null) {
		beanDefinition.setAutowireCandidate(PatternMatchUtils.simpleMatch(this.autowireCandidatePatterns, beanName));  
    }  
}
```
### checkCandidate()
```java
protected boolean checkCandidate(String beanName, BeanDefinition beanDefinition) throws IllegalStateException { 
    if (!this.registry.containsBeanDefinition(beanName)) {  
        return true;  
    }  
    BeanDefinition existingDef = this.registry.getBeanDefinition(beanName);  
    BeanDefinition originatingDef = existingDef.getOriginatingBeanDefinition();  
    if (originatingDef != null) {  
        existingDef = originatingDef;  
    }  
    if (isCompatible(beanDefinition, existingDef)) {  
        return false;  
    }  
	... // throws
}
```
### registerBeanDefinition()
```java
protected void registerBeanDefinition(BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry) { 
	BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, registry);  
}
```
