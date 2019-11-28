[ViewStub简单使用](https://www.jianshu.com/p/175096cd89ac)

```xml
 <ViewStub
        android:id="@+id/home_view_stub"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout="@layout/item_false_refresh" />
```

```java
 try {
            View view = viewStub.inflate();
        } catch (Error e) {
            viewStub.setVisibility(View.VISIBLE);
        }
```