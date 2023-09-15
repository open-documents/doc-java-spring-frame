
```java
@Override  
public Object getBean(String name) throws BeansException {  
    return doGetBean(name, null, null, false);  
}  
  
@Override  
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {  
   return doGetBean(name, requiredType, null, false);  
}  
  
@Override  
public Object getBean(String name, Object... args) throws BeansException {  
   return doGetBean(name, null, args, false);  
}

public <T> T getBean(String name, @Nullable Class<T> requiredType, @Nullable Object... args)  
      throws BeansException {  
  
   return doGetBean(name, requiredType, args, false);  
}
```