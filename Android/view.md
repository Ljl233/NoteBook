![view](https://note.youdao.com/yws/res/3951/WEBRESOURCE7d17995ce87853c04166b8ecadb98362)

View 自身的坐标
- getTop() 获取View顶边到父布局顶边的距离

###  动画
##### 用xml文件设置动画
- res目录下新建anim文件，并创建`translate.xml`
- 在java代码中调用
- 在逐帧动画中view移动之后，对于屏幕的`位置参数`并没有改变，只是视觉上的改变（其他的不清楚），属性动画解决了这个问题



1. scrollTo() 和 scrollBy()
- scrollTo(x,y)将view移动到相对于起始点的相对位置，scroolBy(dx,dy)移动指定的偏移量
- 这两个方法可以看做是手机屏幕的滚动，偏移量应该设置为负值
- scroolTo是一次性的，scroolBy可以进行多次滚动，以scoolTo为基础

```
((View) getParent()).scrollBy(-offsetX, -offsetY);
```
2. `Scroller`
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