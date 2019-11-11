# 功能

- 导航按钮navigation
- 应用程序的logo标识
- 标题和子标题
- 一个和多个自定义的视图控件
- 操作菜单



# 属性

- `android:theme`
 - `android:popuptheme`弹出菜单样式


# 隐藏系统自带导航栏
- 设置NoActionBar的Themes
- 在BaseActivity中调用`        supportRequestWindowFeature(Window.FEATURE_NO_TITLE);`继承于AppCompatActivity。如果继承于Activity则调用`requestWindowFeature(Window.FEATURE_NO_TITLE`

# 设置操作菜单menu

- 在res下创建menu文件夹

- 新建xml文件

- 设置item

- item属性

  ```
  - icon：设置图标当显示在外面时显示，被收起来时不显示
  - title：在外面时隐藏，收起来时显示
  - showAsAction：
    - always：显示在toolbar上
    - ifRoom：有空间的时候显示在toolbar上
    - collapseActionView：折叠
    - never：不显示
  - orderInCategory：顺序，数字小的在前面，大的在后面，默认为0 ，不设置则按布局顺序排列
  ```

- 填充inflate到toolbar上

```java
   		toolbar.inflateMenu(R.menu.toolbar_menu);
        toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {
                Toast.makeText(MainActivity.this, item.getTitle(), Toast.LENGTH_SHORT).show();
                //可以使用switch case
                return true;
            }
        });

```

# 设置Navigation

```java
// 设置icon
  toolbar.setNavigationIcon(R.drawable.star_yellow_16);
  toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "Navigation is onclick!", Toast.LENGTH_SHORT).show();
            }
        });
```

```xml
<!-- 在xml中设置icon-->
android:navigation="@drawable/icon"
```