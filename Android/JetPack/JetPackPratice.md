

# <fragment>

```xml
android:name="androidx.navigation.fragment.NavHostFragment"    //指定当前fragment真正的实现类
app:defaultNavHost="true"  //是否和系统返回键相关联，看当前回退栈中还有没有fragment，有：返回fragment，没有执行系统的返回键
app:navGraph="@navigation/mobile_navigation" //路由结构
```

- xml/navigation/mobile_navigation.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/mobile_navigation"
    app:startDestination="@+id/navigation_home">

    <fragment
        android:id="@+id/navigation_home"
        android:name="com.example.ppjoke.ui.home.HomeFragment"
        android:label="@string/title_home"
        tools:layout="@layout/fragment_home">
        <argument                   ----------------------------------------->创建当前fragment需要的参数
            android:name="arg1"
            android:defaultValue="111"
            app:argType="string" />
        <action                     -------------------------------------------->当前fragment跳转到目标页的动作
            android:id="@+id/id_action"
            app:destination="@id/navigation_dashboard" />
        <deepLink                  ------------------------------------------》用uri跳转到当前页面，相当于隐式intent
            android:id="@+id/deepLink"
            app:uri="www.example.com" />
    </fragment>

    <fragment
        android:id="@+id/navigation_dashboard"
        android:name="com.example.ppjoke.ui.dashboard.DashboardFragment"
        android:label="@string/title_dashboard"
        tools:layout="@layout/fragment_dashboard" />

    <fragment
        android:id="@+id/navigation_notifications"
        android:name="com.example.ppjoke.ui.notifications.NotificationsFragment"
        android:label="@string/title_notifications"
        tools:layout="@layout/fragment_notifications" />
</navigation>
```
