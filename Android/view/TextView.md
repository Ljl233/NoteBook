# ellipsize 属性
```
TextView中可以设置一个ellipsize属性,作用是当文字长度超过textview宽度时的显示方式:

例如，"encyclopedia"显示, 只是举例，以实际显示为准：）

Android:ellipsize=”start”—–省略号显示在开头 "...pedia"

android:ellipsize=”end”——省略号显示在结尾  "encyc..."

android:ellipsize=”middle”—-省略号显示在中间 "en...dia"

android:ellipsize=”marquee”–以横向滚动方式显示(需获得当前焦点时)

对于使用marquee即滚动显示方式的，需要当前textview获得焦点才会滚动。所以有时可能因为实际需要，textview未获得焦点或者需要多个textview都同时滚动显示时，可以采用以下办法：

因为判断textview是否处于focused状态是通过它本身isFocused()方法，这样只要此方法返回为true时，即认为处于focused的状态，就可以滚动啦。

所以可以通过继承TextView类，并overrideisFocused()方法来控制是否滚动咯。

另外如果是组合View，外层layout需要加入以下属性来保证focus状态的传递：addStatesFromChildren="true"

作者：NeWolf
链接：https://www.jianshu.com/p/211e5e4137fe
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```