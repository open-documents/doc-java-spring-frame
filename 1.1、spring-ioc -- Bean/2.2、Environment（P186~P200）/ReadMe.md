
	参考文档：P186

# 总体关系继承图

<img src="D:\Obsidian\note_obsidian\JAVA\Java框架\1、springframe\1、IOC\2.2、Environment（P186~P200）\pic\Pasted image 20230607221238.png" alt="无法加载图片"/>




Environment接口模型化了应用环境的两个方面：profiles和properties。

下面是Environment接口的定义：
```java
public interface Environment extends PropertyResolver {
	String[] getActiveProfiles();
	String[] getDefaultProfiles();
	boolean acceptsProfiles(Profiles profiles);
}
```

Environment关于profile方面的角色：用于决定哪个目前哪个profile被激活，以及默认哪个profile被激活。
Environment关于properties方面的角色：为用户提供一个方便访问configuring property sources的接口以及通过它们进行解析。


## profile

profile是一个有名字的、是一组将要注册到容器中的bean definiton的逻辑组