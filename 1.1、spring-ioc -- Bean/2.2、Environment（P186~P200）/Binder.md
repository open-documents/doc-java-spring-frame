```java
Binder binder = Binder.get(environment);  
return binder.bind(PROPERTY_NAME_AUTOCONFIGURE_EXCLUDE, String[].class).map(Arrays::asList)  
      .orElse(Collections.emptyList());
```