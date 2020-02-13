[官方教程](https://developer.android.google.cn/training/swipe)

-  swipeRefreshLayout 里面只能包含一个可以滑动的子布局

```xml
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swiperefresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</android.support.v4.widget.SwipeRefreshLayout>
```
可以换成androidx的控件

- SwipeRefreshLayout 包含一个CircleImageView
```
 Caused by: java.lang.ClassCastException: androidx.swiperefreshlayout.widget.CircleImageView cannot be cast to androidx.recyclerview.widget.RecyclerView
 ```

# 官方推荐两种手动刷新的方式

1. 下拉手势刷新
2. 添加menu action进行点击刷新

## 下拉手势刷新

```java
mSwipeRefreshLayout.setOnRefreshListener(
                new SwipeRefreshLayout.OnRefreshListener() {
                    @Override
                    public void onRefresh() {
                        myRefreshAction();//当所有刷新动作完成后调用 mSwipeRefreshLayout.setRefreshing(false)来取消刷新的效果
                    }
                }
        );
```
## 添加menu action进行点击刷新


```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:id="@+id/menu_refresh"
        android:showAsAction="never"
        android:title="@string/menu_refresh"/>
</menu>
```

```java
/*
 * Listen for option item selections so that we receive a notification
 * when the user requests a refresh by selecting the refresh action bar item.
 */
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()) {

        // Check if user triggered a refresh:
        case R.id.menu_refresh:
            Log.i(LOG_TAG, "Refresh menu item selected");

            // Signal SwipeRefreshLayout to start the progress indicator
            mySwipeRefreshLayout.setRefreshing(true);

            // Start the refresh background task.
            // This method calls setRefreshing(false) when it's finished.
            myUpdateOperation();

            return true;
    }

    // User didn't trigger a refresh, let the superclass handle this action
    return super.onOptionsItemSelected(item);
}
```

# SwipeRefreshLayout 上拉加载更多

实现原理：

dispatchTouchEvent 方法中进行手势的拦截

根据位置和距离，isLoading(是否正在加载) 判断需不需要触发监听器

```java

public class MyRefreshLayout extends SwipeRefreshLayout {

    private OnLoadMoreListener mLoadMoreListener;
    private RecyclerView mListView;
    private float mDownY, mUpY;
    private boolean isLoading;
    private View mFooter;
    private int mScaledTouchSlop;
    private int mItemCount;

    public MyRefreshLayout(@NonNull Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        mFooter = View.inflate(getContext(), R.layout.item_home_footer, null);
        mScaledTouchSlop = ViewConfiguration.get(context).getScaledTouchSlop();

    }

    public void setOnLoadMoreListener(OnLoadMoreListener listener) {
        mLoadMoreListener = listener;
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {

        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:

                mDownY = ev.getY();
                break;

            case MotionEvent.ACTION_MOVE:
                if (canLoadMore()) {
                    loadData();
                }
                break;
            case MotionEvent.ACTION_UP:
                mUpY = getY();

                break;
        }
        return super.dispatchTouchEvent(ev);
    }

    private void loadData() {
        if (mLoadMoreListener != null && !isLoading) {
            setLoading(true);
            mLoadMoreListener.onLoadMore();
        }
    }

    private boolean canLoadMore() {
        mListView = (RecyclerView) getChildAt(0); 
        Log.e("SwipeRefreshLayout--->", mListView.toString());

        //1. 上拉
        boolean condition1 = (mDownY - mUpY) >= mScaledTouchSlop;
        if (condition1) {
            Log.e("SwipeRefreshLayout--->", "上拉.....");
        }

        //2. 最后一条
        boolean condition2 = false;
        if (mListView != null && mListView.getAdapter() != null) {
            LinearLayoutManager manager = (LinearLayoutManager) mListView.getLayoutManager();

            condition2 = manager.findLastVisibleItemPosition() <= (mListView.getAdapter().getItemCount() - 4);

        }
        if (condition2) {
            Log.e("SwipeRefreshLayout--->", "最后一条可见");
        }


        //3. 不是正在刷新
        boolean condition3 = !isLoading;
        if (condition3) {
            Log.e("SwipeRefreshLayout--->", "不是正在刷新");
        }

        return condition1 & condition2 & condition3;
    }



    public void setLoading(boolean loading) {
        isLoading = loading;
        mDownY = 0;
        mUpY = 0;
    }

    public interface OnLoadMoreListener {
        void onLoadMore();
    }
}

```