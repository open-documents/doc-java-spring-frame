
```java
public static ResolvableType forMethodReturnType(Method method) {  
    ... // Assert
    return forMethodParameter(new MethodParameter(method, -1));  
}
public static ResolvableType forMethodReturnType(Method method, Class<?> implementationClass) {  
    ... // Assert 
    MethodParameter methodParameter = new MethodParameter(method, -1, implementationClass);  
	return forMethodParameter(methodParameter);   
}
```

```java
public static ResolvableType forMethodParameter(Method method, int parameterIndex, Class<?> implementationClass) {  
    ... // Assert
    MethodParameter methodParameter = new MethodParameter(method, parameterIndex, implementationClass);  
    return forMethodParameter(methodParameter);  
}
public static ResolvableType forMethodParameter(Method method, int parameterIndex) {  
   Assert.notNull(method, "Method must not be null");  
   return forMethodParameter(new MethodParameter(method, parameterIndex));  
}
public static ResolvableType forMethodParameter(MethodParameter methodParameter) {  
    return forMethodParameter(methodParameter, (Type) null);  
}
public static ResolvableType forMethodParameter(MethodParameter methodParameter, @Nullable Type targetType) {  
   Assert.notNull(methodParameter, "MethodParameter must not be null");  
   return forMethodParameter(methodParameter, targetType, methodParameter.getNestingLevel());  
}
static ResolvableType forMethodParameter(  
      MethodParameter methodParameter, @Nullable Type targetType, int nestingLevel) {  
  
   ResolvableType owner = forType(methodParameter.getContainingClass()).as(methodParameter.getDeclaringClass());  
   return forType(targetType, new MethodParameterTypeProvider(methodParameter), owner.asVariableResolver()).  
         getNested(nestingLevel, methodParameter.typeIndexesPerLevel);  
}


public static ResolvableType forMethodParameter(MethodParameter methodParameter,  
      @Nullable ResolvableType implementationType) {  
  
   Assert.notNull(methodParameter, "MethodParameter must not be null");  
   implementationType = (implementationType != null ? implementationType :  
         forType(methodParameter.getContainingClass()));  
   ResolvableType owner = implementationType.as(methodParameter.getDeclaringClass());  
   return forType(null, new MethodParameterTypeProvider(methodParameter), owner.asVariableResolver()).  
         getNested(methodParameter.getNestingLevel(), methodParameter.typeIndexesPerLevel);  
}
```

```java
static ResolvableType forType(  
      @Nullable Type type, @Nullable TypeProvider typeProvider, @Nullable VariableResolver variableResolver) {  
  
   if (type == null && typeProvider != null) {  
      type = SerializableTypeWrapper.forTypeProvider(typeProvider);  
   }  
   if (type == null) {  
      return NONE;  
   }  
  
   // For simple Class references, build the wrapper right away -  
   // no expensive resolution necessary, so not worth caching...   if (type instanceof Class) {  
      return new ResolvableType(type, typeProvider, variableResolver, (ResolvableType) null);  
   }  
  
   // Purge empty entries on access since we don't have a clean-up thread or the like.  
   cache.purgeUnreferencedEntries();  
  
   // Check the cache - we may have a ResolvableType which has been resolved before...  
   ResolvableType resultType = new ResolvableType(type, typeProvider, variableResolver);  
   ResolvableType cachedType = cache.get(resultType);  
   if (cachedType == null) {  
      cachedType = new ResolvableType(type, typeProvider, variableResolver, resultType.hash);  
      cache.put(cachedType, cachedType);  
   }  
   resultType.resolved = cachedType.resolved;  
   return resultType;  
}
```