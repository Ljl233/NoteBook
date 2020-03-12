# JDK JVM JRE :white_check_mark:
- JDK (Java Development Kit)      Java 开发工具包
- JRE (Java Runtime Environment)  Java 运行时环境
- JVM (Java Virtual Machine)      Java 虚拟机
JDK 包括 JRE 包括 JVM 

JRE 提供了运行Java代码所需要的环境 即各种运行和lib库，但是不包含任何编译调试工具

Java 生成的字节码是运行在虚拟机上的，虚拟机可以跨平台安装

- 环境变量：etc/profile
```
export JAVA_HOME=/usr/local/jdk1.7.0_71

export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

export PATH=$JAVA_HOME/bin:$PATH
```


# 包装类 :white_check_mark:
> 8大基本数据类型： int float double char boolean byte short long 
> 变量的声明就是在内存中分配一块连续的空间，并给这个位置取个名字指向这块内存空间所在的位置。

> 每一个基本类型都有一个包装类，int 对应的包装类是 Integer，Integer只有实例化之后蔡能使用，而int可以直接使用。
> Java中很多代码 只能操作对象，为了操作基本数据类型，需要其对应的包装类。
> 包装类提供了很多方法，可以方便对基本数据类型的操作。

> 不可变性：
> 包装类都是不可变的类型，一旦实例被创建，就不可修改。
> 这样是为了保证安全的数据共享，让程序变得更加简单安全，尤其是在多线程的环境下，避免了数据在不同线程同时被修改的情况发生。

# Java 各访问修饰符的意义和常用用法和区别 :white_check_mark:

# java中参数是值传递还是引用传递 :white_check_mark:

# 抽象类和接口的区别 :white_check_mark:

# 面向对象的四大特征
- 封装
- 抽象
- 继承
- 多态

# String、StringBuffer和StringBuilder的区别？

# String a=""和String a=new String("")的的关系和异同？

# Object的equal()和==的区别？

# 重写和重载的区别？

# 为什么匿名内部类中使用局部变量要用final修饰？

# 成员变量和局部变量的区别？

# 原子性，有序性，可见性

# String str="i"与 String str=new String("i")一样吗？


#  List、Set、Map 之间的区别是什么？


#  守护线程是什么？

# 线程有哪些状态？

# 线程的 run()和 start()有什么区别？

#  什么是死锁？
- 死锁是锁无法被解开
- 一般在避免一段代码量同时被多个线程调用时，会给它加上锁，当这段代码运行时，其他想要调用这段代码的线程需要等待当前线程结束。
- 但是假如 A线程获得了代码a的锁，a想要调用代码b，但是b被B获得了锁，就要等待B结束，而b需要调用a，又要等待A结束，这样形成了一个死循环，也就是死锁。



# 什么是闭包(closure)？
- 闭包是能够读取其他函数内部的函数。
- 在JavaScript中只有函数内部的子函数才能读取函数的局部变量
- 闭包其实就是一个函数内部的函数
- 本质上，闭包是一个将函数内部与函数外部连接的桥梁
- 一般来说闭包就是lambda表达式

```kotlin
fun a(): (Int) -> Int {
    val i = 1
    fun b(a: Int): Int {
        return i + a;
    }
    return ::b
}

fun main() {
    val res = a()
    println(res(1))
}
```