
和接口ApplicationContext有关的属性：
```java
private String id = ObjectUtils.identityToString(this);
private String displayName = ObjectUtils.identityToString(this);
private ApplicationContext parent;
private long startupDate;


private MessageSource messageSource;


```

和接口ConfigurableApplicationContext有关的属性：
```java
@Nullable  
private ConfigurableEnvironment environment;

private Thread shutdownHook;

private ApplicationStartup applicationStartup = ApplicationStartup.DEFAULT;

private final List<BeanFactoryPostProcessor> beanFactoryPostProcessors = new ArrayList<>();





private final AtomicBoolean active = new AtomicBoolean();
private final AtomicBoolean closed = new AtomicBoolean();


```
和父类有关属性：
```java
private ResourcePatternResolver resourcePatternResolver;
```


自定义属性：
```java


private LifecycleProcessor lifecycleProcessor;
```

## this.startupDate

getter:普通

setter方法，所以startupDate不能设置。

与之有关的方法：
prepareRefresh()
```java
protected void prepareRefresh() {
	this.startupDate = System.currentTimeMillis();
}
```
-- --
## this.environment

setter: 普通
- Default value is determined by createEnvironment(). Replacing the default with this method is one option but configuration through getEnvironment() should also be considered. In either case, such modifications should be performed before refresh().
getter:
```java
@Override  
public ConfigurableEnvironment getEnvironment() {  
   if (this.environment == null) {  
      this.environment = createEnvironment();  // 默认实现: return new StandardEnvironment();
   }  
   return this.environment;  
}
```
与之相关的方法：
initPropertySources()：在prepareRefresh()中被调用。
```java
protected void prepareRefresh(){
	... 
	// Initialize any placeholder property sources in the context environment.
	initPropertySources();
	// Validate that all properties marked as required are resolvable:  
    // see ConfigurablePropertyResolver#setRequiredProperties
	getEnvironment().validateRequiredProperties();
}
```






# 与ApplicationEventPublisher接口有关

## 与之相关的属性
```java
private ApplicationEventMulticaster applicationEventMulticaster;
```
this.applicationEventMulticaster：
- 普通的getter方法：getApplicationEventMulticaster()
- 和setter方法

```java
private Set<ApplicationEvent> earlyApplicationEvents;
private final Set<ApplicationListener<?>> applicationListeners = new LinkedHashSet<>();
private Set<ApplicationListener<?>> earlyApplicationListeners;

```
prepareRefresh()方法在refresh()方法中被调用。
```java
protected void prepareRefresh() {
	...
	// Store pre-refresh ApplicationListeners...  
	if (this.earlyApplicationListeners == null) {  
	    this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);  
    }  
	else {  // Reset local application listeners to pre-refresh state.  
	    this.applicationListeners.clear();  
	    this.applicationListeners.addAll(this.earlyApplicationListeners);  
	}  
	// Allow for the collection of early ApplicationEvents,  
	// to be published once the multicaster is available...  
	this.earlyApplicationEvents = new LinkedHashSet<>();
}
```
## 实现
```java
@Override
public void publishEvent(ApplicationEvent event) {  
   publishEvent(event, null);  
}
@Override
public void publishEvent(Object event) {  
   publishEvent(event, null);  
}
```
真正的逻辑：
将事件的发布交给真正的this.applicationEventMulticaster（类型ApplicationEventMulticaster）来完成。
```java
protected void publishEvent(Object event, @Nullable ResolvableType eventType) {  
   Assert.notNull(event, "Event must not be null");  
   // Decorate event as an ApplicationEvent if necessary  
   ApplicationEvent applicationEvent;  
   if (event instanceof ApplicationEvent) {  
      applicationEvent = (ApplicationEvent) event;  
   }  
   else {  
      applicationEvent = new PayloadApplicationEvent<>(this, event);  
      if (eventType == null) {  
         eventType = ((PayloadApplicationEvent<?>) applicationEvent).getResolvableType();  
      }  
   }  
  
   // Multicast right now if possible - or lazily once the multicaster is initialized  
   if (this.earlyApplicationEvents != null) {  
      this.earlyApplicationEvents.add(applicationEvent);  
   }  
   else {  
      getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);  
   }  
  
   // Publish event via parent context as well...  
   if (this.parent != null) {  
      if (this.parent instanceof AbstractApplicationContext) {  
         ((AbstractApplicationContext) this.parent).publishEvent(event, eventType);  
      }  
      else {  
         this.parent.publishEvent(event);  
      }  
   }  
}
```




