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