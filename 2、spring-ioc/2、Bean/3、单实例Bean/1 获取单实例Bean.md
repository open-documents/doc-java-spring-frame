






AbstractBeanFactory的父父类DefaultSingletonBeanRegistry提供了两个获取单实例的重载方法getSingleton()。
# getSingleton() -- 1
```java
// ---------------------------------- DefaultSingletonBeanRegistry ---------------------------------- 
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

# getSingleton() -- 2
```java
// ---------------------------------- DefaultSingletonBeanRegistry ----------------------------------
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {  
	... // assert
    synchronized (this.singletonObjects) {  
	    Object singletonObject = this.singletonObjects.get(beanName);  
	    if (singletonObject == null) {  
		    // this.singletonsCurrentlyInDestruction: 默认初值为false, 不支持set方法, 只能在固定方法中改变
	        if (this.singletonsCurrentlyInDestruction) {  
	            throw new BeanCreationNotAllowedException("Singleton bean creation not allowed while singletons of this factory are in destruction (Do not request a bean from a BeanFactory in a destroy method implementation!)");  
            }  
	        ... // log: "Creating shared instance of singleton bean(beanName)"

			
            beforeSingletonCreation(beanName);  
            boolean newSingleton = false;  
            boolean recordSuppressedExceptions = (this.suppressedExceptions == null);  
		    if (recordSuppressedExceptions) {  
	            this.suppressedExceptions = new LinkedHashSet<>();  
            }  
	        try {  
				// 1. 调用 createBean(beanName, mbd, args), 创建(单)实例对象
				// 2. 
	            singletonObject = singletonFactory.getObject();  
	            newSingleton = true;  
            }  
		    catch (IllegalStateException ex) {  
	            // Has the singleton object implicitly appeared in the meantime ->  
	            // if yes, proceed with it since the exception indicates that state.    
	            singletonObject = this.singletonObjects.get(beanName);  
	            if (singletonObject == null) {  
	                throw ex;  
                }  
            }  
	        catch (BeanCreationException ex) {  
	            if (recordSuppressedExceptions) {  
	                for (Exception suppressedException : this.suppressedExceptions) {  
	                    ex.addRelatedCause(suppressedException);  
                    }  
                }  
	            throw ex;  
            }  
	        finally {  
	            if (recordSuppressedExceptions) {  
	                this.suppressedExceptions = null;  
	            }  
	            afterSingletonCreation(beanName);  
            }  
		    if (newSingleton) {  
	            addSingleton(beanName, singletonObject);  
            }  
        }  
        return singletonObject;  
    }  
}
```
## beforeSingletonCreation()
```java
protected void beforeSingletonCreation(String beanName) {  
	// 1.
	// 2. this.singletonsCurrentlyInCreation:
	// - 类型 Set<String>
	// - 默认初值 Collections.newSetFromMap(new ConcurrentHashMap<>(16));
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {  
        throw new BeanCurrentlyInCreationException(beanName);  
    }   
}
```
## afterSingletonCreation()
```java
protected void afterSingletonCreation(String beanName) {  
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.remove(beanName)) {  
        throw new IllegalStateException("Singleton '" + beanName + "' isn't currently in creation");  
    }  
}
```

## addSingleton()
```java
protected void addSingleton(String beanName, Object singletonObject) {  
    synchronized (this.singletonObjects) {  
        this.singletonObjects.put(beanName, singletonObject);  
        this.singletonFactories.remove(beanName);  
        this.earlySingletonObjects.remove(beanName);  
        this.registeredSingletons.add(beanName);  
    }  
}
```