
知识点：
- @ConponentScan能通过@ConponentScans注解重复标注。
- @ComponentScan的解析本质上是由ClassPathBeanDefinitionScanner完成。
- 通过@ComponentScan注册的BeanDefinitionHolder中的BeanDefinition默认是ScannedGenericBeanDefinition。

# 属性

| 属性                 | 类型         | 描述                                                       | 特点                                  | 默认值 |
| -------------------- | ------------ | ---------------------------------------------------------- | ------------------------------------- | ------ |
| value()              | `String[]`   | basePackages的别名,具体看basePackages                      |                                       |        |
| basePackages()       | `String[]`   | - value的别名</br>- 和basePackageClasses一起用于被扫描的包 | - 支持占位符</br>- 支持'/'和'.'分隔符 |        |
| basePackageClasses() | `Class<?>[]` |                                                            |                                       |        |
| includeFilters()     | `Filter[]`   |                                                            |                                       |        |
| excludeFilters()     | `Filter[]`   |                                                            |                                       |        |
|                      |              |                                                            |                                       |        |


# 条件

根据ConfigurationClassPostProcessor（一个内置BeanDefinitionRegistryPostProcessor实现类）处理@ComponentScan的逻辑来看，扫描的包中被注册到容器中需要满足下面的条件：
- 不能满足excludeFilters()属性中的任何一个。（必须满足）
- 需要满足includeFilters()属性中的任何一个，同时，满足@Conditional注解。（必须满足）
- 需要是顶层类或静态内部类，即能够单独实例化的类。（和下面的条件满足一个）
- 标注为抽象，但是内部有@Lookup方法。（和上面的条件满足一个）

# 设置的BeanDefinitionHolder的属性

beanName：
scope：如果类上标注了@Scope注解。
如果BeanDefinition是AbstractBeanDefinition（），设置：
LazyInit（前面）
自动装配模式
依赖性检查
初始化方法名称
强制使用初始化方法
销毁方法名称
强制使用销毁方法
根据ClassPathBeanDefinitionScanner的
Primary
DependsOn
Role
Description