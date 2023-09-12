
```java
public static Properties loadProperties(Resource resource) throws IOException {  
    Properties props = new Properties();  
    fillProperties(props, resource);  
    return props;  
}
```

```java
public static void fillProperties(Properties props, Resource resource) throws IOException {  
    try (InputStream is = resource.getInputStream()) {  
        String filename = resource.getFilename();  
        if (filename != null && filename.endsWith(XML_FILE_EXTENSION)) {  
            ...
            props.loadFromXML(is);  
        }  
        else {
            props.load(is);  
        }  
    }  
}
```