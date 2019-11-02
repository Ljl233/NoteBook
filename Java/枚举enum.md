# 枚举介绍
- 枚举是是一种特殊的数据，取值有限，使用关键字`enum`定义。
- 相比较类而言，更加简洁、安全和方便。这样只会有`null`和几个特定的取值
- 枚举类型自带的方法：
  - `name()`返回值与`toString()`一样
  - 使用`==`和`equals()`进行比较
  - `int ordinal()`表示枚举类型在声明的时候的顺序，从`0`开始
  - `compareTo()`比较ordinal值的大小
  - 静态方法`valueOf(String)`返回字符串对应的枚举值
  - 静态方法`values()`返回一个包括所有值的数组
- 在枚举类型之前（java 5）一般在类中定义静态整型常量来实现类似功能
- 枚举的本质上就是一个类，只不过大部分由编译器来自动实现，所以简洁、安全、方便

# 枚举高级用法
- 将枚举与字符串或者int关联起来。For instance：`SAMLL("小","S",1)`
- 并且有一个私有的构造器，还有`getter`和`setter`方法
- 还可以关联类定义体，声明抽象方法，可以实现接口，在接口中定义枚举

```java
public class EnumDemo {
    public static void main(String[] args) {
        Size s = Size.LARGE;
        Size s2 = Size.valueOf("SMALL");
        System.out.println(s);
        System.out.println(s2);
        System.out.println(s.getId());

        //改变id并不会改变枚举的值
        s.setId(10);
        System.out.println(s.getId());
    }
}

enum Size {
    SMALL("小", 10), MEDIUM("中", 20), LARGE("大", 30);

    private String s;
    private int id;

    private Size(String s, int id) {
        this.s = s;
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setS(String s) {
        this.s = s;
    }

    public String getS() {
        return s;
    }

    public static Size fromAbbr(String s) {
        for (Size size : Size.values()) {
            if (size.getS().equals(s)) {
                return size;
            }
        }
        return null;
    }
}
```
```
LARGE
SMALL
30
10
```