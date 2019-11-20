![view](https://note.youdao.com/yws/res/3951/WEBRESOURCE7d17995ce87853c04166b8ecadb98362)

View 自身的坐标
- getTop() 获取View顶边到父布局顶边的距离

- 三个构造器区别，使用情况

#  动画
##### 用xml文件设置动画
- res目录下新建anim文件，并创建`translate.xml`
- 在java代码中调用
- 在逐帧动画中view移动之后，对于屏幕的`位置参数`并没有改变，只是视觉上的改变（其他的不清楚），属性动画解决了这个问题



## scrollTo() 和 scrollBy()
- scrollTo(x,y)将view移动到相对于起始点的相对位置，scroolBy(dx,dy)移动指定的偏移量
- 这两个方法可以看做是手机屏幕的滚动，偏移量应该设置为负值
- scroolTo是一次性的，scroolBy可以进行多次滚动，以scoolTo为基础

```
((View) getParent()).scrollBy(-offsetX, -offsetY);
```
## `Scroller`
> 实现过渡效果的滑动

- `computeScrollOffset()`是否完成滑动
> To track the changing positions of the x/y coordinates, use computeScrollOffset(). The method returns a boolean to indicate whether the scroller is finished. If it isn't, it means that a fling or programmatic pan operation is still in progress. You can use this method to find the current offsets of the x and y coordinates, for example:
```
if (mScroller.computeScrollOffset()) {
     // Get current x and y positions
     int currX = mScroller.getCurrX();
     int currY = mScroller.getCurrY();
    ...
 }
 ```
 - 重写computeScroll方法
```java
  @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            //getCurrX():   The new X offset as an absolute distance from the origin.
            ((View) getParent()).scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            //使无效。。。 将旧的view从主UI线程队列中pop掉，刷新view
            invalidate();
        }
    }
```
- 写一个smoothScroll方法调用startScroll方法

```java
public void smoothScrollTo(int destX, int destY) {
        int scrollX = getScrollX();
        Log.e("tag", scrollX + "");
        int delta = destX - scrollX;
        mScroller.startScroll(scrollX, 0, delta, 0, 2000);
        invalidate();
    }
```
startScroll会不断调用computeScoll方法来实现滑动效果


# View 工作流程

### measureSpec
> A MeasureSpec encapsulates the layout requirements passed from parent to child. Each MeasureSpec represents a requirement for either the width or the height. A MeasureSpec is comprised of a size and a mode. There are three possible modes:


- UNSPECIFIED

The parent has not imposed any constraint on the child. It can be whatever size it wants.
- EXACTLY

The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be.
- AT_MOST

The child can be as large as it wants up to the specified size.




#### Activity 的构成

![Activity构成](http://artimg.ishenping.com/20190731052934499_FMECKH.jpg)
```
Activity-> setContentView(int)
    ||
PhoneWindow-> setContentView(view)
    ||
generalDecor(view,layoutPamas)
    ||
加载布局，包含titleBar、contentView
```
sds


# 自定义View

## 继承View类

## 继承ViewGroup实现类（以titleBar为例）
#### 设置xml布局
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/title_layout"
    android:layout_width="match_parent"
    android:layout_height="45dp">

    <ImageView
        android:id="@+id/title_left"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:padding="10dp"
        android:src="@drawable/ic_chevron_left_black_24dp" />

    <TextView
        android:id="@+id/title_title"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_centerInParent="true"
        android:gravity="center"
        android:text="title"
        android:textSize="20sp"
        android:textStyle="bold" />

    <ImageView
        android:id="@+id/title_right"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_alignParentEnd="true"
        android:layout_alignParentRight="true"
        android:padding="10dp"
        android:src="@drawable/ic_keyboard_arrow_down_black_24dp" />

</RelativeLayout>
```
#### 自定义TitleBar类继承RelativeLayout重写父类方法
1. 解析自定义属性
2. 加载布局，设置默认属性
3. 暴露公共方法，改变自定义属性

###### 解析自定义属性
- res->values->attrs.xml 用xml文件设置自定义属性
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <declare-styleable name="TitleBar">
        <attr name="title_text_color" format="color" />
        <attr name="title_text" format="string" />
        <attr name="title_bg" format="color" />
    </declare-styleable>
</resources>
```
- 在`public TittleBar(Context context, AttributeSet attrs)`构造器中进行解析

```java
//解析自定义属性
//TypedArray is a indices used to retrieve values from this structrue
//从属性集attrs中提取相应的属性目录
TypedArray mTypedArray = context.obtainStyledAttributes(attrs, R.styleable.TitleBar);
//根据目录找到每一个属性
//get方法来自在attrs.xml中设置format 例如：format="color" 所以方法为getColor()
//得到布局中设置的自定义属性的值，如果没有设置，就返回为默认值，即第二个参数
mColor = mTypedArray.getColor(R.styleable.TitleBar_title_bg, Color.RED);
mTextColor = mTypedArray.getColor(R.styleable.TitleBar_title_text_color, Color.BLACK);
//回收哦，没用了就及时回收呀
mTypedArray.recycle();

```
###### 加载布局，应用属性
- 因为属于view的绘制，所以放在每一个构造器中调用
```java
 private void initView(Context context) {
        //加载布局
        LayoutInflater.from(context).inflate(R.layout.titlebar_layout, this, true);

        mIvLeft = findViewById(R.id.title_left);
        mIvRight = findViewById(R.id.title_right);
        mTvTitle = findViewById(R.id.title_title);
        mTitleLayout = findViewById(R.id.title_layout);

        //设置背景颜色
        mTitleLayout.setBackgroundColor(mColor);
        //设置字体颜色
        mTvTitle.setTextColor(mTextColor);
    }
```
###### 暴露公共方法，改变自定义属性
- 自定义了属性，总要让别人在activity里用java来动态改变吧！
- 还有监听器总要封装一下吧！
```java
 //三个方法供外部调用 设置标题，设置左右的监听器
    public void setTitle(String title) {
        if (!TextUtils.isEmpty(title)) {
            mTvTitle.setText(title);
        }
    }

    public void setLeftListener(OnClickListener onClickListener) {
        mIvLeft.setOnClickListener(onClickListener);
    }

    public void setRightListener(OnClickListener onClickListener) {
        mIvRight.setOnClickListener(onClickListener);
    }
```
## 直接继承ViewGroup (实现简单的ViewPager)
# 自定义View 迷惑行为
- inflate最后一个参数为true，一般情况都是flase
- 自动notitle？
    -继承的是Activity 没有titleBar，AppcompatActivity有Bar

- 自动margin？
- 初始化设置标题
- 多个构造器
