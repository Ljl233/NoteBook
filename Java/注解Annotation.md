# 注解的定义
- 定义：
  - 注解（Annotation），也叫元数据。一种代码级别的说明。
  - 它是JDK1.5及以后版本引入的一个特性，与类、接口、枚举是在同一个层次。
  - 它可以声明在包、类、字段、方法、局部变量、方法参数等的前面，
  - 用来对这些元素进行说明，注释。

注解和注释有点类似：它们都是对代码的一种说明。注释是给程序员看的，而注解则是给代码看的。是一种代码级别的说明。

- 作用分类：
    - ①编写文档：通过代码里标识的元数据生成文档【生成文档doc文档】
    - ②代码分析：通过代码里标识的元数据对代码进行分析【使用反射】
    - ③编译检查：通过代码里标识的元数据让编译器能够实现基本的编译检查【Override】

# 自带的常用注解
1. @Override
   - 重写父类方法时的注解
   - 写不写这个注解对子类有没有重写父类方法没有影响
   - 当父类的方法名被修改时或者子类与父类方法名不一样是，这个注解会提示子类。从而减少编译错误。

2. @Depricated
   - 提示该类或方法被废弃
   - 因为某种原因，API 决定废弃某个类、方法、接口等等，并提供其他可替代的方法，而不是直接在原有的代码上进行修改。在原有的代码上修改会导致原有的大量基于此的代码发生错误从而选择使用注解的方式来告诉开发者。

3. @SuppressWarnings
   - 警告压制    `@SuppressWarnings("all")`
   - 看着一些编译器的警告犯强迫症？那就使用这个注解吧！

# 自定义注解

格式：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

元注解
public @interface 注解名{
}

javac MyAnno.java 
javap MyAnno.class          //反编译

Compiled from "MyAnno.java"
public interface MyAnno extends java.lang.annotation.Annotation {
}
```

1. 主体
```java
public @interface 注解名{}
```

2. 元注解
   - @Target(value) 指定注解修饰的对象，value是一个ElementType[]
   - @Retention(RetentionPolicy.RUNTIME) 表示注解保留到什么时候，取值只能有一个RetentionPolicy 
     - SOURCE 只在源代码中保留，编译器将代码编译成字节码后会丢掉
     - CLASS 保留到字节码文件中，但Java虚拟机将字节码加载到内存时不一定会在内存中保留
     - RUNTIME 一直保留到运行时


3. 属性（方法）
定义一个接口中的抽象方法
    返回类型：
          - 基本类型
          - String
          - 枚举
          - 注解
          - 以上类型的数组
```java
@Documented
@Target({ElementType.METHOD, ElementType.FIELD})
@interface a {
    int value() default 1;

    String string();

    ElementType em();

    ElementType[] ems();

    String[] ss();

    Override s();
}
```
如果只有一个属性，且名字为value可以省略。通过default设置默认值。数组赋值时只有一个值大括号可以省略。

# 注解的使用
1. 获取被注解位置的对象  Class Method Field
2. 获取指定的注解
3. 获取注解的属性
