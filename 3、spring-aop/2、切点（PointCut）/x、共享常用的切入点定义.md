
```java
package com.xyz.myapp;

@Aspect 
public class CommonPointcuts {   
	/**   
	 * A join point is in the web layer if the method is defined   
	 * in a type in the com.xyz.myapp.web package or any sub-package   
	 * under that.   
	 **/   
	 @Pointcut("within(com.xyz.myapp.web..*)")
	public void inWebLayer() {}   
	/**   
	 * A join point is in the service layer if the method is defined   
	 * in a type in the com.xyz.myapp.service package or any sub-package   
	 * under that.   
	 **/   
	 @Pointcut("within(com.xyz.myapp.service..*)")   
	 public void inServiceLayer() {}   
	 /**   
	  * A join point is in the data access layer if the method is defined   
	  * in a type in the com.xyz.myapp.dao package or any sub-package   
	  * under that.   
	  **/   
	  @Pointcut("within(com.xyz.myapp.dao..*)")   
	  public void inDataAccessLayer() {}   
	  /**  
	   * A business service is the execution of any method defined on a service   
	   * interface. This definition assumes that interfaces are placed in the   
	   * "service" package, and that implementation types are in sub-packages.  
	   *
	   * If you group service interfaces by functional area (for example,   
	   * in packages com.xyz.myapp.abc.service and com.xyz.myapp.def.service) then   
	   * the pointcut expression "execution(* com.xyz.myapp..service.*.*(..))"   
	   * could be used instead.  
	   *   
	   * Alternatively, you can write the expression using the 'bean'   
	   * PCD, like so "bean(*Service)". (This assumes that you have   
	   * named your Spring service beans in a consistent fashion.)   
	   **/   
	   @Pointcut("execution(* com.xyz.myapp..service.*.*(..))")   
	   public void businessService() {}   
	   /**
	    * A data access operation is the execution of any method defined on a   
	    * dao interface. This definition assumes that interfaces are placed in the   
	    * "dao" package, and that implementation types are in sub-packages.   
	    **/   
	    @Pointcut("execution(* com.xyz.myapp.dao.*.*(..))")   
	    public void dataAccessOperation() {}
}
```

引用
```xml
<aop:config>   
<aop:advisor   
	pointcut="com.xyz.myapp.CommonPointcuts.businessService()"   
	advice-ref="tx-advice"/> 
</aop:config>
<tx:advice id="tx-advice">
	<tx:attributes>
	<tx:method name="*" propagation="REQUIRED"/>
	</tx:attributes>
</tx:advice>
```