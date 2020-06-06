Jetpack 的 Navigation 由三部分组成，
1. `Navigation Graph`
2. `NavHostFragment`
3. `NavController`

# `Navigation Graph`

这是一个新的XML资源文件格式，用来配置App内部的导航，规定界面之间如何跳转。
1. 需要在`res`下新建一个文件夹`navigation`，`Navigation Graph`需要定义在这个文件夹下
2. `<navigation>`标签下可以定义`<fragment>` `<activity>`两种子标签，表示一个节点`destination`
3. 三级标签有`<action>` `<argument>` `<deeplink>`

## Actions
一个`Action`代表一次界面的跳转



# `NavHostFragment`
这是一个`Fragment`和 `Navigation Graph`配合来作为一个容器来承载 `graph` 中所有的节点

```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.example.android.codelabs.navigation.MainActivity">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@color/colorPrimary"
        android:theme="@style/ThemeOverlay.MaterialComponents.Dark.ActionBar" />

    <fragment
        android:id="@+id/my_nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        app:defaultNavHost="true"
        app:navGraph="@navigation/mobile_navigation" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_nav_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:menu="@menu/bottom_nav_menu" />
</LinearLayout>
```
`app:defaultNavHost="true"`表示是否和系统的返回键相关联
`app:navGraph="@navigation/mobile_navigation"` 指定对应的`navigation`文件


# `NavController`
控制界面的跳转
```kotlin
val button = view.findViewById<Button>(R.id.navigate_destination_button)
        button?.setOnClickListener {
            findNavController().navigate(R.id.flow_step_one_dest, null)
        }
```
增加界面跳转动画

```kotlin
 val options = navOptions {
    anim {
        enter = R.anim.slide_in_right
        exit = R.anim.slide_out_left
        popEnter = R.anim.slide_in_left
        popExit = R.anim.slide_out_right
    }
}
view.findViewById<Button>(R.id.navigate_destination_button)?.setOnClickListener {
    findNavController().navigate(R.id.flow_step_one_dest, null, options)
}
```
- 调用NavHostFragemnt中的findNavController
- 每个Navigation Graph对应一个NavController，这个NavController存储在NavHostFrgment中。
```kotlin
fun Fragment.findNavController(): NavController =
        NavHostFragment.findNavController(this)
```

- 参数的类型安全
为了保证参数的类型安全，使用了一个插件`safe args`
它会自动生成一个与相应的Fragment对应的FragmentArgs类
```kotlin
 val safeArgs: FlowStepFragmentArgs by navArgs()
 val flowStepNumber = safeArgs.flowStepNumber
```

- 传递参数进行跳转

通过自动生成的`Direction`类来为`action`添加参数，保证类型安全
```kotlin
val flowStepNumberArg = 1
val action = HomeFragmentDirections.nextAction(flowStepNumberArg)
Navigation.createNavigateOnClickListener(action)
```

# `NavigationUI`
将与导航相关的控件与`NavController`绑定，如：`bottomNavigation` `over flow menu` `Drawer`
