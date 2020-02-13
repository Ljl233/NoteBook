[官网链接](https://developer.android.google.cn/guide/topics/resources/drawable-resource?hl=en)

# layer-list
## 前言
写项目的时候碰到一个需求，要求屏幕的背景上面的有图的，下面是空白。但是可怜的是设计师给的只有上面的图，没有留白，如果直接当背景放上去，图片会被拉伸就会很丑，本来想让设计师再重新做一张图，但是一咬牙就自己用代码实现好了。给我一通好找。发现了layer-list可以实现这个需求。

实现思路：放置一个纯白的底层，露出白色部分。这样看起来就有白色的下面的部分了。（为什么不直接在下面加一个白色的，非要加一层呢？没找到这种的实现方法，以后再去看文档找找有没有这样的实现吧，总之能实现就好）

## 代码

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item >
        <shape>
            <solid android:color="#fff"/>
        </shape>
    </item>
    
    <item android:bottom="80dp">
        <bitmap android:src="@drawable/home_bg" />
    </item>
    
</layer-list>
```