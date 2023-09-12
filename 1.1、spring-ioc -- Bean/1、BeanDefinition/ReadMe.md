
整个BeanDefinition体系使用在BeanDefinitionRegistry接口的体系中，而BeanDefinitionRegistry接口的实现类是Spring应用（非Springboot应用）的启动类AnnotationConfigApplicationContext。


# 总体继承关系

下面是总体继承关系。
<img src="D:\Obsidian\note_obsidian\JAVA\Java框架\1、springframe\1、IOC\1.1、BeanDefinition\pic\Pasted image 20230527195820.png" alt="图片不能加载"/> 


# 小知识点1

对于AnnotatedGenericBeanDefinition，getIntrospectedClass()方法和AbstractBeanDefinition的getBeanClass()方法返回的Class是同一个。
<img src="D:\Obsidian\note_obsidian\JAVA\Java框架\1、springframe\1、IOC\1.1、BeanDefinition\pic\Pasted image 20230601224640.png" alt="图片不能加载"/> 


