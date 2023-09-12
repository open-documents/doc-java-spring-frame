@Named注解和@ManagedBean注解可以理解为@Component注解的平替。
不同的是，它两不能标注在其他注解上，即不同用于合成注解，即不能自定义@Component注解。


```java
@Named 
public class SimpleMovieLister {
	private MovieFinder movieFinder;   
	@Inject   
	public void setMovieFinder(MovieFinder movieFinder) { 
		this.movieFinder = movieFinder;
	} 
}
```



```java
@Named("movieListener") // @ManagedBean("movieListener") could be used as well 
public class SimpleMovieLister {   
	private MovieFinder movieFinder;   
	@Inject   
	public void setMovieFinder(MovieFinder movieFinder) {   
		this.movieFinder = movieFinder;  
	}   
}
```


