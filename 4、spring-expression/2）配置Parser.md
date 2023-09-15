与该部分有关的类：`org.springframework.expression.spel.SpelParserConfiguration`

# 一、配置解析行为

	参考文档：P282~P284

使用SpelParserConfiguration能够控制解析一些表示式时的行为。

1）当索引数组或集合时元素为null，能够自动创建元素。
2）当索引数组或集合时超过了数组或集合的容量，能够自动扩充容量。补充的元素先使用元素默认的构造函数创建，如果元素类没有默认的构造函数，则用null填充。

```java
class Demo {   
	public List<String> list;
} 
// Turn on: 
// - auto null reference initialization 
// - auto collection growing 
SpelParserConfiguration config = new SpelParserConfiguration(true, true); 
ExpressionParser parser = new SpelExpressionParser(config); 
Expression expression = parser.parseExpression("list[3]"); 
Demo demo = new Demo(); 
Object o = expression.getValue(demo); 
// demo.list will now be a real collection of 4 entries 
// Each entry is a new empty String
```

-- --
# 二、配置编译器模式

	参考文档：P284

Spring4.1引入了基础的表达式编译器。

解释模式：表达式一般采用被解释的方式，解释模式提供了动态灵活性，但是性能不是最优。

编译模式：编译器产生体现表达式行为的java字节码，从而达到更快的效果。


## 解析器编译模式

默认情况下，编译器是不打开的。

org.springframework.expression.spel.SpelCompilerMode Enum列出了编译器工作的三种模式：
- OFF：编译器关闭。
- IMMEDIATE：在第一次解释后，立刻对表达式进行编译。在此模式下发生错误（如类型转换），表达式计算的调用者能够捕获到异常。
- MIXED：在数次解释后，编译器转成编译模式。如果错误发生（如类型转换失败），编译器返回解释模式。

IMMEDIATE之所以存在，是因为MIXED存在一定的副作用：编译的表达式部分成功，用户不会想编译器重新运行在解释模式，因为这可能导致表达式的部分被执行两次。

## 配置解析器编译模式

```java
SpelParserConfiguration config = new SpelParserConfiguration(SpelCompilerMode.IMMEDIATE,   this.getClass().getClassLoader()); 
SpelExpressionParser parser = new SpelExpressionParser(config); 
Expression expr = parser.parseExpression("payload"); 
MyMessage message = new MyMessage(); 
Object payload = expr.getValue(message);
```