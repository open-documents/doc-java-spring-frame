	参考文档：P305~P306

Variables通过EvaluationContext的setVariable()方法设置。

对context中的变量使用 `#variableName` 的语法访问。

# 一、基本使用

```java
Inventor tesla = new Inventor("Nikola Tesla", "Serbian");
EvaluationContext context = SimpleEvaluationContext.forReadWriteDataBinding().build(); 
context.setVariable("newName", "Mike Tesla"); 
parser.parseExpression("name = #newName").getValue(context, tesla);
System.out.println(tesla.getName()) // "Mike Tesla"
```

# 二、`#root` & `#this`

```java
// create an array of integers 
List<Integer> primes = new ArrayList<Integer>(); primes.addAll(Arrays.asList(2,3,5,7,11,13,17)); 
// create parser and set variable 'primes' as the array of integers 
ExpressionParser parser = new SpelExpressionParser(); 
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataAccess(); 
context.setVariable("primes", primes); 
// all prime numbers > 10 from the list (using selection ?{...}) 
// evaluates to [11, 13, 17] 
List<Integer> primesGreaterThanTen = (List<Integer>) parser.parseExpression("#primes.?[#this>10]").getValue(context);
```

# 三、方法变量

```java
public abstract class StringUtils {
	public static String reverseString(String input) {
		StringBuilder backwards = new StringBuilder(input.length());
		for (int i = 0; i < input.length(); i++) {
			backwards.append(input.charAt(input.length() - 1 - i)); 
		} 
	return backwards.toString();
	} 
}

ExpressionParser parser = new SpelExpressionParser(); 
EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
context.setVariable("reverseString", StringUtils.class.getDeclaredMethod("reverseString", String.class));
String helloWorldReversed = parser.parseExpression("#reverseString('hello')").getValue(context, String.class);
```