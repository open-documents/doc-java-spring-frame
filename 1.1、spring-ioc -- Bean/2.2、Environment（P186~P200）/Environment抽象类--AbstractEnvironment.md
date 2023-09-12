
# 构造函数

```java
public AbstractEnvironment() {  
    this(new MutablePropertySources());  
}
protected AbstractEnvironment(MutablePropertySources propertySources) {  
    this.propertySources = propertySources;  
    this.propertyResolver = createPropertyResolver(propertySources);  
    customizePropertySources(propertySources);  
}
// 
protected ConfigurablePropertyResolver createPropertyResolver(MutablePropertySources propertySources) {  
    return new PropertySourcesPropertyResolver(propertySources);  
}
```


下面的方法都交由PropertyResolver（默认实现类是在构造函数中通过createPropertyResolver()方法提供的PropertySourcesPropertyResolver）实现。
# 代为实现方法--resolvePlaceholders()
```java
public String resolvePlaceholders(String text) {  
    return this.propertyResolver.resolvePlaceholders(text);  
}
```

# 代为实现方法--resolveRequiredPlaceholders()

下面是该方法的定义，可以看到
- resolveRequiredPlaceholders()方法由父类AbstractEnvironment提供。
- AbstractEnvironment交由属性ConfigurablePropertyResolver（更具体，实现类PropertySourcesPropertyResolver）完成，而ConfigurablePropertyResolver进一步交给它的属性PropertyPlaceholderHelper完成，即该方法是交由PropertyPlaceholderHelper完成。

下面具体展开介绍。
1）AbstractEnvironment交由属性ConfigurablePropertyResolver（更具体，实现类PropertySourcesPropertyResolver）完成。
```java
// --------------------------- AbstractEnvironment -----------------------------------------------
public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {  
    return this.propertyResolver.resolveRequiredPlaceholders(text);  
}
```
属性ConfigurablePropertyResolver是在构造函数中实例化。
```java
protected AbstractEnvironment(MutablePropertySources propertySources) {  
    this.propertySources = propertySources;  
    /* .. ConfigurablePropertyResolver createPropertyResolver(MutablePropertySources propertySources){
     *     return new PropertySourcesPropertyResolver(propertySources);  
     * }
     */
    this.propertyResolver = createPropertyResolver(propertySources);  
    customizePropertySources(propertySources);  
}
```
2）ConfigurablePropertyResolver进一步交给它的属性PropertyPlaceholderHelper完成。
```java
// ---------------------- PropertySourcesPropertyResolver.resolveRequiredPlaceholders() --------------
public String resolveRequiredPlaceholders(String text) throws IllegalArgumentException {  
    if (this.strictHelper == null) {  
		/* .. PropertyPlaceholderHelper createPlaceholderHelper(boolean ignoreUnresolvablePlaceholders){
		 *     return new PropertyPlaceholderHelper(this.placeholderPrefix, this.placeholderSuffix,  
         *         this.valueSeparator, ignoreUnresolvablePlaceholders);
         * }
		 */ 
	    this.strictHelper = createPlaceholderHelper(false);  
    }  
    return doResolvePlaceholders(text, this.strictHelper);  
}
private String doResolvePlaceholders(String text, PropertyPlaceholderHelper helper) {  
    return helper.replacePlaceholders(text, this::getPropertyAsRawString);  
}
```

最后，来看下<font color=44cf57>真正完成该功能的方法</font>。
```java
// ----------------------------- PropertyPlaceholderHelper -------------------------------------------
public String replacePlaceholders(String value, PlaceholderResolver placeholderResolver) {  
    ... 
    return parseStringValue(value, placeholderResolver, null);  
}
protected String parseStringValue(String value, PlaceholderResolver placeholderResolver, @Nullable Set<String> visitedPlaceholders) {  
  
    int startIndex = value.indexOf(this.placeholderPrefix);  
    if (startIndex == -1) {  
        return value;  
    }  
  
   StringBuilder result = new StringBuilder(value);  
   while (startIndex != -1) {  
      int endIndex = findPlaceholderEndIndex(result, startIndex);  
      if (endIndex != -1) {  
         String placeholder = result.substring(startIndex + this.placeholderPrefix.length(), endIndex);  
         String originalPlaceholder = placeholder;  
         if (visitedPlaceholders == null) {  
            visitedPlaceholders = new HashSet<>(4);  
         }  
         if (!visitedPlaceholders.add(originalPlaceholder)) {  
            throw new IllegalArgumentException(  
                  "Circular placeholder reference '" + originalPlaceholder + "' in property definitions");  
         }  
         // Recursive invocation, parsing placeholders contained in the placeholder key.  
         placeholder = parseStringValue(placeholder, placeholderResolver, visitedPlaceholders);  
         // Now obtain the value for the fully resolved key...  
         String propVal = placeholderResolver.resolvePlaceholder(placeholder);  
         if (propVal == null && this.valueSeparator != null) {  
            int separatorIndex = placeholder.indexOf(this.valueSeparator);  
            if (separatorIndex != -1) {  
               String actualPlaceholder = placeholder.substring(0, separatorIndex);  
               String defaultValue = placeholder.substring(separatorIndex + this.valueSeparator.length());  
               propVal = placeholderResolver.resolvePlaceholder(actualPlaceholder);  
               if (propVal == null) {  
                  propVal = defaultValue;  
               }  
            }  
         }  
         if (propVal != null) {  
            // Recursive invocation, parsing placeholders contained in the  
            // previously resolved placeholder value.            propVal = parseStringValue(propVal, placeholderResolver, visitedPlaceholders);  
            result.replace(startIndex, endIndex + this.placeholderSuffix.length(), propVal);  
            if (logger.isTraceEnabled()) {  
               logger.trace("Resolved placeholder '" + placeholder + "'");  
            }  
            startIndex = result.indexOf(this.placeholderPrefix, startIndex + propVal.length());  
         }  
         else if (this.ignoreUnresolvablePlaceholders) {  
            // Proceed with unprocessed value.  
            startIndex = result.indexOf(this.placeholderPrefix, endIndex + this.placeholderSuffix.length());  
         }  
         else {  
            throw new IllegalArgumentException("Could not resolve placeholder '" +  
                  placeholder + "'" + " in value \"" + value + "\"");  
         }  
         visitedPlaceholders.remove(originalPlaceholder);  
      }  
      else {  
         startIndex = -1;  
      }  
   }  
   return result.toString();  
}
```
