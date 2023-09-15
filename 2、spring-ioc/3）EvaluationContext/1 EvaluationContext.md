	参考文档：P281

EvaluationContext在解析表达式时会用到。

有两种EvaluationContext：SimpleEvaluationContext、StandardEvaluationContext。

涉及到表达式模板的类：TemplateParserContext。下面是定义：
```java
public class TemplateParserContext implements ParserContext {   
	public String getExpressionPrefix() {   return "#{";  }   
	public String getExpressionSuffix() {   return "}";  }   
	public boolean isTemplate() {   return true;  }
}
```





