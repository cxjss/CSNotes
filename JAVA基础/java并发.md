# JAVA并发编程实战

## 线程安全性

### Java同步机制

- `synchronized`
- `volatile`变量
- 显式锁
- 原子变量（`java.util.concurrent.atomic`包中）

> 如果当多个线程访问同一个可变的变量时没有使用合适的同步，那么程序就会出现错误，解决方法：
>
> - 不在线程之间共享该状态变量
> - 将状态变量修改为不可变的变量
> - `在访问状态变量时使用同步`

从一开始就设计一个线程安全的类，比以后再将这个类修改为线程安全的类要容易的多。

首先使代码正确运行，然后再提高代码的速度。

### 线程安全性的定义

最核心的概念是`安全性`。

当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类是线程安全的。

> 无状态对象（不包含任何域，也不包含对其他类中域的引用）一定是线程安全的。

### 原子性

```java
++count;
```

该代码并非是原子性的，分为三步，读取-修改-写入

#### 竞态条件

由于不恰当执行时序而出现不正确的结果。

最常见的竞态条件类型：先检查后执行

先检查后执行之`延迟初始化`中的的竞态条件：

```java
@NotThreadSafe
public class LazyInitRace{
    private ExpensiveObject instance = null;
    
    public ExpensiveObject getInstance(){
        if(instance == null)
            instance = new ExpensiveObject();
        return instance;
    }
}
//懒汉式单例模式
```

使用线程安全的类来执行操作，例如使用`AtomicLong`来代替`Long`进行计数器的表示。

#### 单例模式

##### 1. 懒汉式，线程不安全

```java
public class Singleton{
    private static Singleton instance;
    private Singleton(){};
    public static Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
```

##### 2. 懒汉式，线程安全

```java
public class Singleton {  
    private static Singleton instance;  
    private Singleton (){}  
    public static synchronized Singleton getInstance() {  //使用了同步方法保证线程安全，但是每次获取实例都需要加锁，效率大大降低
    	if (instance == null) {  
        	instance = new Singleton();  
    	}  
    	return instance;  
    }  
}
```

##### 3. 饿汉式

```java
public class Singleton {  
    private static Singleton instance =  new Singleton();
    private Singleton(){};
    public static Singleton getInstance(){
        return instance;
    }
}
```

##### 4. 双重校验锁(DCL,double-checked locking)

```java
public class Singleton{
    private volatile static Singleton instance = null;
    private Singleton(){};
    public static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

##### 5. 静态内部类

```java
public class Singleton{
    private static class SingletonHolder{
        private static final Singleton INSTANCE = new Singleton();
    }
    private Singleton(){};
    public static final Singleton getInstance(){
        return SingletonHolder.INSTANCE;
    }
}
```

##### 6. 枚举

```java
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}
```

### 加锁

#### 内置锁

同步代码块`synchronized`，可重入(自己可以进入自己持有的锁，其他的不可以。当线程请求一个未被持有的锁的时候，JVM将记下锁的持有人，将获取计数器置为1。同一个线程再次获取该锁，计数值将增加，退出则减少。计数值为0时，锁将被释放。)

