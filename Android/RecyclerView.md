# 步骤
- 添加拓展
- 新建布局
    - item布局
    - 活动布局
- item数据类(Bean类)
- 适配器Adapter
- 主活动类
    - 布局管理器

# 使用示范



### 布局
#### item布局 rv_item.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="@dimen/md_common_view_height">
    <TextView
        android:id="@+id/item_tv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        tools:text="item"/>
</LinearLayout>
```

#### Activity主布局 activity.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <android.support.v7.widget.RecyclerView
        android:id="@+id/my_recycler_view"
        android:scrollbars="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```
###### RecyclerView属性
- scrollbars 
    - android:scrollbars=”vertical”是否显示滚动条，它的取值可以是vertical，horizontal或none。 
    - android:fadeScrollbars=”true”（默认参数）是在滑块不滚动时，隐藏 
    - android:fadeScrollbars=”false”是在滑块不滚动时，不隐藏 
    - android:scrollbarThumbVertical=”@drawable/ic_launcher”自定义滑块的背景图 
    - android:scrollbarStyle=”insideOverlay” 
        - insideOverlay：默认值，表示在padding区域内并且覆盖在view上 
        - insideInset：表示在padding区域内并且插入在view后面 
        - outsideOverlay：表示在padding区域外并且覆盖在view上 
        - outsideInset：表示在padding区域外并且插入在view后面

### 数据类 Bean.java
```
public class Bean{
    int num;
    //String name;
    int getNum(){
        return num;
    }
    void setNum(int num){
        this.num = num;
    }
    
}
```

### 主活动类

```
recyclerView = (RecyclerView) findViewById(R.id.recyclerView);  
LinearLayoutManager layoutManager = new LinearLayoutManager(this );  
//设置布局管理器  
recyclerView.setLayoutManager(layoutManager);  
//设置为垂直布局，这也是默认的  
layoutManager.setOrientation(OrientationHelper. VERTICAL);  
//设置Adapter  
recyclerView.setAdapter(recycleAdapter);  
 //设置分隔线  
recyclerView.addItemDecoration( new DividerGridItemDecoration(this ));  
//设置增加或删除条目的动画  
recyclerView.setItemAnimator( new DefaultItemAnimator());  
```

### Adapter类
```

public class TestAdapter extends RecyclerView.Adapter<TestAdapter.MyViewHolder> {

    //自定义ViewHolder
    public class MyViewHolder extends RecyclerView.ViewHolder {
        View itemView;
        TextView textView;
        ImageView imageView;

        public MyViewHolder(@NonNull View view) {
            super(view);
            itemView = view;
            textView = view.findViewById(R.id.item_tv);
            imageView = view.findViewById(R.id.item_iv);

        }
    }

    private List<Data> datas;

    //构造器
    public TestAdapter(List<Data> datas) {
        this.datas = datas;
    }

    //将item封装为一个ViewHolder
    @NonNull
    @Override
    public MyViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.rc_item, parent, false);
        return new MyViewHolder(view);
    }

    //适配渲染数据到View中
    @Override
    public void onBindViewHolder(@NonNull MyViewHolder holder, final int position) {
        holder.imageView.setImageResource(R.drawable.ic_launcher_background);
        holder.textView.setText(position);
        holder.itemView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(v.getContext(), "onclick!!!" + position, Toast.LENGTH_SHORT).show();
            }
        });

    }

    @Override
    public int getItemCount() {
        return datas.size();
    }

}
```
#### ViewHolder内部类
- 继承于`RecyclerView.ViewHolder`
- 初始化itemVitem及其内部