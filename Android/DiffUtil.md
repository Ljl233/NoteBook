# DiffUtil 介绍

- 主要配合RecyclerView， 对新旧两个数据集进行对比，然后以最小的代价在旧数据集上进行变动和局部的刷新

RecyclerView提供了一系列的方法来进行局部更新，而且方便设置更新动画：
adapter.notifyItemChange()
adapter.notifyItemInserted()
adapter.notifyItemRemoved()
adapter.notifyItemMoved();
以上方法都是为了对数据集中，单一项进行操作，并且为了操作连续的数据集的变动，还提供了对应的 notifyRangeXxx() 方法。

但对于比较复杂的recyclerView的更新操作，较为普遍，最方便的做法就是无脑调用 notifyDataSetChanged()，用于更新 adapter 的数据集。
但这样做有一些缺点：
- 不会触发 RecyclerView 的局部更新的动画。
- 性能低，会刷新整个 RecyclerView 可视区域。
一般比较高效的做法就是自己写一个比较的方法来进行对比，再调用对应的方法进行局部的更新。

为了解决这种问题，google官方发布了DiffUtil，封装了对与两个数据集的比较过程

# DiffUtil 使用

DiffUtil可以很方便地对两个数据集进行比较，计算出变动情况，配合RecyclerView的adapter，根据变动情况，调用adapter的对应方法。

DiffUtil 不仅只能配合 RecyclerView 使用，它实际上可以单独用于比对两个数据集，然后如何操作是可以定制的

DiffUtil 在使用起来，主要需要关注几个类：
- DiffUtil.Callback：具体用于限定数据集比对规则。
- DiffUtil.DiffResult：比对数据集之后，返回的差异结果。

### DiffUtil.Callback

DiffUtil.Callback 主要就是为了限定两个数据集中，子项的比对规则。毕竟开发者面对的数据结构多种多样，既然没法做一套通用的内容比对方式，那么就将比对的规则，交还给开发者来实现即可。

在 Callback 中，其实只需要实现 4 个方法：
- getOldListSize()：旧数据集的长度。
- getNewListSize()：新数据集的长度
- areItemsTheSame()：判断是否是同一个Item。
- areContentsTheSame()：如果是同一个Item，此方法用于判断是否同一个 Item 的内容也相同。

areItemsTheSame()主要是用来判断是否属于一个ViewType
当areItemsTheSame(）判断相同时，再通过areContentsTheSame()判断两个item的内容是否相同。


实际上，这个Callback抽象类还有一个方法getChangePayload()，这个方法的作用是我们可以通过这个方法告诉Adapter对这个Item进行局部的更新而不是整个更新。

先要知道这个payload是什么？payload是一个用来描述Item变化的对象，也就是我们的Item发生了哪些变化，这些变化就封装成一个payload，所以我们一般可以用Bundle来充当。

接着，getChangePayload()方法是在areItemsTheSame()返回true，而areContentsTheSame()返回false时被回调的，也就是一个Item的内容发生了变化，而这个变化有可能是局部的（例如微博的点赞，我们只需要刷新图标而不是整个Item）。所以可以在getChangePayload()中封装一个Object来告诉RecyclerView进行局部的刷新。

假设上例中学号和姓名用不同的TextView显示，当我们修改了一个学号对应的姓名时，局部刷新姓名即可（这里例子可能显得比较多余，但是如果一个Item很复杂，用处就比较大了）：

先是重写Callback中的该方法：


```java
 @Nullable
 @Override
 public Object getChangePayload(int oldItemPosition, int newItemPosition) {
     Student newStudent = newStudents.get(newItemPosition);
     Bundle diffBundle = new Bundle();
     diffBundle.putString(NAME_KEY, newStudent.getName());
     return diffBundle;
 } 
```
返回的这个对象会在什么地方收到呢？实际上在RecyclerView.Adapter中有两个onBindViewHolder方法，一个是我们必须要重写的，而另一个的第三个参数就是一个payload的列表：
```java
@Override
public void onBindViewHolder(RecyclerView.ViewHolder holder, int position, List payloads) {}
```
所以我们只需在Adapter中重写这个方法，如果List为空，执行原来的onBindViewHolder进行整个Item的更新，否则根据payloads的内容进行局部刷新：

```java
@Override
public void onBindViewHolder(RecyclerView.ViewHolder holder, int position, List payloads) {
    if (payloads.isEmpty()) {
        onBindViewHolder(holder, position);
    } else {
        MyViewHolder myViewHolder = (MyViewHolder) holder;
        Bundle bundle = (Bundle) payloads.get(0);
        if (bundle.getString(NAME_KEY) != null) {
            myViewHolder.name.setText(bundle.getString(NAME_KEY));
            myViewHolder.name.setTextColor(Color.BLUE);
        }
    }
}
```
![](https://images2015.cnblogs.com/blog/852227/201609/852227-20160901145317215-2028201275.gif)
### DiffUtil.DiffResult
DiffUtil.DiffResult 其实就是 DiffUtil 通过 DiffUtil.Callback 计算出来，两个数据集的差异。它是可以直接使用在 RecyclerView 上的。如果有必要，也是可以通过实现 ListUpdateCallback 接口，来比对这些差异的。

```java
DiffUtil.DiffResult diffResult = DiffUtil.calculateDiff(new DiffCallBack(oldDatas, newDatas), true);
diffResult.dispatchUpdatesTo(mAdapter);
```

calculateDiff 方法主要是用于通过一个具体的 DiffUtils.Callback 实现对象，来计算出两个数据集差异的结果，得到 DiffUtil.DiffResult 。

而 calculateDiff 的另外一个参数，用于标记是否需要检测 Item 的移动。

DiffUtil 使用的是 Eugene Myers 的差别算法，这个算法本身是不检查元素的移动的。也就是说，有元素的移动它也只是会先标记为删除，然后再标记插入。而如果需要计算元素的移动，它实际上也是在通过 Eugene Myers 算法比对之后，再进行一次移动检查。所以，如果集合本身已经排序过了，可以不进行移动的检查。

而 dispatchUpdatesTo() 就是将这个数据集差异的结果，通过 adapter 更新到 RecyclerView 上面。

![dispatchUpdatesTo()](https://upload-images.jianshu.io/upload_images/1420036-d6a3ebbe171022ae.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

# 代码示例
1、实现 DiffUtil.Callback
为了简单，RecyclerView 中使用单一 ViewType ，并且使用一个TextView 承载一个 字符串来显示。

那么我们开始实现 Callback：
![Callback](https://upload-images.jianshu.io/upload_images/1420036-303907472b2df8d1.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

2、切换数据集
既然已经有了 DiffUtil.Callback 的实现之后，我们就需要对切换数据集的点击事件进行处理了。
![use](https://upload-images.jianshu.io/upload_images/1420036-76c5bd2b0aa3207b.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

3、效果
![result show](https://upload-images.jianshu.io/upload_images/1420036-df917ddc0ba771b9.gif?imageMogr2/auto-orient/strip|imageView2/2/w/380/format/webp)

> 参考资料：
> https://www.jianshu.com/p/b9af71778b0d
> https://www.cnblogs.com/Fndroid/p/5789470.html