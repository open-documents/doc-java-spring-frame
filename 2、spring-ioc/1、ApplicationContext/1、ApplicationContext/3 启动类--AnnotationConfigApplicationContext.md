
<font color=44cf57>一般来说，AnnotationConfigApplicationContext会以配置类（标注@Configuration的类）的方式创建ApplicationContext</font>，但是AnnotationConfigApplicationContext的构造函数也提供扫描包的方式来创建ApplicationContext。



基本使用如下：
```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(xxx.class)
```
# 1、启动流程

普通的Spring应用（非Springboot应用）常见的启动是这样的。
```java
AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(xxx.class);
```
然后开始使用。由此可见重要的流程都在该类的构造函数中。
# 2、构造函数

AnnotationConfigApplicationContext有4个构造函数，分别如下：
```java
// 两个属性都在构造函数中实例化并初始化
private final AnnotatedBeanDefinitionReader reader;  
private final ClassPathBeanDefinitionScanner scanner;

public AnnotationConfigApplicationContext(){
	... 
	this.reader = new AnnotatedBeanDefinitionReader(this);  
	...
	this.scanner = new ClassPathBeanDefinitionScanner(this);
}
public AnnotationConfigApplicationContext(DefaultListableBeanFactory beanFactory){
	super(beanFactory);  
	this.reader = new AnnotatedBeanDefinitionReader(this);  
	this.scanner = new ClassPathBeanDefinitionScanner(this);
}
public AnnotationConfigApplicationContext(Class<?>... componentClasses){
	this();
	register(componentClasses);  
	refresh();
}
public AnnotationConfigApplicationContext(String... basePackages){
	this();  
	scan(basePackages);  
	refresh();
}
```
## 相关方法--register()
```java
// --------------------- AnnotationConfigApplicationContext ---------------------------
public void register(Class<?>... componentClasses) {  
    ...
    // xxx: "spring.context.component-classes.register"
    StartupStep registerComponentClass = this.getApplicationStartup().start(xxx)  
         .tag("classes", () -> Arrays.toString(componentClasses));  
    this.reader.register(componentClasses);  
    registerComponentClass.end();  
}
```
## 相关方法--scan()
```java
public void scan(String... basePackages) {  
    ... 
    // xxx: "spring.context.base-packages.scan"
    StartupStep scanPackages = this.getApplicationStartup().start(xxx)  
         .tag("packages", () -> Arrays.toString(basePackages));  
    this.scanner.scan(basePackages);  
    scanPackages.end();  
}
```






