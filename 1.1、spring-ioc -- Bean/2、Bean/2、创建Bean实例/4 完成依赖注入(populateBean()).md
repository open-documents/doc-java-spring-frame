
参考文档：https://juejin.cn/post/7082995289988005924

# 初始化过程
```java
/* ------------------------------ AbstractAutowireCapableBeanFactory ------------------------------
 * 
 */
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {  
	// 需要填充的Bean为空
    if (bw == null) {  
	    // 需要填充的Bean为空,但是又通过<property/>指定了属性,则需要抛出异常
	    if (mbd.hasPropertyValues()) {  
            throw new BeanCreationException(..., "Cannot apply property values to null instance");  
        }  
        else {  
            // Skip property population phase for null instance.  
            return;  
        }  
    }  

	// 需要填充的Bean为Record类型
    if (bw.getWrappedClass().isRecord()) { 
	    // 需要填充的Bean为Record类型,尽管通过<property/>指定了属性,但是无法为Record类型实例填充属性
	    if (mbd.hasPropertyValues()) {  
		    throw new BeanCreationException(..., "Cannot apply property values to a record");  
        }  
        else {  
            // Skip property population phase for records since they are immutable.  
            return;  
        }  
    }  
  
	// 除了Spring提供的

	// 有两个作用:
	// 1. 给InstantiationAwareBeanPostProcessors一个修改bean实例的机会
	// 2. 一个特殊的用法: 返回false,直接绕过属性填充
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {  
	    for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {  
		    // 
		    if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {  
	            return;  
            }  
        }  
    }  

	// xml的<property/>标签会被抽象为PropertyValue实例
	// PropertyValues包含一个或多个PropertyValue对象
    PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);  
  
    int resolvedAutowireMode = mbd.getResolvedAutowireMode();  
    if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {  
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);  
        // 通过byName的方式完成属性的自动装配
        if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {   
            autowireByName(beanName, mbd, bw, newPvs);  // 1
        }  
        // 通过byType的方式完成属性的自动装配
        if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {  
            autowireByType(beanName, mbd, bw, newPvs);  
        }  
        pvs = newPvs;
    }  


    if (hasInstantiationAwareBeanPostProcessors()) {  
		if (pvs == null) {  
            pvs = mbd.getPropertyValues();  
        }  
        // Spring和Springboot中getBeanPostProcessorCache().instantiationAware默认只有下面1个:
        //   - ConfigurationClassPostProcessor$ImportAwareBeanPostProcessor
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) { 
	        // 调用BeanPostProcessor分别解析@Autowired、@Resource、@Value 等等注解，得到属性值 
            PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);  
            if (pvsToUse == null) {
	            return;  
            }  
            pvs = pvsToUse;
        }   
    }

	// 是否进行依赖检查,默认不进行依赖检查
    boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);  
    if (needsDepCheck) {  
        PropertyDescriptor[] filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);  
        checkDependencies(beanName, mbd, filteredPds, pvs);  
    }
  
    if (pvs != null) {  
        applyPropertyValues(beanName, mbd, bw, pvs);  
    }  
}
```

# 1 按名称自动装配
```java
/* --------------------------------- AbstractAutowireCapableBeanFactory ---------------------------------
 * 参数:
 *   
 *   pvs: MutablePropertyValues,其中包含了通过<property/>标签抽象出来的诸多PropertyValue对象
 */
protected void autowireByName(String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {  
	// 获取需要自动装配的属性名称
    String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);  // 1
    for (String propertyName : propertyNames) {  
	    // 父类AbstractBeanFactory提供的方法
	    // 
        if (containsBean(propertyName)) {  // 2
            Object bean = getBean(propertyName);  
            pvs.add(propertyName, bean);  
            // 注册Bean和Bean之间的依赖关系
            registerDependentBean(propertyName, beanName);  
            ... // log: Added autowiring by name from bean name .. via property .. to bean named ..);  
            }  
        }  
        else {  
		    ... // log: "Not autowiring property .. of bean .. by name: no matching bean found);  
		}  
    }  
}
```
## 1 相关方法--unsatisfiedNonSimpleProperties()
```java
/* --------------------------------- AbstractAutowireCapableBeanFactory ---------------------------------
 * 功能: 获得当前实例所有公开的(包括父类) set 方法,有如下几种情况：
 *   - 无返回值的set有参方法(只能是一个参数),如public void setOrderService(OrderService order)
 *   - 无参有返回值的get方法(返回类型不能是void),如public String getba(){ return null;}
 *   - 返回值为boolean类型的is无参方法,如public boolean isDel(){return false;}
 * 参数:
 *   mbd: 
 *   bw: 
 */
protected String[] unsatisfiedNonSimpleProperties(AbstractBeanDefinition mbd, BeanWrapper bw) {  
    Set<String> result = new TreeSet<>();  

    PropertyValues pvs = mbd.getPropertyValues();  
    // 获得当前实例的所有属性对应的PropertyDescriptor
	// PropertyDescriptor已经不是Spring的类,它属于java.beans包
    PropertyDescriptor[] pds = bw.getPropertyDescriptors();  
    for (PropertyDescriptor pd : pds) {  
	    // 1. 属性有对应的set方法
	    // 2. 没有被排除在依赖检查之外,需要满足下面的条件
	    //   a. 该方法不是定义在CGLIB中的方法,或者虽然是CGLIB中的方法但是在CGLIB代理类(即用户定义的类)中也定义过的方法
	    //   b. 属性的类型没有出现在ignoredDependencyTypes的诸多类型中
	    //   c. set方法没有定义在ignoredDependencyInterfaces的诸多接口中
	    // 3. 没有通过<property/>标签为该属性赋值
	    // 4. 该属性的类型不是简单属性,简单属性类型包括:Void,void,primitive及其包装类,Enum、CharSequence、Number、
		//    Date、Temporal、URI、URL、Locale、Class及其子类
	    if (pd.getWriteMethod() != null 
		    && !isExcludedFromDependencyCheck(pd) 
		    && !pvs.contains(pd.getName())
		    && !BeanUtils.isSimpleProperty(pd.getPropertyType())) {  
            result.add(pd.getName());  
        }  
    }  
    return StringUtils.toStringArray(result);  
}
```
## 2 相关方法--containsBean()
```java
/* Class: AbstractBeanFactory
 *
 */
public boolean containsBean(String name) {  
    String beanName = transformedBeanName(name);  
    if (containsSingleton(beanName) || containsBeanDefinition(beanName)) {  
        return (!BeanFactoryUtils.isFactoryDereference(name) || isFactoryBean(name));  
    }  
    // Not found -> check parent.  
    BeanFactory parentBeanFactory = getParentBeanFactory();  
    return (parentBeanFactory != null && parentBeanFactory.containsBean(originalBeanName(name)));  
}
```

# 2 按类型自动装配
```java
/* Class: AbstractAutowireCapableBeanFactory
 *
 */
protected void autowireByType(String beanName, AbstractBeanDefinition mbd, BeanWrapper bw, MutablePropertyValues pvs) {  
  
   TypeConverter converter = getCustomTypeConverter();  
   if (converter == null) {  
      converter = bw;  
   }  
  
   Set<String> autowiredBeanNames = new LinkedHashSet<>(4);  
   String[] propertyNames = unsatisfiedNonSimpleProperties(mbd, bw);  
   for (String propertyName : propertyNames) {  
      try {  
         PropertyDescriptor pd = bw.getPropertyDescriptor(propertyName);  
         // Don't try autowiring by type for type Object: never makes sense,  
         // even if it technically is an unsatisfied, non-simple property.         if (Object.class != pd.getPropertyType()) {  
            MethodParameter methodParam = BeanUtils.getWriteMethodParameter(pd);  
            // Do not allow eager init for type matching in case of a prioritized post-processor.  
            boolean eager = !(bw.getWrappedInstance() instanceof PriorityOrdered);  
            DependencyDescriptor desc = new AutowireByTypeDependencyDescriptor(methodParam, eager);  
            Object autowiredArgument = resolveDependency(desc, beanName, autowiredBeanNames, converter);  
            if (autowiredArgument != null) {  
               pvs.add(propertyName, autowiredArgument);  
            }  
            for (String autowiredBeanName : autowiredBeanNames) {  
               registerDependentBean(autowiredBeanName, beanName);  
               if (logger.isTraceEnabled()) {  
                  logger.trace("Autowiring by type from bean name '" + beanName + "' via property '" +  
                        propertyName + "' to bean named '" + autowiredBeanName + "'");  
               }  
            }  
            autowiredBeanNames.clear();  
         }  
      }  
      catch (BeansException ex) {  
         throw new UnsatisfiedDependencyException(mbd.getResourceDescription(), beanName, propertyName, ex);  
      }  
   }  
}
```

# 注册