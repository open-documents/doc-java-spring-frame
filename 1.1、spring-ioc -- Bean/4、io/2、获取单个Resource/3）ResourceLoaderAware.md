
同其他Aware一样，表明组件内部希望得到一个ResourceLoader实例的引用。下面是定义：
```java
public interface ResourceLoaderAware {   
	void setResourceLoader(ResourceLoader resourceLoader);
}
```

# 默认提供

因为ApplicationContext实现了ResourceLoader接口，默认提供的实例就是ApplicationContext。

通过@Autowired注解，能够将其注入到属性、构造函数、setter函数、其他任意包含ResourceLoader参数的方法之中。

