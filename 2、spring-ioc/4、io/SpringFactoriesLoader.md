
类似的一个类：springboot.ImportCandidates，它从 `META-INF/spring/类的全限定名.imports` 中导入实现类。 




# loadFactoryNames()（重要）

静态方法loadFactoryNames()做了下面的事情：
- 从--class-path指定的参数中获取 `META-INF/spring.factories`下的URL。

```java
// ------------------------------- SpringFactoriesLoader ----------------------------------------
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {  
    ClassLoader classLoaderToUse = classLoader;  
    if (classLoaderToUse == null) {  
        classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();  
    }  
    String factoryTypeName = factoryType.getName();  
    // 获取指定类factoryType的实现类的全限定名
    return loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());  
}
```
## 主要逻辑
```java
private static Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) {  
    Map<String, List<String>> result = cache.get(classLoader);  
    if (result != null) {  
        return result;  
    }  
  
    result = new HashMap<>();  
    try {  
		// FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"
        Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);  
        while (urls.hasMoreElements()) {  
            URL url = urls.nextElement();  
            UrlResource resource = new UrlResource(url);
			// 加载文件内容
            Properties properties = PropertiesLoaderUtils.loadProperties(resource);  
            // 获取接口:实现类数组的Map
            for (Map.Entry<?, ?> entry : properties.entrySet()) {  
	            String factoryTypeName = ((String) entry.getKey()).trim();  
	            // 将','分隔的字符串转换为数组
	            String[] factoryImplementationNames =  
                    StringUtils.commaDelimitedListToStringArray((String) entry.getValue());  
	            for (String factoryImplementationName : factoryImplementationNames) {  
		            result.computeIfAbsent(factoryTypeName, key -> new ArrayList<>())  
                     .add(factoryImplementationName.trim());  
	            }  
            }  
        }  
  
        // Replace all lists with unmodifiable lists containing unique elements  
        result.replaceAll((factoryType, implementations) -> implementations.stream().distinct()  
.collect(Collectors.collectingAndThen(Collectors.toList(), Collections::unmodifiableList)));  
        cache.put(classLoader, result);  
    }  
    ...
    return result;  
}
```