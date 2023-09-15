
# 属性


AbstractTypeHierarchyTraversingFilter是

```java
public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)  
      throws IOException {  
  
    // This method optimizes avoiding unnecessary creation of ClassReaders  
    // as well as visiting over those readers.   
    // 默认实现是直接返回false.
    if (matchSelf(metadataReader)) {  
        return true;  
    }  
    ClassMetadata metadata = metadataReader.getClassMetadata();  
    // 默认实现是直接返回false,支持重写,重写了该方法的子类:
    //   - AssignableTypeFilter
    if (matchClassName(metadata.getClassName())) {  
        return true;  
    }  
  
    if (this.considerInherited) {  
        String superClassName = metadata.getSuperClassName();  
        if (superClassName != null) {  
            // Optimization to avoid creating ClassReader for superclass.  
            // 默认返回null,支持重写,重写该方法的子类:
            //  - AssignableTypeFilter
            Boolean superClassMatch = matchSuperClass(superClassName);  
            if (superClassMatch != null) {  
	            if (superClassMatch.booleanValue()) {  
	                return true;  
                }  
            }  
            else {  // Need to read superclass to determine a match... 
	            try {  
		            if (match(metadata.getSuperClassName(), metadataReaderFactory)) {  
		                return true;  
	                }  
	            }  
	            catch (IOException ex) {  ...  }  
            }  
        }  
    }  
  
   if (this.considerInterfaces) {  
      for (String ifc : metadata.getInterfaceNames()) {  
         // Optimization to avoid creating ClassReader for superclass  
         Boolean interfaceMatch = matchInterface(ifc);  
         if (interfaceMatch != null) {  
            if (interfaceMatch.booleanValue()) {  
               return true;  
            }  
         }  
         else {  
            // Need to read interface to determine a match...  
            try {  
               if (match(ifc, metadataReaderFactory)) {  
                  return true;  
               }  
            }  
            catch (IOException ex) {  
               if (logger.isDebugEnabled()) {  
                  logger.debug("Could not read interface [" + ifc + "] for type-filtered class [" +  
                        metadata.getClassName() + "]");  
               }  
            }  
         }  
      }  
   }  
  
   return false;  
}
```