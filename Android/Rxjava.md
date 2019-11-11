- 依赖
```
implementation "io.reactivex.rxjava2:rxjava:2.2.13"
implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
```

我觉得rxjava主要是观察者模式加上链式表达使得异步更加方便，方便的转换线程。



#  观察者模式
> 观察者模式（发布－订阅模式）：属于行为型模式的一种，是一种一对多的依赖关系，多个观察者观察一个被观察者。当被观察者变化时，所有观察者都会收到通知，并进行自身的改变。
> 
> 就相当于，在微博，微信平台上关注了一些明星。当他们状态更新了之后就会给你发消息提醒“您关注的ｃｘｋ有新的动态哦！"，你会收到相关的提醒，会作出反映，打开app查看，或者不理会(不做任何处理也相当于一种处理)

### 使用场景
- 关联行为场景，比如注册之后自动登录(注册成功后自动调用登录的接口)，需要注意的是，关联行为是可拆分的，而不是"组合"关系
- 事件多级触发场景
- 跨系统的信息交换场景，如消息队列(“消息队列”是在消息的传输过程中保存消息的容器。)，事件总线的处理机制
- Java 中的消息消息通知是按照顺序执行的，如果一个观察者卡顿，会影响整体的执行效率，一般采用异步实现

# 使用

## 基本使用 `observable.subscribe(observer);`
```java
observable.subscribe(observer);
//被观察者被观察者订阅；
```
```java
//实现接口+链式表达
Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(ObservableEmitter<Integer> emitter) throws Exception {
                Log.d(TAG, "emit" + 1);
                emitter.onNext(1);
                Log.d(TAG, "emit" + 2);
                emitter.onNext(2);
                Log.d(TAG, "emit" + 3);
                emitter.onNext(3);
                Log.d(TAG, "emit complete");
                emitter.onComplete();
                Log.d(TAG, "emit" + 1);
                emitter.onNext(1);
                Log.d(TAG, "emit" + 2);
                emitter.onNext(2);
                Log.d(TAG, "emit" + 3);
                emitter.onNext(3);
            }
        }).subscribe(new Observer<Integer>() {
             Disposable mDisposable;
            @Override
            public void onSubscribe(Disposable d) {
                //订阅时首先调用这个方法
                Log.d(TAG, "onSubscribe");
                //可以在这个方法中获得　Disposable 的对象
                 mDisposable = d;
            }

            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "onNext" + integer);
                if (integer == 2) {
                    Log.d(TAG, "dispose");
                    mDisposable.dispose();
                    Log.d(TAG, "isDisposed:" + mDisposable.isDisposed());
                }
            }

            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "error");
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "onComplete");
            }
        });
```
ObservableEmitter:　用来发射事件`onNext(T values)` `onComplete` `onError(Throwable erroer)`
- Observable可以发送无限个onNext, 只要observer接收的过来。在同一个线程中发送一个接受一个，再发送一个，再接受一个
- onComplete 后Observable继续发送，但observer 不再接受
- 发射一个onError 后Obserable 继续发送，observer 接收到onError后不再接收
- 不能发射多个onComplete和onError，因为发送一个onComplete或者onError后，后面的onComplete和onError不会再被处理，那样onComplete 就没有意义了，然而onError  抛出的异常就不会被处理而导致系统崩溃。
- dispose()会中断obserable和observer的联系，obserable继续发送事件，observer接受不到事件。                      ----> 在这里看来似乎与onComplete功能一样。

subscribe还有几个重载方法
```java
    public final Disposable subscribe() {}
    public final Disposable subscribe(Consumer<? super T> onNext) {}
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError) {} 
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete) {}
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError, Action onComplete, Consumer<? super Disposable> onSubscribe) {}
    public final void subscribe(Observer<? super T> observer) {}
  ```
这里面除了刚才用到的以外，其他的方法都是返回了一个disposable对象，方法里面的参数对应着包含相当于onNext，onError，onDisposable的抽象方法的Cousumer泛型接口，onComplete在Action接口里。

除了disposable以外其他方法的使用和上面讲的那个方法差不多，既然这几个方法返回值是disposable，那么就应该使用这个返回值来dispose。
但是调用了却没有反应，如果在参数重写accept(Disaposable)就会首先调用。？？？？？？


# 操作符
## 线程调度
- Observable 和observer 默认是在同一个线程下
- rxjava 内置的线程调度器可以轻松的实现线程的转换
    - subscribeOn() 指定的是上游发送事件的线程,
        - 多次指定上游的线程只有第一次指定的有效
    - observeOn() 指定的是下游接收事件的线程。
        - 多次指定下游的线程是可以的, 也就是说每调用一次observeOn() , 下游的线程就会切换一次.
        - 多次指定下游是如何切换的

代码实现
```java
 Observable observable = Observable.create(new ObservableOnSubscribe() {
            @Override
            public void subscribe(ObservableEmitter emitter) throws Exception {
                emitter.onNext(1);
                Log.e(TAG, "observable emit--->" + 1);
                Log.e(TAG, "after the thread is---->" + Thread.currentThread().getName());

                emitter.onNext(2);
                Log.e(TAG, "observable emit--->" + 2);
                Log.e(TAG, "after the thread is---->" + Thread.currentThread().getName());
            }
        });
        Consumer<Integer> consumer = new Consumer<Integer>() {
            @Override
            public void accept(Integer o) throws Exception {
                Log.e(TAG, "consumer accept---->" + o +
                        "        the thread is----->" + Thread.currentThread().getName());
            }
        };

        observable.subscribeOn(AndroidSchedulers.mainThread())
                .observeOn(Schedulers.newThread())
                .doOnNext(consumer)
                .observeOn(Schedulers.io())
                .subscribe(consumer);
    }

  //observer改变线程会在两个线程内有顺序的收到Observable 发射的数据，会分别往两个线程发射数据
```

####  Rxjava内置线程选项
- Schedulers.io() 代表io操作的线程, 通常用于网络,读写文件等io密集型的操作
- Schedulers.computation() 代表CPU计算密集型的操作, 例如需要大量计算的操作
- Schedulers.newThread() 代表一个常规的新线程
- AndroidSchedulers.mainThread()  代表Android的主线程


## `rxjava`  +  `retrofit`网络请求实践
网络请求步骤
- 创建Retrofit客户端，包含baseurl，拦截器，解析器
- 创建Api接口，每个抽象方法返回一个Call对象
- 使用Call对象进行同步或者异步的网络请求
- 用Retrofit对象通过create()方法创建Api接口的对象
- 再用这个对象调用接口里的方法获得Call对象

使用rxJava后
- 在Api接口内的方法返回的是Observable对象
- 用rxJava替代Call来进行异步请求

我们在子线程里进行网络请求，将请求结果返回主线程更新ui，如果Activity退出后，更新ui就会报空指针错误。这种情况就可以在Activity中保存这个Disposable对象，当Activity退出时调用dispose()方法。这样就不会接受到数据，就不会更新ui了。

## 嵌套请求
### `Map`
> 将Observable 的结果转换成另外一种形式，对其进行加工
```java
observable.map(new Function<Integer, String>() {
            @Override
            public String apply(Integer integer) throws Exception {
                return "This is result " + integer;
            }
        }).subscribe(observer)
```

### `FlatMap`
> 将Observable 的结果转换成另外一个Observable
```java
observable.flatMap(new Function<Integer, ObservableSource<String>>() {
            @Override
            public ObservableSource<String> apply(Integer integer) throws Exception {
                final List<String> list = new ArrayList<>();
                for (int i = 0; i < 3; i++) {
                    list.add("I am value " + integer);
                }
                return Observable.fromIterable(list).delay(10,TimeUnit.MILLISECONDS);
            }
        }).subscribe(observer);
```
flatMap返回的结果是无序的，如果想要有序的结果，使用concatMap




## `zip`
> 将多个Observable 对象的结果打包成一个进行处理（最多9个），并实现Function 接口来定义处理操作
> 将每个Observable对象中发射的数据一一对应进行打包
> 当一个Observable对象complete后，另一个也会停止产生数据

```java
Observable.zip(observable1, observable2, new BiFunction<Integer, String, String>() {                 
    @Override                                                                                        
    public String apply(Integer integer, String s) throws Exception {                            
        return integer + s;     
    } 
}).subscribe(observer);

```

# 背压 Backpressure
一般会用rxjava的线程管理来处理耗时操作，比如网络请求，读写数据库，因为如果都在主线程操作的话会堵塞线程，而更新ui的操作只能在主线程。

这是上游的操作耗时太长这样处理，那上游太快了，下游来不及处理怎么办呢？
这时候就需要用到背压了

## flowable.subscribe(subscriber);
在使用create方法构造Flowable对象时，增加了一个参数，这个参数就对应着一个背压策略

除了这个和Observable 不一样以外，在observer里的onSubscribe传进来的方法不再是disposable对象而是Subscription对象，subscription.cancle()和disposable.dispose()有同样的效果。他们的区别在于subscription增加了request()方法



### request
- request(int)方法体现了响应式拉取的思想
- 根据subscriber的处理能力，即request（int）里的参数，来告诉flowable需要发送几个数据
- 异步的时候会有一个可以装下128个对象的队列

### 背压策略
- BackpressureStrategy.DROP
    - 将存不下的直接丢弃
- BackpressureStrategy.ERROR
    - 超过128后直接报错
- BackpressureStrategy.LATEST
    - 只保留最新事件
- BackpressureStrategy.BUFFER
    - 没有大小限制