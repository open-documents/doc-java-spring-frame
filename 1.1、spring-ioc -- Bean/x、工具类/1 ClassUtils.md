
# forName()
```java
public static Class<?> forName(String name, @Nullable ClassLoader classLoader)  
      throws ClassNotFoundException, LinkageError {  
  
   Class<?> clazz = resolvePrimitiveClassName(name);  
   if (clazz == null) {  
      clazz = commonClassCache.get(name);    // commonClassCache : Map<String, Class<?>>
   }  
   if (clazz != null) {  
      return clazz;  
   }  
  
   // "java.lang.String[]" style arrays  
   if (name.endsWith(ARRAY_SUFFIX)) {  
      String elementClassName = name.substring(0, name.length() - ARRAY_SUFFIX.length());  
      Class<?> elementClass = forName(elementClassName, classLoader);  
      return Array.newInstance(elementClass, 0).getClass();  
   }  
  
   // "[Ljava.lang.String;" style arrays  
   if (name.startsWith(NON_PRIMITIVE_ARRAY_PREFIX) && name.endsWith(";")) {  
      String elementName = name.substring(NON_PRIMITIVE_ARRAY_PREFIX.length(), name.length() - 1);  
      Class<?> elementClass = forName(elementName, classLoader);  
      return Array.newInstance(elementClass, 0).getClass();  
   }  
  
   // "[[I" or "[[Ljava.lang.String;" style arrays  
   if (name.startsWith(INTERNAL_ARRAY_PREFIX)) {  
      String elementName = name.substring(INTERNAL_ARRAY_PREFIX.length());  
      Class<?> elementClass = forName(elementName, classLoader);  
      return Array.newInstance(elementClass, 0).getClass();  
   }  
  
   ClassLoader clToUse = classLoader;  
   if (clToUse == null) {  
      clToUse = getDefaultClassLoader();  
   }  
   try {  
      return Class.forName(name, false, clToUse);  
   }  
   catch (ClassNotFoundException ex) {  
      int lastDotIndex = name.lastIndexOf(PACKAGE_SEPARATOR);  
      if (lastDotIndex != -1) {  
         String nestedClassName =  
               name.substring(0, lastDotIndex) + NESTED_CLASS_SEPARATOR + name.substring(lastDotIndex + 1);  
         try {  
            return Class.forName(nestedClassName, false, clToUse);  
         }  
         catch (ClassNotFoundException ex2) {  
            // Swallow - let original exception get through  
         }  
      }  
      throw ex;  
   }  
}
```

# getPackageName()

给定全限定类名，返回其所在的包名。

```java
public static String getPackageName(String fqClassName) {  
    ... 
    // PACKAGE_SEPARATOR = '.'
    int lastDotIndex = fqClassName.lastIndexOf(PACKAGE_SEPARATOR);  
    return (lastDotIndex != -1 ? fqClassName.substring(0, lastDotIndex) : "");  
}
```