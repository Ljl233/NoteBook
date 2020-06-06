一个整个view，拆分成多个，在重新组合成一个新的view
视频和图片显示


mvvm

# bean


# layout

作者信息


在布局文件里绑定
app/build.gradle

```xml
databinding{

enabled = true
}
```

convern to databinding layout

```xml
<data>

<variable 
    name="user"
    type="User" />

<import type= "User"></import>

</data>

android:text="@{user.name}"

tools:text=""占位显示

对于图片资源的数据绑定

继承AppCompatImagetView
@BindingAdapter（value={“image_url”,"isCircle"}）
public static void setImageUrl(PPImageView view, String image-url,boolean isCircle){
    Glide.with(view).load(image_url).into(view);
}
1. 继承ImageView，写一个被@BindingAdapter注解标记的静态方法
2. 该注解有两个属性： 
    - value：一个属性数组，是对xml布局中的属性扩展，应作为参数传到静态方法中
    - requireAll： 默认true，所有的扩展属性被赋值才会调用此修饰方法，false，不需要全部属性

3. 在方法中利用url等属性，来进行处理
```
# MaterialButton

style parent="Theme.MaterialComponents.Light.NoActionBar


重写 Style Widget。MaterialCompoents.Button


<Space>