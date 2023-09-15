Spring内置实现类位于 `org.springframework.format` 的各个子包内。

`org.springframework.format.number` 包：
- `NumberStyleFormatter`
- `CurrencyStyleFormatter`
- `PercentStyleFormatter`

`org.springframework.format.number` 包：
- `DateFormatter`

-- --
DateFormatter

```java
public final class DateFormatter implements Formatter<Date> {   
	private String pattern;   
	public DateFormatter(String pattern) {   
		this.pattern = pattern;
	}
	public String print(Date date, Locale locale) {
		if (date == null) {  
			return ""; 
		}
		return getDateFormat(locale).format(date); 
	}
	public Date parse(String formatted, Locale locale) throws ParseException {
		if (formatted.length() == 0) {
			return null;
		}  
		return getDateFormat(locale).parse(formatted);
	}  
	protected DateFormat getDateFormat(Locale locale) {
		DateFormat dateFormat = new SimpleDateFormat(this.pattern, locale);   
		dateFormat.setLenient(false);  
		return dateFormat;
	}
}
```