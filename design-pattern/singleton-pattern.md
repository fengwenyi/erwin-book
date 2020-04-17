<h1 align="center">设计模式之单例模式</h1>

## 一、何为单例模式？

单例模式，顾名思义就是一个类只能有一个实例，并且在整个项目中都能访问到这个实例。



特点：

- 类构造器私有
- 持有自己类型的属性
- 对外提供获取实例的静态方法



## 二、为什么需要用单例模式？

首先是一个资源，只有一份，不论怎样的高并发，取到的结果都一样。为了保证性能，需要的时候，才会去取。



## 三、单例模式代码示例

#### 写法1：饿汉模式

```java
// 不允许被继承
public final class Singleton {
	
  //私有构造函数，不允许外部new
	private Singleton(){}
	
  // instance被ClassLoader加载后很长一段时间才被使用，它所开辟的堆内存会驻留更久
	private static Singleton instance = new Singleton();
	
	public static Singleton getInstance(){
		return instance;
	}
}
```

最简单的写法，缺点在于实例在类初始化的时候就创建了，如果在整个项目中都没有使用到该类，就会创建内存空间的浪费。



#### 写法2：懒汉模式

```java
public final class Singleton {
	
	private Singleton(){}
	
	private static Singleton instance;
	
	public static Singleton getInstance(){
		if(instance == null){
			instance = new Singleton();		
		}
		return instance;
	}
}
```

解决了写法1在类初始化的时候就创建实例的问题，然而只能在单线程中使用，在多线程中使用如果多个线程同时进入if语句中，就可能出现创建多个实例的问题。



#### 写法3：加锁

```java
public class Singleton {
	
	private Singleton(){}
	
	private static Singleton instance;
	
	public synchronized static Singleton getInstance(){
		if(instance == null){
			instance = new Singleton();		
		}
		return instance;
	}
}
```

解决了写法2可能出现的问题，可以在多线程中使用。缺点在于synchronized关键字会强制一次只能让一个线程进入方法中，其他线程不得不阻塞等待该线程退出方法。



#### 写法4：双重加锁

```java
public class Singleton {
	
	private Singleton(){}
	
	private static Singleton instance;
	
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

解决了写法3出现的问题，缺点在于 JVM 在执行对引用赋值时并不是一个原子操作。具体是首先在堆中为 Singleton 分配内存空间，然后对 Singleton 执行初始化操作， 最后再将引用变量指向该实例所对应的内存地址以完成赋值。而 JVM 对这一步进行了优化，使得步骤二和步骤三可以不按顺序执行。所以就有可能出现一个线程在执行了步骤一之后执行了步骤三，然后另一个线程调用了该方法，此时 instance 引用变量不为空，然而 Singleton 实例还没有完成初始化，就会造成报错。



```
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```



双重检查模式，进行了两次的判断，第一次是为了避免不要的实例，第二次是为了进行同步，避免多线程问题。由于`singleton=new Singleton()`对象的创建在JVM中可能会进行重排序，在多线程访问下存在风险，使用`volatile`修饰`signleton`实例变量有效，解决该问题。



#### 写法5：加锁

```java
public class Singleton {
	
	private Singleton(){}
	
	private static volatile Singleton instance;
	
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

volatile关键字能够禁止指令重排，保证在写操作没有完成之前不能调用读操作。



#### 写法6：静态内部类

```java
public class Singleton { 
    private Singleton(){
    }
      public static Singleton getInstance(){  
        return Inner.instance;  
    }  
    private static class Inner {  
        private static final Singleton instance = new Singleton();  
    }  
} 
```

只有第一次调用getInstance方法时，虚拟机才加载 Inner 并初始化instance ，只有一个线程可以获得对象的初始化锁，其他线程无法进行初始化，保证对象的唯一性。目前此方式是所有单例模式中最推荐的模式，但具体还是根据项目选择。



#### 写法7：枚举

```java
public enum Singleton {
    INSTANCE;
}
```

默认枚举实例的创建是线程安全的，并且在任何情况下都是单例。实际上

- 枚举类隐藏了私有的构造器。
- 枚举类的域 是相应类型的一个实例对象
  那么枚举类型日常用例是这样子的：

```java
public enum Singleton  {
    INSTANCE 
 
    //doSomething 该实例支持的行为
      
    //可以省略此方法，通过Singleton.INSTANCE进行操作
    public static Singleton get Instance() {
        return Singleton.INSTANCE;
    }
}
```

枚举单例模式在《Effective Java》中推荐的单例模式之一。但枚举实例在日常开发是很少使用的，就是很简单以导致可读性较差。
在以上所有的单例模式中，推荐静态内部类单例模式。主要是非常直观，即保证线程安全又保证唯一性。
众所周知，单例模式是创建型模式，都会新建一个实例。那么一个重要的问题就是反序列化。当实例被写入到文件到反序列化成实例时，我们需要重写`readResolve`方法，以让实例唯一。

```java
private Object readResolve() throws ObjectStreamException{
        return singleton;
}
```