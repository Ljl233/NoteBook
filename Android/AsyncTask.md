# AsyncTask 源码分析

## AsyncTask 使用流程
![https://blog.csdn.net/carson_ho/article/details/79314325](https://imgconvert.csdnimg.cn/aHR0cDovL3VwbG9hZC1pbWFnZXMuamlhbnNodS5pby91cGxvYWRfaW1hZ2VzLzk0NDM2NS0zMWRmNzk0MDA2YzY5NjIxLnBuZz9pbWFnZU1vZ3IyL2F1dG8tb3JpZW50L3N0cmlwJTdDaW1hZ2VWaWV3Mi8yL3cvMTI0MA)


## 开始准备工作
### execute()
```java
   @MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }
```
- execute()只能在主线程执行
- 调用executeOnExecutor方法，将sDefaultExecutor当作参数传入方法，这个是默认的执行的Executer

### 看一下sDefaultExecutor是个什么东西
```java
//sDefaultExecutor初始值是SerialExecutor的实例化
    private static class SerialExecutor implements Executor {
        //mTasks是一个维护Runnable 的双端队列，没有容量限制，其容量自增加，不支持并发
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        //mActive表示正在执行的任务
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            //将r进行封装
            //将这个任务放到队尾
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        //当执行这个任务结束之后，执行下一个任务
                        scheduleNext();
                    }
                }
            });
            //当前没有任何任务的之后，立即调用这个方法来执行下一个
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```
SerialExecutor的用处
- 将执行任务进行进一步的封装，真正处理这个任务的是THREAD_POOL_EXECUTOR
- 将新任务放进一个队列中，保证每次处理的都是按顺序的，也就是所谓的串行处理
- 用THREAD_POOL_EXECUTOR线程池来进行任务的调度。

### execteOnExecutor
```java
    @MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                //如果当前的AsyncTask是运行状态就抛出异常，不再执行新的任务
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                //如果当前的AsyncTask是已完成状态也抛出异常，不再执行新任务
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();
        //将参数传递给mWorker
        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }
```
将所有的处理任务所需要的信息，和对结果的控制放在mFuture里面，由线程池来调度任务的执行


- onPreExecute();
```java
//在真正开始执行之前先进行onPreExecute，重写此方法
    @MainThread
    protected void onPreExecute() {
    }
```
- mWorker 是一个WorkerRunnable的对象

```java
//指定泛型result的Callable
     private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
        Params[] mParams;
    }
```

## 实际任务的处理
### AsynaTask的构造函数，将mWorker和mFuture实例化
```java
    //AsyncTask的构造函数需要在ui线程调用
    public AsyncTask(@Nullable Looper callbackLooper) {
        //默认的主线程调用构造方法，如果不是主线程就创建一个这个looper的handler
        mHandler = callbackLooper == null || callbackLooper == Looper.getMainLooper()
            ? getMainHandler()
            : new Handler(callbackLooper);

        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                //mTaskInvoked 是一个可以保证原子性的布尔值
                //call执行的时候，设为true表示该任务开始执行
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    //将call执行的线程，设置为后台线程级别
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    //调用doInBackground实际执行任务，将结果返回，`在工作线程执行方法`,该线程由执行这个回调方法的线程来控制
                    result = doInBackground(mParams);
                    //刷新等待指令
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    //将mCancelled设置为true，进行 异常判断，是否完成当前任务
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    //将结果传给postresult，postresult将结果发送到当前handler的消息队列中
                    postResult(result);
                }
                return result;
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {//任务完成或取消都会调用done方法
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }
```
- 将对参数params的处理包装成一个nWorker，在mWorker的call方法中调用doInBackground来处理。
- 在用FutureTask将mWorker进行封装，来控制mWorker的完成情况

### doInBackground

在mWorker的done的方法被调用
```java
protected abstract Result doInBackground(Params... params);
```

## 任务执行过程中和结束时的消息处理

利用postResult将result中的信息发送给InternalHandler来处理
```java
    private void postResultIfNotInvoked(Result result) {
        final boolean wasTaskInvoked = mTaskInvoked.get();
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }
```
```java
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```

只是将result和task的实例进行封装，没有实例方法
```java
    private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;

        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
```

- InternalHandler是一个继承了Handler的内部类，重写了handleMessage方法
- msg有两种
  - MESSAGE_POST_RESULT： 表示任务已经完成，调用finish方法来处理结果
  - MESSAGE_POST_PROGRESS： 表示阶段性完成，publishProgress()发送的msg，调用onProgressUpdate来显示进度条等操作

```java
    private static class InternalHandler extends Handler {
        public InternalHandler(Looper looper) {
            super(looper);
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
```

根据状态来处理，已取消或者已完成来分别处理
```java
    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);//调用onCancelled（）方法，需要重写
        } else {
            //onPostExecute需要重写
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
```

## 总结

AsyncTask的执行流程：

- 在主线程（可以不是主线程）调用execute()方法，传入`param`参数，以`SerialExecutor`的实例化对象`sDefaultExecutor`为参数调用executeOnExecutor(Executor exec, Params... params)来处理params
- `SerialExecutor`是继承executor一个内部类，内部维护了一个任务队列，保证事务的串行执行，实际是在线程池THREAD_POOL_EXECUTOR的execte方法来处理的，处理的就是FutureTask的对象
- 将doInBackground方法封装在mWorker的call()方法中，再把mWorker当参数实例化一个FutureTask的实例，由FutureTask来处理doInBackground中的异常
- mFuture在执行过程中发生异常时，是通过调用postResultIfNotInvoked方法调用postResult()
- 执行成功时直接调用postResult来finish()，finish()再调用重写的onPostExecute()来处理结果