有些Resource接口的实现类还实现了WritableResource接口以支持对资源的写操作。

# 一、UrlResource

UrlResource包装了 `java.net.URL`，能够用它访问正常能通过URL访问的对象，如文件，HTTPS对象，FTP对象。

所有的URL有一个标准的String类型格式，以便合适的格式前缀能够用来表示URL的类型，如：

- `file:`：通过文件系统路径访问资源的URL。
- `https:`：通过https协议访问资源的URL。
- `ftp:`：通过ftp协议访问资源的URL。

URLResource可以通过构造器显示地构造，但是一般都是通过调用一个表示路径的String参数的方法来隐式地构造，在这种（后者）情况下，PropertyEditor最终决定构造哪种类型的Resource，即如果String类型的参数的前缀（前面提到的几种）能够被识别，就根据前缀来确定，如果不能识别，就将其认为是普通的URL字符串然后创建一个URLResource。

# 二、FileSystemResource


当单独使用FileSystemResource，传入的String path的绝对路径和相对路径是按正常来表现。

当FileSystemResource和FileSystemApplicationContext配合使用时，都按相对路径处理（不管路径是否以 "/" 开始）。举个例子：

下面的写法是等价的：
```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("conf/context.xml");
ApplicationContext ctx = new FileSystemXmlApplicationContext("/conf/context.xml");
```

```java
FileSystemXmlApplicationContext ctx = ...; 
ctx.getResource("some/resource/path/myTemplate.txt");

FileSystemXmlApplicationContext ctx = ...; 
ctx.getResource("/some/resource/path/myTemplate.txt");
```

想要使绝对路径生效，使用 `file:///URL`作为参数。举个例子：
```java
// force this FileSystemXmlApplicationContext to load its definition via a UrlResource 
ApplicationContext ctx = new FileSystemXmlApplicationContext("file:///conf/context.xml");
// actual context type doesn't matter, the Resource will always be UrlResource 
ctx.getResource("file:///some/resource/path/myTemplate.txt");
```