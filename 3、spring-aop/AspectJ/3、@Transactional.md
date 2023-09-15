	参考文档:P387

包含在Spring-aspects.jar中。

解释@Transactional注解的切面是AnnotationTransactionAspect。

当使用这个切面时，必须将@Transactional注解标注在实现类或者实现类的方法上。

AspectJ遵循Java规则，即接口上的注解是不能被继承的。