# 代理模式
> 代理是一种设计模式，可以为其他对象提供一种代理以控制对这个对象的访问。
> 生活中的类似于代理模式的场景：代理上网(无法直接访问某些IP地址，通过某些可以访问的地址间接访问)、打官司

![代理结构模式示意图](http://www.liuhaihua.cn/wp-content/uploads/2015/09/ruUvAnN.png)
## 代理模式中的角色
1. 代理对象（Proxy）
- 保存一个引用使得代理可以访问实体；
- 实现与 `Subject` 接口，这样代理就能在使用真正对象的地方使用代理对象；
- 控制对象的存取，并可能负责创建和删除它。

2. 主题接口（ `Subject` ）：
- 定义 `RealSubject` 和 `Proxy` 的共同接口，这样就可以在任何使用 `RealSubject` 的地方都可以使用 `Proxy`。

3. 真正对象（ RealSubject ）；
- 定义被 `Proxy` 代理的对象。

```java

public class ProxyTest {


    public interface IService {
        void buyTicket();
    }


    static class TrainService implements IService {

        @Override
        public void buyTicket() {
            System.out.println("买火车票");
        }
    }

    static class ProxyService implements IService {

        @Override
        public void buyTicket() {
            IService service = new TrainService();
            service.buyTicket();
            System.out.println("在携程上买的，多收20块手续费");//对接口方法进行修改
        }
    }

    public static void main(String[] args) {
        IService proxy = new ProxyService();
        proxy.buyTicket();
    }

}
```

## 代理模式的好处

1. 对访问对象的控制
- 代理模式为另一个对象提供代表，以便控制客户对对象的访问。
- 例：当用户查看某些重要的文件时，可以用代理模式检查是否用户是否有这个权限（是否登录）

2. 延迟加载
- 当你需要从网络上面查看一张很大的图片时，你可以使用代理模式先查看它的缩略图，等图片完全下载完在显示

3. 降低耦合度，增加系统的弹性和可扩展性
- 因为用户持有的是代理对象的引用并不是执行用户请求的对象，所以它实现了调用者与被调用者的完全解耦！

4. 代理对象可以在客户端和目标对象之间起到中介的作用，这样起到了中介的作用和保护了目标对象的作用。

# 动态代理

上面介绍的是静态代理，对于一个接口都对应着一个代理类
而动态代理则是根据反射，在运行期间动态创建代理类

```java
    public interface IService {
        void buyTicket();
    }

    static class TrainService implements IService {

        @Override
        public void buyTicket() {
            System.out.println("买火车票");
        }
    }

    static class MyInvocationHandler implements InvocationHandler {

        private Object realObject;

        public MyInvocationHandler(Object object) {
            realObject = object;
        }

        /**
        * 类似与静态代理中重写抽象方法的逻辑，调用被代理对象的方法
        * @param proxy 代理对象本身，被动态创建的代理类的实例对象，一般
        * @param method 正在被调用的方法
        * @param args 被调用方法所需要的参数
        * @return 方法执行的结果
        * @throws Throwable
        */
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

            //输出跟踪调试信息
            System.out.println("enter :" + method.getName());

            Object result = method.invoke(realObject, args);
            System.out.println("在携程上买的(动态代理)");

            System.out.println("out :" + method.getName());

            return result;
        }
    }

    public static void main(String[] args) {

        IService trainService = new TrainService();
        IService proxy = (IService) Proxy.newProxyInstance(
                IService.class.getClassLoader(),        //类加载器，类加载所必须的
                new Class[]{IService.class},            //代理类所需要实现的接口
                new MyInvocationHandler(trainService)); //代理类实际的代码逻辑
        proxy.buyTicket();

    }
```
- Proxy.newProxyInstance 的内部逻辑（反射）
```java
Class<?> proxyCls = Proxy.getProxyClass(IService.class.getClassLoader());               //通过Proxy.getProxyClass 获得代理类的类信息，相关接口的信息
Constructor<?> constructor = proxyCls.getConstructor(new Class<?>[]{IService.class});   //获取代理类的构造方法
InvocationHandler h = new MyInvocationHandler(trainService);                            //创建InvocationHandler实例
IService proxy = constructor.newInstance(h);                                            //调用构造器，创建代理类的实例
```


- 动态创建的代理类示例

```java

final class $Proxy0 extends Proxy implements ProxyTest.IService {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    static {
        m3 = Class.forName("ProxyTest$IService")
                .getMethod("buyTicket", new Class[0]);
                
        m1 = Class.forName("java.lang.Object").getMethod("equals",
                new Class[]{Class.forName("java.lang.Object")});
        m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
        m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
    }

    public $Proxy0(InvocationHandler paramInvocationHandler) {
        super(paramInvocationHandler);
    }
    
    @Override
    public void buyTicket() {
        return this.h.invoke(this, m3, null);
    }

    public final boolean equals(Object paramObject) {
        return ((Boolean) this.h.invoke(this, m1,
                new Object[]{paramObject})).booleanValue();
    }

    public final String toString() {
        return (String) this.h.invoke(this, m2, null);
    }

    public final int hashCode() {
        return ((Integer) this.h.invoke(this, m0, null)).intValue();
    }
}
```

- 这个动态生成的代理类父类是`Proxy`，它有一个构造方法接受`InvocationHandler`类型的参数。
- 为了实例变量h，h定义在父类Proxy中。
- 实现接口`IService`，调用 `InvocationHandler` 中的`invoke`方法

动态代理的优缺点：

- 比起静态代理，动态代理更为复杂。
- 使用动态代理，可以编写通用的代码逻辑，用于各种类型的被代理对象，不需要为每一个被代理的类型创建一个静态代理类
