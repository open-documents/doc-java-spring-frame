FactoryBean接口的实现对象在BeanFactory中存在并且被使用。如果一个bean实现了FactoryBean接口，这个bean不会被作为实例直接返回，而是作为工厂返回其他实例对象。

接口定义如下：
```java
package org.springframework.beans.factory;

public interface FactoryBean<T> {

}
```

# 自定义FactoryBean





容器通过getBean方法来FactoryBean创建的实例时实际获取的不是FactoryBean 本身而是具体创建的T泛型实例。等下我们会来验证这个事情。
-   Class<?> getObjectType() 获取 T getObject()中的返回值 T 的具体类型。这里强烈建议如果T是一个接口，返回其具体实现类的类型。
-   default boolean isSingleton() 用来规定 Factory创建的的bean是否是单例。这里通过默认方法定义为单例。

### **3.1 FactoryBean使用场景**

FactoryBean 用来创建一类bean。比如你有一些同属鸟类的bean需要被创建，但是它们自己有各自的特点，你只需要把他们的特点注入FactoryBean中就可以生产出各种鸟类的实例。举一个更加贴近实际生产的例子。甚至这个例子你可以应用到实际java开发中去。我们需要自己造一个定时任务的轮子。用FactoryBean 再合适不过了。我们来用代码说话一步步来演示FactoryBean的使用场景。

### **3.2 构建一个FactoryBean**

我们声明定时任务一般具有下列要素：

-   时间周期，肯定会使用到cron表达式。
-   一个任务的执行抽象接口。
-   定时任务具体行为的执行者。

Task任务执行抽象接口的实现。实现包含两个方面：

-   SomeService 是具体任务的执行逻辑。
-   cron时间表达式

![](https://ask.qcloudimg.com/http-save/6430374/ge8a7e9diq.png?imageView2/2/w/1620)

通过以上的定义。任务的时间和任务的逻辑可以根据不同的业务做到差异化配置。然后我们实现一个关于Task的FactoryBean。

![](https://ask.qcloudimg.com/http-save/6430374/epfqcx7sbn.png?imageView2/2/w/1620)

### **3.3 FactoryBean 注入IoC**

你可以使用xml的注入方式，当然也可以使用javaConfig的配置方式。这里我们使用javaConfig注入。我们将两个FactroyBean注入到Spring容器中去。

![](https://ask.qcloudimg.com/http-save/6430374/7kyh6gcwfk.png?imageView2/2/w/1620)

### **3.4 FactoryBean的一些特点**

一般如上声明后，@Bean注解如果不显式声明bean名称则方法名作为bean的名称，而且返回值作为注入的Bean。但是我们通过debug发现却是这样的：

![](https://ask.qcloudimg.com/http-save/6430374/vnep5gc0hf.png?imageView2/2/w/1620)

也就是说通过方法名是返回FactoryBean 创建的Bean。那么如何返回该FactoryBean呢？上图中也给出了答案在方法前增加引用符“&”。具体的原因还用从BeanFactory中寻找，真是不是冤家不聚头。

![](https://ask.qcloudimg.com/http-save/6430374/u02lkns6y3.png?imageView2/2/w/1620)

我们对上面声明的两个bean进行测试，也出色地完成了不同的定时任务业务逻辑。

![](https://ask.qcloudimg.com/http-save/6430374/lb062picd9.png?imageView2/2/w/1620)