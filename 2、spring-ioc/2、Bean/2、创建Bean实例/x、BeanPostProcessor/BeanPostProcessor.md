首先介绍BeanPostProcessor接口的定义：
```java
public interface BeanPostProcessor {
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```
BeanPostProcessor在bean的创建过程中所处的位置：#生命周期概览。
BeanPostProcessor通常用来1）检查回调接口；2）将bean封装成proxy（SpringAOP的一些基础类就是通过BeanPostProcessor实现的）。

BeanPostProcessor和它们所直接引用到的类会被容器区别对待，它们在启动阶段就被初始化，然后被应用到其他后续的普通Bean上。由于SpringAOP是通过BeanPostProcessor实现的，所以BeanPostProcessor和它们直接引用的类是无法被代理的（neither BeanPostProcessor instances nor the beans they directly reference are eligible for auto-proxying and, thus, do not have aspects woven into them.）。



# 自定义BeanPostProcessor


# 注册BeanPostProcessor

## ApplicationContext（推荐）

ApplicationContext自动检测在Configuration Definition中定义了的BeanPostProcessor接口实现类，并将这些bean注册为BeanPostProcessor。BeanPostProcessor类的bean定义和普通bean定义类似。

通过@Bean的方法定义，返回类型必须是实现类本身或至少是BeanPostProcessor接口类型，否则ApplicationContext在完全创建出对象前是无法通过类型检测到它的。然后BeanPostProcessor的早初始化是极其必要的，因为要将其应用到容器中其他bean的初始化过程中，所以早一点需要尽早完成类型检测。

## ConfigurableBeanFactory

通过ConfigurableBeanFactory的 `addBeanPostProcessor() `方法。

适用场景：
- 当需要在注册BeanPostProcessor之前进行部分逻辑评估
- for copying bean post processors across contexts in a hierarchy.


# 多个BeanPostProcessor执行顺序

首先执行的是：通过ConfigurableBeanFactory显示注册的。在显示注册内部按照注册顺序依次执行。







