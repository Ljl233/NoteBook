# Activity 的概念

> 移动应用体验与桌面体验的不同之处在于，用户与应用的互动并不总是在同一位置开始，而是经常以不确定的方式开始。例如，如果您从主屏幕打开电子邮件应用，可能会看到电子邮件列表，如果您通过社交媒体应用启动电子邮件应用，则可能会直接进入电子邮件应用的邮件撰写界面。
>
>Activity 类的目的就是促进这种范式的实现。当一个应用调用另一个应用时，调用方应用会调用另一个应用中的 Activity，而不是整个应用。通过这种方式，Activity 充当了应用与用户互动的入口点。您可以将 Activity 实现为 Activity 类的子类。
>
>Activity 提供窗口供应用在其中绘制界面。此窗口通常会填满屏幕，但也可能比屏幕小，并浮动在其他窗口上面。通常，一个 Activity 实现应用中的一个屏幕。例如，应用中的一个 Activity 实现“偏好设置”屏幕，而另一个 Activity 实现“选择照片”屏幕。
>
>大多数应用包含多个屏幕，这意味着它们包含多个 Activity。通常，应用中的一个 Activity 会被指定为主 Activity，这是用户启动应用时出现的第一个屏幕。然后，每个 Activity 可以启动另一个 Activity，以执行不同的操作。例如，一个简单的电子邮件应用中的主 Activity 可能会提供显示电子邮件收件箱的屏幕。主 Activity 可能会从该屏幕启动其他 Activity，以提供执行写邮件和打开邮件这类任务的屏幕。
>
>虽然应用中的各个 Activity 协同工作形成统一的用户体验，但每个 Activity 与其他 Activity 之间只存在松散的关联，应用内不同 Activity 之间的依赖关系通常很小。事实上，Activity 经常会启动属于其他应用的 Activity。例如，浏览器应用可能会启动社交媒体应用的“分享”Activity。

摘自Android官网 https://developer.android.google.cn/guide/components/activities/intro-activities?hl=zh-cn

为什么要先讲Activity呢？
我之前其实一直把Activity和View搞混淆，以为Activity就是View。其实这是不准确的，真正展示View的是Window。Activity是提供窗口供应用在其中绘制界面的。而Activity的作用主要是控制这个页面的显示，各种状态（生命周期方法），事件的传递和分发。

![从Activity创建到View呈现](https://img-blog.csdnimg.cn/20190613171455216.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FuZHJvaWRfU0U=,size_16,color_FFFFFF,t_70)

![Activity](https://s2.ax1x.com/2019/05/31/VlEngI.png)


# Window 和 WindowManager

Window表示一个窗口的概念。Window是一个抽象类，具体的实现类是PhoneWindow。Android中所有的视图都是由Window来实现的，无论是Activity，Dialog还是Toast，还是悬浮窗，它们的视图都是依附在window上显示的，Window实际是View的直接管理者。

WindowManager的作用就是管理Window，是外界访问Window的入口。但是实际上对于window的添加，更新视图，删除操作具体的实现又是在WindowManagerService上的，WindowManager和WindowManagerService的交互是一个IPC（Inter-Process Communication 跨进程通信）过程

为了理解它们之间是怎么协作的，将从Window的添加过程来分析

## Window的添加过程

Window的添加过程需要WindowManger的addView来实现，WindowManager是一个接口，真正的实现类是WindowManagerImpl类。在WindwoManagerImpl中Window的三大操作：

```java
@Override
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
}

@Override
public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.updateViewLayout(view, params);
}

@Override
public void removeView(View view) {
    mGlobal.removeView(view, false);
}
```
WindowManager将所有的操作全部委托给WindowManagerGlobal来实现。
WindowManagerGlobal的addView方法主要分为几步

1. 检查参数是否合法，如果是子Window那么还需要调整一些布局参数
```java
 if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }
        if (display == null) {
            throw new IllegalArgumentException("display must not be null");
        }
        if (!(params instanceof WindowManager.LayoutParams)) {
            throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
        }

        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
        if (parentWindow != null) {
            parentWindow.adjustLayoutParamsForSubWindow(wparams);
        } else {
            // If there's no parent, then hardware acceleration for this view is
            // set from the application's hardware acceleration setting.
            final Context context = view.getContext();
            if (context != null
                    && (context.getApplicationInfo().flags
                            & ApplicationInfo.FLAG_HARDWARE_ACCELERATED) != 0) {
                wparams.flags |= WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED;
            }
        }
```
2. 创建ViewRootImpl并将View添加到列表中
```java
root = new ViewRootImpl(view.getContext(), display);
view.setLayoutParams(wparams);
mViews.add(view);
mRoots.add(root);
mParams.add(wparams);
```
mViews，mRoots，mParams存储的是每一个Window的视图，ViewRootImpl，参数。通过相同的索引来进行关联。

3. 通过ViewRootImpl来更新界面并完成Window的添加过程
```java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
    ……
    requestLayout();

}

public void requestLayout() {
    if (!mHandlingLayoutInLayoutRequest) {
        checkThread();//检查ui线程
        mLayoutRequested = true;
        scheduleTraversals();//measure，layout，draw
    }
}
```


接着通过WindowSession来完成Window的添加，Session内部会通过WindowManagerService来实现Window 的添加。
```java
try {
    mOrigWindowType = mWindowAttributes.type;
    mAttachInfo.mRecomputeGlobalAttributes = true;
    collectViewAttributes();
    res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
            getHostVisibility(), mDisplay.getDisplayId(), mTmpFrame,
            mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
            mAttachInfo.mOutsets, mAttachInfo.mDisplayCutout, mInputChannel,
            mTempInsets);
    setFrame(mTmpFrame);
} catch (RemoteException e) {
    mAdded = false;
    mView = null;
    mAttachInfo.mRootView = null;
    mInputChannel = null;
    mFallbackEventHandler.setView(null);
    unscheduleTraversals();
    setAccessibilityFocus(null, null);
    throw new RuntimeException("Adding window failed", e);
}
```
window创建过程调用链：

windowManagerImpl 》 WindowManagerGlobal 》ViewRootImpl, WindowSession 》 WindowManagerService 


## Activity的Window创建过程


1. 先从Activity的`attach`方法开始
```java
final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
        attachBaseContext(context);

        mFragments.attachHost(null /*parent*/);

//创建window
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(this);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        ………………

//为Window添加WindowManager
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager();
        mCurrentConfig = config;

        mWindow.setColorMode(info.colorMode);

        ………………
    }
```

这里通过new关键字就创建好了一个Window，但是这其实还不够，还有视图没有添加进去。

添加视图的入口是在`onCreateView`方法中调用的`setContentView`方法。

```java
@Override
public void setContentView(View view, ViewGroup.LayoutParams params) {
    if (mContentParent == null) {
        installDecor();
    } 
    …………
    if (！hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        mContentParent.addView(view, params);
    }
    …………
}
private void installDecor() {
    if (mDecor == null) {
        mDecor = generateDecor(-1);
        mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        mDecor.setIsRootNamespace(true);
        if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
            mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
        }
    }
    if (mContentParent == null) {
        mContentParent = generateLayout(mDecor);
        ……
    }
}

```

如果mContentParent为null就执行installDecor()方法，在这里面生成mDecor和mContentParent。
generateLayout方法是为mDecor创建默认布局。

2. 在Activity resume过程中通过WindowManager.addView()将DecorView绘制完成
这个就和之间Window 的创建过程一样了。


- ActivityThread.handleResumeActivity
```java
//处理Activity的onRestart onResume生命周期。
ActivityClientRecord r = performResumeActivity(token, clearHide);

if (r != null) {
    if (r.window == null && !a.mFinished) {
        r.window = r.activity.getWindow();
        View decor = r.window.getDecorView();
        //设置DecorView不可见
        decor.setVisibility(View.INVISIBLE);
        
        ViewManager wm = a.getWindowManager();
        WindowManager.LayoutParams l = r.window.getAttributes();
        
        if (a.mVisibleFromClient) {
            a.mWindowAdded = true;
            //利用WindowManager添加DecorView。
            wm.addView(decor, l);
        }
    }
```

这里windowManager添加的是decor，因为这个时候decor已经包括activity的布局了。

>参考
>
>《Android开发艺术探索》
>
>[从Activity创建到View呈现中间发生了什么？](https://juejin.im/post/5d00cc9fe51d4510624f97b4)
