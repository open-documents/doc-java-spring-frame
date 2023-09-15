
大概来说就是，perthis针对每一个符合的代理对象创建一个切面，pertarget针对每一个符合的被代理对象创建一个切面。


# 准备工作

```java
public interface TargetDao {
    public void print();
}
@Repository("demodao")
@Scope("prototype")    //添加prototype
public class DemoDao implements TargetDao{
    public void print(){
        System.out.println("print empty");
    }
}

```
测试：
```java
public class DemoTest {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext anno = new AnnotationConfigApplicationContext(Appconfig.class);
        TargetDao dao = (TargetDao) anno.getBean("demodao");
        TargetDao dao1 = (TargetDao) anno.getBean("demodao");
        dao.print();  //拿取切面1
        dao1.print(); //拿去切面2
    }
}

```

# 单实例初始化

```java
@Component
@Aspect
public class Demo2Aspect {
    @Pointcut("this(com.demo.dao.TargetDao)")
    public void myAspectModelThis(){ }
    
    @Around("myAspectModelThis()")    
	public void aroundModel(ProceedingJoinPoint pjp){
        System.out.println("arround start ------ ");
        System.out.println(this.hashCode());
        try {
            pjp.proceed();             //执行连接点
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("arround end------ ");
    }
}


```
测试结果：
```shell
# 对比执行结果，很明显这次两个使用的是同一个切面类实例对象。
arround start ------ 
483525032
print empty
arround end------ 
arround start ------ 
483525032
print empty
arround end------
```

# perthis初始化
```java
@Component
@Aspect("perthis(myAspectModelThis())")
@Scope("prototype")    //添加prototype
public class Demo2Aspect {
    @Pointcut("this(com.demo.dao.TargetDao)")
    public void myAspectModelThis(){ }
    
    @Around("myAspectModelThis()")    
	public void aroundModel(ProceedingJoinPoint pjp){
        System.out.println("arround start ------ ");
        System.out.println(this.hashCode());  //打印hashcode
        try {
            pjp.proceed();             //执行连接点
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("arround end------ ");
    }
}

```


结果：
```shell
# 执行，发现确实hashcode不一样，就说明构造了两个实例
arround start ------ 
1923634801
print empty
arround end------ 
arround start ------ 
1730337646
print empty
arround end------
```


# pertarget

```java
@Aspect("pertarget(myAspectModelThis())")
@Scope("prototype")
public class Demo2Aspect {
    @Pointcut("this(com.demo.dao.TargetDao)")
    public void myAspectModelThis(){ }
    
    @Around("myAspectModelThis()")    
	public void aroundModel(ProceedingJoinPoint pjp){
        System.out.println("arround start ------ ");
        System.out.println(this.hashCode());
        try {
            pjp.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("arround end------ ");
    }
}

```
测试结果：
```shell
# 执行，也是两个hashcode，说明这个pertarget也能生成两个对象。
arround start ------ 
1923634801
print empty
arround end------ 
arround start ------ 
1730337646
print empty
arround end------
```

# 区别

```java
@Aspect("perthis(myAspectModelThis())")
@Scope("prototype")
public class Demo2Aspect {
    @Pointcut("this(com.demo.dao.DemoDao)")   //注意这里
    public void myAspectModelThis(){ }
    
    @Around("myAspectModelThis()")
    public void aroundModel(ProceedingJoinPoint pjp){
        System.out.println("arround start ------ ");
        System.out.println(this.hashCode());
        try {
            pjp.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("arround end------ ");
    }
}


```

测试结果：
```shell
# 打印结果
print empty
print empty

```

```java
@Aspect("pertarget(myAspectModelThis())")
@Scope("prototype")
public class Demo2Aspect {
    @Pointcut("target(com.demo.dao.DemoDao)")
    public void myAspectModelThis(){ }
    
    @Around("myAspectModelThis()")
    public void aroundModel(ProceedingJoinPoint pjp){
        System.out.println("arround start ------ ");
        System.out.println(this.hashCode());
        try {
            pjp.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("arround end------ ");
    }
}



```

```shell
# 打印结果
arround start ------ 
418513504
print empty
arround end------ 
arround start ------ 
1256405521
print empty
arround end------
```