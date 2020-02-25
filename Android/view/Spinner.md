# Spinner 使用

1. xml

```xml
<Spinner
    android:id="@+id/spinner"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"/>
```
```xml
android:prompt="@string/app_name"//提示信息，只有在dialog模式下才显示
android:spinnerMode="dialog"     // dialog/dropdown（默认）两种模式，
android:dropDownVerticalOffset="50dp" //垂直偏移量
android:dropDownHorizontalOffset=”50dp”//水平偏移量
```


2. string-array


`res/values/strings.xml`
```xml
<resources>
    <string name="app_name">SpinnerDemo</string>

    <string-array name="plants_array">
        <item>Mercury</item>
        <item>Venus</item>
        <item>Earth</item>
        <item>Mars</item>
        <item>Jupiter</item>
        <item>Saturn</item>
        <item>Uranus</item>
        <item>Neptune</item>
    </string-array>
</resources>
```

3. SpinnerAdapter(interface)

```java
ArrayAdapter<CharSequence> arrayAdapter =
                ArrayAdapter.createFromResource(this,
                        R.array.plants_array,
                        android.R.layout.simple_spinner_item);

arrayAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);//动态改变item的布局，覆盖adapter初始化时添加的layout

mSpinner.setAdapter(arrayAdapter);
```


4. onItemSelectListener

```java
mSpinner.setOnItemSelectedListener(new AdapterView.OnItemSelectedListener() {
        @Override
        public void onItemSelected(AdapterView<?> parent, View view, int position, long id) {
            String itemString = mSpinner.getSelectedItem().toString();//获取选中值
            //Activity.this.getResources().getStringArray(R.array.spinner_array)[position];
            Toast.makeText(MainActivity.this, "onItemSelected", Toast.LENGTH_SHORT).show();
        }

        @Override
        public void onNothingSelected(AdapterView<?> parent) {
            Toast.makeText(MainActivity.this, "onNothingSelected", Toast.LENGTH_SHORT).show();
        }
});
```

# 显示
- 下拉框遮住了原来的框 =》设置偏移量
- 下拉框的宽度=spinner的宽度-下拉图标的宽度，宽度设置了matchparent也一样
- spinner加载的时候会自动点击第一个选项，回调`onItemSelected()`


# 改变背景

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape>
            <stroke
                android:width="1dip"
                android:color="#306932" />

            <corners android:radius="5dip" />

            <solid android:color="#ffffff" />
        </shape>
    </item>
    <item
        android:width="20dp"
        android:height="20dp"
        android:drawable="@drawable/ic_arrow_drop_down_black_24dp"
        android:gravity="right|center_vertical"
        android:right="10dp" />          //如果图片是png可以直接使用bitmap标签
</layer-list>
```