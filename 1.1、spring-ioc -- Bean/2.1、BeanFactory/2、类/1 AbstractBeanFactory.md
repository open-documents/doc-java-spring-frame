
# 属性

```java
// key: 
// value: 
// 只有getMergedBeanDefinition()向该属性中添加元素
// 而getMergedBeanDefinition()也会在下面的方法中被调用:
//   - getBeanNamesForType()
private final Map<String, RootBeanDefinition> mergedBeanDefinitions = new ConcurrentHashMap<>(256);


```


因为该接口的直接实现类是抽象类AbstractBeanFactory，因此下面的属性如无特殊说明，均是在出现在抽象子类AbstractBeanFactory中。

| 属性| 类型| 描述 | bean方法|
| ------------------------ | ------------------------------ | ---- | ---------------------------------------------------------------------------------------------------------------------- |
| beanExpressionResolver   | `BeanExpressionResolver`       |      | - getBeanExpressionResolver()</br> - setBeanExpressionResolver(@Nullable BeanExpressionResolver)                       |
| beanClassLoader          | `ClassLoader`                  |      | - getBeanClassLoader()</br> - setBeanClassLoader(@Nullable ClassLoader)                                                |
|beanPostProcessors|`List<BeanPostProcessor>`|      |- addBeanPostProcessor(BeanPostProcessor)</br>- addBeanPostProcessors(`Collection<? extends BeanPostProcessor>`)</br>- getBeanPostProcessorCount(): int |
|                          |                                |      |                                                                                                                        |



# 类型转换属性

| 属性 | 类型 | 描述 | bean方法 |
| ---- | ---- | ---- | -------- |
|embeddedValueResolvers|`CopyOnWriteArrayList<StringValueResolver>`|      |- hasEmbeddedValueResolver(): boolean</br>- addEmbeddedValueResolver(StringValueResolver)</br>- resolveEmbeddedValue(String): String |
| propertyEditorRegistrars | `Set<PropertyEditorRegistrar>` |      | - addPropertyEditorRegistrar(PropertyEditorRegistrar)</br>- getPropertyEditorRegistrars(): 在AbstractBeanFactory中提供 |
|conversionService|ConversionService||- setConversionService(@Nullable ConversionService)</br> - getConversionService()|

初始化

| 属性              | 初始化                                                                                                                                                                                                     |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|propertyEditorRegistrars||
conversionService

在AbstractApplicationContext.refresh()的finishBeanFactoryInitialization()中,获取BeanFactory中名为conversionService的Bean,如果类型为ConversionService,则通过set...()方法,设置BeanFactory的ConversionService




| 属性                   | 初始化|
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| beanExpressionResolver | 在AbstractApplicationContext中根据SpringProperties的spring.spel.ignore属性来设置,如果设置,实际类型为StandardBeanExpressionResolver. |
|                        |                                                                                                                                     |











# 1、BeanFactory接口方法--getBean()

这一系列方法没有在子类中重写。
```java
public Object getBean(String name) ..{  
    return doGetBean(name, null, null, false);  
}
public <T> T getBean(String name, Class<T> requiredType) ..{  
    return doGetBean(name, requiredType, null, false);  
}
public Object getBean(String name, Object... args) ..{  
    return doGetBean(name, null, args, false);  
}
public <T> T getBean(String name, @Nullable Class<T> requiredType, @Nullable Object... args) ..{  
    return doGetBean(name, requiredType, args, false);  
}
```

# getMergedLocalBeanDefinition()
```java
/*
    Function: 
 */
public BeanDefinition getMergedBeanDefinition(String name)..{  
	// 1. 递归去掉name中所有"&"前缀,举个例子: &abc&def&g -> g
	// 2. 获取别名,如果有的话
    String beanName = transformedBeanName(name);  
    // containsBeanDefinition(): 抽象方法
    // 如果当前BeanFactory中没有名为beanName的BeanDefinition,则尝试从父BeanFactory中获取
    if (!containsBeanDefinition(beanName) && getParentBeanFactory() instanceof ConfigurableBeanFactory parent) {  
        return parent.getMergedBeanDefinition(beanName);  
    }
    // Resolve merged bean definition locally.  
    return getMergedLocalBeanDefinition(beanName);  // 进入下面的函数
}

protected RootBeanDefinition getMergedLocalBeanDefinition(String beanName)..{  
    // Quick check on the concurrent map first, with minimal locking.  
    // 先尝试从缓存中查找,如果在此之前已经处理过beanName,那么直接获取
    RootBeanDefinition mbd = this.mergedBeanDefinitions.get(beanName);  
    if (mbd != null && !mbd.stale) {  
        return mbd;  
    }  
    return getMergedBeanDefinition(beanName, getBeanDefinition(beanName));  // 进入下面的函数
}

protected RootBeanDefinition getMergedBeanDefinition(String beanName, BeanDefinition bd)..{  
    return getMergedBeanDefinition(beanName, bd, null);  // 进入下面的函数
}

protected RootBeanDefinition getMergedBeanDefinition(String beanName, BeanDefinition bd, @Nullable BeanDefinition containingBd)..{  

	synchronized (this.mergedBeanDefinitions) {  
	
        RootBeanDefinition mbd = null;  
        RootBeanDefinition previous = null;  
  
	    // Check with full lock now in order to enforce the same merged instance.  
	    // 待理解
	    if (containingBd == null) {  
            mbd = this.mergedBeanDefinitions.get(beanName);  
        }  
  
        if (mbd == null || mbd.stale) {  
	        previous = mbd;  
            if (bd.getParentName() == null) {  
	            // Use copy of given root bean definition.  
	            if (bd instanceof RootBeanDefinition rootBeanDef) {  
		            mbd = rootBeanDef.cloneBeanDefinition();  
	            }  
	            else {
		            // 创建一个给定BeanDefinition的深拷贝,即将给定BeanDefinition的属性赋值到RootBeanDefinition实例中
	                mbd = new RootBeanDefinition(bd);  
	            }  
	        }  
            else {  
	            // Child bean definition: needs to be merged with parent.  
	            BeanDefinition pbd;  
	            try {  
	                String parentBeanName = transformedBeanName(bd.getParentName());  
	                if (!beanName.equals(parentBeanName)) {  
	                    pbd = getMergedBeanDefinition(parentBeanName);  // public的getMergedBeanDefinition()
	                }  
	                else {  // 待理解
		                if (getParentBeanFactory() instanceof ConfigurableBeanFactory parent) {  
	                        pbd = parent.getMergedBeanDefinition(parentBeanName);  
	                    }  
	                    else {  
	                        throw new NoSuchBeanDefinitionException(parentBeanName,  
                            "Parent name '" + parentBeanName + "' is equal to bean name '" + beanName +  
                                 "': cannot be resolved without a ConfigurableBeanFactory parent");  
                        }  
                    }  
                }  
	            catch (NoSuchBeanDefinitionException ex) {  
	                throw new BeanDefinitionStoreException(bd.getResourceDescription(), beanName,  
                     "Could not resolve parent bean definition '" + bd.getParentName() + "'", ex);  
	            }  
	            // 先用BeanDefinition的父BeanDefinition属性来创建RootBeanDefinition
	            mbd = new RootBeanDefinition(pbd);  
	            // 再用子BeanDefinition(也就是该BeanDefinition)的属性来覆盖父BeanDefinition的属性
				// 至此就完成了子和父Definition的合并,完成了该方法的名称的应有之义
	            mbd.overrideFrom(bd);  
	        }  
  
	        // Set default singleton scope, if not configured before.  
	        if (!StringUtils.hasLength(mbd.getScope())) {  
	            mbd.setScope(SCOPE_SINGLETON);  
            }
  
            // A bean contained in a non-singleton bean cannot be a singleton itself.  
            // Let's correct this on the fly here, since this might be the result of         
            // parent-child merging for the outer bean, in which case the original inner bean         
            // definition will not have inherited the merged outer bean's singleton status.         
            if (containingBd != null && !containingBd.isSingleton() && mbd.isSingleton()) {  
	            mbd.setScope(containingBd.getScope());  
            }  
  
            // Cache the merged bean definition for the time being  
            // (it might still get re-merged later on in order to pick up metadata changes)         
            if (containingBd == null && isCacheBeanMetadata()) {  
	            this.mergedBeanDefinitions.put(beanName, mbd);  
            }  
        }
        if (previous != null) {  
            copyRelevantMergedBeanDefinitionCaches(previous, mbd);  
        }  
        return mbd;  
    }  
}
```
## 相关方法--copyRelevantMergedBeanDefinitionCaches()
```java
private void copyRelevantMergedBeanDefinitionCaches(RootBeanDefinition previous, RootBeanDefinition mbd) {  
   if (ObjectUtils.nullSafeEquals(mbd.getBeanClassName(), previous.getBeanClassName()) &&  
         ObjectUtils.nullSafeEquals(mbd.getFactoryBeanName(), previous.getFactoryBeanName()) &&  
         ObjectUtils.nullSafeEquals(mbd.getFactoryMethodName(), previous.getFactoryMethodName())) {  
      ResolvableType targetType = mbd.targetType;  
      ResolvableType previousTargetType = previous.targetType;  
      if (targetType == null || targetType.equals(previousTargetType)) {  
         mbd.targetType = previousTargetType;  
         mbd.isFactoryBean = previous.isFactoryBean;  
         mbd.resolvedTargetType = previous.resolvedTargetType;  
         mbd.factoryMethodReturnType = previous.factoryMethodReturnType;  
         mbd.factoryMethodToIntrospect = previous.factoryMethodToIntrospect;  
      }  
   }  
}
```



```java
public boolean isSingletonCurrentlyInCreation(String beanName) {  
	// private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));
    return this.singletonsCurrentlyInCreation.contains(beanName);  
}
```



# getSingleton()
```java
public Object getSingleton(String beanName) {  
   return getSingleton(beanName, true);  
}
protected Object getSingleton(String beanName, boolean allowEarlyReference) {  
   // Quick check for existing instance without full singleton lock  
   Object singletonObject = this.singletonObjects.get(beanName);  
   if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {  
      singletonObject = this.earlySingletonObjects.get(beanName);  
      if (singletonObject == null && allowEarlyReference) {  
         synchronized (this.singletonObjects) {  
            // Consistent creation of early reference within full singleton lock  
            singletonObject = this.singletonObjects.get(beanName);  
            if (singletonObject == null) {  
               singletonObject = this.earlySingletonObjects.get(beanName);  
               if (singletonObject == null) {  
                  ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);  
                  if (singletonFactory != null) {  
                     singletonObject = singletonFactory.getObject();  
                     this.earlySingletonObjects.put(beanName, singletonObject);  
                     this.singletonFactories.remove(beanName);  
                  }  
               }  
            }  
         }  
      }  
   }  
   return singletonObject;  
}
```

```java
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {  
   if (factory.isSingleton() && containsSingleton(beanName)) {  
      synchronized (getSingletonMutex()) {  
         Object object = this.factoryBeanObjectCache.get(beanName);  
         if (object == null) {  
            object = doGetObjectFromFactoryBean(factory, beanName);  
            // Only post-process and store if not put there already during getObject() call above  
            // (e.g. because of circular reference processing triggered by custom getBean calls)            Object alreadyThere = this.factoryBeanObjectCache.get(beanName);  
            if (alreadyThere != null) {  
               object = alreadyThere;  
            }  
            else {  
               if (shouldPostProcess) {  
                  if (isSingletonCurrentlyInCreation(beanName)) {  
                     // Temporarily return non-post-processed object, not storing it yet..  
                     return object;  
                  }  
                  beforeSingletonCreation(beanName);  
                  try {  
                     object = postProcessObjectFromFactoryBean(object, beanName);  
                  }  
                  catch (Throwable ex) {  
                     throw new BeanCreationException(beanName,  
                           "Post-processing of FactoryBean's singleton object failed", ex);  
                  }  
                  finally {  
                     afterSingletonCreation(beanName);  
                  }  
               }  
               if (containsSingleton(beanName)) {  
                  this.factoryBeanObjectCache.put(beanName, object);  
               }  
            }  
         }  
         return object;  
      }  
   }  
   else {  
      Object object = doGetObjectFromFactoryBean(factory, beanName);  
      if (shouldPostProcess) {  
         try {  
            object = postProcessObjectFromFactoryBean(object, beanName);  
         }  
         catch (Throwable ex) {  
            throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);  
         }  
      }  
      return object;  
   }  
}
```
# isFactoryBean()
```java
/*
	class: AbstractBeanFactory
*/
public boolean isFactoryBean(String name) .. {  
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
```

# getObjectFromFactoryBean()
```java
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {  
   if (factory.isSingleton() && containsSingleton(beanName)) {  
      synchronized (getSingletonMutex()) {  
         Object object = this.factoryBeanObjectCache.get(beanName);  
         if (object == null) {  
            object = doGetObjectFromFactoryBean(factory, beanName);  
            // Only post-process and store if not put there already during getObject() call above  
            // (e.g. because of circular reference processing triggered by custom getBean calls)            Object alreadyThere = this.factoryBeanObjectCache.get(beanName);  
            if (alreadyThere != null) {  
               object = alreadyThere;  
            }  
            else {  
               if (shouldPostProcess) {  
                  if (isSingletonCurrentlyInCreation(beanName)) {  
                     // Temporarily return non-post-processed object, not storing it yet..  
                     return object;  
                  }  
                  beforeSingletonCreation(beanName);  
                  try {  
                     object = postProcessObjectFromFactoryBean(object, beanName);  
                  }  
                  catch (Throwable ex) {  
                     throw new BeanCreationException(beanName,  
                           "Post-processing of FactoryBean's singleton object failed", ex);  
                  }  
                  finally {  
                     afterSingletonCreation(beanName);  
                  }  
               }  
               if (containsSingleton(beanName)) {  
                  this.factoryBeanObjectCache.put(beanName, object);  
               }  
            }  
         }  
         return object;  
      }  
   }  
   else {  
      Object object = doGetObjectFromFactoryBean(factory, beanName);  
      if (shouldPostProcess) {  
         try {  
            object = postProcessObjectFromFactoryBean(object, beanName);  
         }  
         catch (Throwable ex) {  
            throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);  
         }  
      }  
      return object;  
   }  
}
```

# prepareMethodOverrides()
```java
public void prepareMethodOverrides() throws BeanDefinitionValidationException {  
    // Check that lookup methods exist and determine their overloaded status.  
    if (hasMethodOverrides()) {  
        getMethodOverrides().getOverrides().forEach(this::prepareMethodOverride);  
    }  
}
protected void prepareMethodOverride(MethodOverride mo) throws BeanDefinitionValidationException {  
    int count = ClassUtils.getMethodCountForName(getBeanClass(), mo.getMethodName());  
    if (count == 0) {  
	    throw new BeanDefinitionValidationException(  
            "Invalid method override: no method with name '" + mo.getMethodName() +  
            "' on class [" + getBeanClassName() + "]");  
    }  
    else if (count == 1) {  
        // Mark override as not overloaded, to avoid the overhead of arg type checking.  
        mo.setOverloaded(false);  
    }  
}
```
相关方法--
```java
// class: ClassUtils
public static int getMethodCountForName(Class<?> clazz, String methodName) {  
	... // Assert.notNull
    int count = 0;  
    Method[] declaredMethods = clazz.getDeclaredMethods();  
    for (Method method : declaredMethods) {  
	    if (methodName.equals(method.getName())) {  
            count++;  
        }   
    }  
    Class<?>[] ifcs = clazz.getInterfaces();  
    for (Class<?> ifc : ifcs) {  
        count += getMethodCountForName(ifc, methodName);  
    }  
    if (clazz.getSuperclass() != null) {  
        count += getMethodCountForName(clazz.getSuperclass(), methodName);  
    }  
    return count;  
}
```

```java

```
