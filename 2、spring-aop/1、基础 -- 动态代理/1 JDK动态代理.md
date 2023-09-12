

# 材料准备

1）接口
```java
public interface MyInterface1 {  
    void fun1MyInterface1();  
    void fun2MyInterface1(Long var1);  
}
public interface MyInterface2 {  
    void fun1MyInterface2(String var1, boolean var2);  
}
```
2）实现类
```java
public class MyInterfaceImpl implements MyInterface1, MyInterface2{  
    @Override  
    public void fun1MyInterface1() {  
        System.out.println("MyInterface1的fun1");  
    }  
    @Override  
    public void fun2MyInterface1(Long var1) {  
        System.out.println("MyInterface1的fun2: 参数1 -- " + var1);  
    }  
    @Override  
    public void fun1MyInterface2(String var1, boolean var2) {  
        System.out.println("MyInterface2的fun1: 参数1 -- " + var1 + "参数2 -- " + var2);  
    }  
  
}
```
3）自定义InvocationHandler实现类
```java
public class MyInvocationHandler implements InvocationHandler {  
    private Object target;  
  
    public MyInvocationHandler(Object target) {  
        this.target = target;  
    }  
  
    @Override  
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
        System.out.println("before invoke.........");  
        method.invoke(target, args);  
        System.out.println("after invoke.........");  
  
        return null;    }  
}
```
4）jdk动态代理
```java
public class MyMain {  
    public static void main(String[] args) {  
        System.getProperties().put("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");  
        MyInterface1 impl = new MyInterfaceImpl();  
        MyInterface1 proxy1 = (MyInterface1)Proxy.newProxyInstance(MyMain.class.getClassLoader(), impl.getClass().getInterfaces(), new MyInvocationHandler(impl));  
        MyInterface2 proxy2 = (MyInterface2)Proxy.newProxyInstance(MyMain.class.getClassLoader(), impl.getClass().getInterfaces(), new MyInvocationHandler(impl));  
    }  
}
```

# 生成的动态代理类

最终只会生成一个动态代理类 $Proxy0。


```java
public final class $Proxy0 extends Proxy implements MyInterface1, MyInterface2 {  
    private static final Method m0;  
    private static final Method m1;  
    private static final Method m2;  
    private static final Method m3;  
    private static final Method m4;  
    private static final Method m5;  
  
    public $Proxy0(InvocationHandler param1) {  
        super(var1);  
    }  
  
    public final int hashCode() {  
        try {  
            return (Integer)super.h.invoke(this, m0, (Object[])null);  
        } catch (RuntimeException | Error var2) {  
            throw var2;  
        } catch (Throwable var3) {  
            throw new UndeclaredThrowableException(var3);  
        }  
    }  
  
    public final boolean equals(Object var1) {  
        try {  
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});  
        } catch (RuntimeException | Error var2) {  
            throw var2;  
        } catch (Throwable var3) {  
            throw new UndeclaredThrowableException(var3);  
        }  
    }  
  
    public final String toString() {  
        try {  
            return (String)super.h.invoke(this, m2, (Object[])null);  
        } catch (RuntimeException | Error var2) {  
            throw var2;  
        } catch (Throwable var3) {  
            throw new UndeclaredThrowableException(var3);  
        }  
    }  
  
    public final void fun2MyInterface1(Long var1) {  
        try {  
            super.h.invoke(this, m3, new Object[]{var1});  
        } catch (RuntimeException | Error var2) {  
            throw var2;  
        } catch (Throwable var3) {  
            throw new UndeclaredThrowableException(var3);  
        }  
    }  
  
    public final void fun1MyInterface1() {  
        try {  
            super.h.invoke(this, m4, (Object[])null);  
        } catch (RuntimeException | Error var2) {  
            throw var2;  
        } catch (Throwable var3) {  
            throw new UndeclaredThrowableException(var3);  
        }  
    }  
  
    public final void fun1MyInterface2(String var1, boolean var2) {  
        try {  
            super.h.invoke(this, m5, new Object[]{var1, var2});  
        } catch (RuntimeException | Error var3) {  
            throw var3;  
        } catch (Throwable var4) {  
            throw new UndeclaredThrowableException(var4);  
        }  
    }  
  
    static {  
        try {  
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");  
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));  
            m2 = Class.forName("java.lang.Object").getMethod("toString");  
            m3 = Class.forName("org.example.MyInterface1").getMethod("fun2MyInterface1", Class.forName("java.lang.Long"));  
            m4 = Class.forName("org.example.MyInterface1").getMethod("fun1MyInterface1");  
            m5 = Class.forName("org.example.MyInterface2").getMethod("fun1MyInterface2", Class.forName("java.lang.String"), Boolean.TYPE);  
        } catch (NoSuchMethodException var2) {  
            throw new NoSuchMethodError(var2.getMessage());  
        } catch (ClassNotFoundException var3) {  
            throw new NoClassDefFoundError(var3.getMessage());  
        }  
    }  
  
    private static Lookup proxyClassLookup(Lookup var0) throws IllegalAccessException {  
        if (var0.lookupClass() == Proxy.class && var0.hasFullPrivilegeAccess()) {  
            return MethodHandles.lookup();  
        } else {  
            throw new IllegalAccessException(var0.toString());  
        }  
    }  
}
```