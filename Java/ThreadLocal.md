# ThreadLocal 的理解
- 一个`ThreadLocal`对象存储多个Thread实例对于同一个对象的副本，互不干扰互不同步。
- 每个线程对于同一个变量有不同的副本
- 在方法或者类间共享。

# ThreadLocal 使用
```java
threadLocal.set(T value);//将value以当前ThreadLocal（！线程）为键放入threadLocalMap中
threadLocal.get();//返回和当前线程对应的value
```
# set get 方法解析
## set
```java
    public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);//获取当前线程的map
      if (map != null)
        map.set(this, value);//覆盖map中的对应的值
    else
        createMap(t, value);//当前线程没有map，就实例化map
    }
//获得map将（当前ThreadLocal，value）键值队放入map
//为什么是ThreadLocal而不是Thread，而且createMap放入的却是thread
```
## createMap

```java
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
    //每个thread中的map不一样，但是都是和同一个ThreadLocal 关联
    //构造函数的用处是将（threadLocal， value）映射放入map维护的一个以thread的哈希值为索引的Entry（threadLocal，value）数组中
```


## get

```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    //获取当前Thread，关于this ThreadLocal的，对应的value
```
## ThreadLocalMap
![ThreadMap](http://www.jasongj.com/img/java/threadlocal/ThreadMap.png)
ThreadLocalMap是ThreadLocal的静态内部类，内部维护一个以threadLcoal为键，value为值的表。储存的是当前Thread所有的ThreadLocal的本地变量
- 每个Thread有唯一一个属于自己的Map，不存在多个线程之间的同步和线程安全问题