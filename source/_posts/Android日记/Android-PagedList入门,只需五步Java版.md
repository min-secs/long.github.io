---
title: PagedList入门
date: 2021-02-06 08:35:05
tags:
categories: Android日记
---



>最近有接到分页的需求, 想当然准备使用监听onScroll然后分段加载, 看到谷歌爸爸出品了Jetpack系列文章有Paging Lib于是点开看了下, 介绍的太复杂了有木有!!! room什么鬼, ViewModel云云, 抽丝剥茧
简单Demo入门, 记录如下

新建Activity, 我使用的Fragment+ViewModel的方式, 很方便
![Fragment+ViewModel](https://upload-images.jianshu.io/upload_images/2226681-74073a146e6e7448.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####1.导入依赖
```java
implementation 'android.arch.lifecycle:extensions:1.1.1'
implementation 'android.arch.paging:runtime:1.0.0'
implementation 'com.android.support:recyclerview-v7:27.0.2'
//可选, 我习惯使用约束布局
implementation 'com.android.support.constraint:constraint-layout:1.1.3'
```
版本不同方法可能有差异, 建议初学使用我这的版本, 方便跟着后面走
首先照着后面的代码复制一遍, 大概知道有哪些东西
####2.新建实体类
```java
class Concert {

    public int id;
    private String name;

    public Concert(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return "Concert{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }

    @Override
    public int hashCode() {
        return id + name.hashCode();
    }

    @Override
    public boolean equals( Object obj) {
        if (obj instanceof Concert){
            boolean b = ((Concert) obj).id == id && ((Concert) obj).name.equals(name);
            return b;
        }else {
            return false;
        }
    }
}

//使用到的布局R.layout.paged_list_fragment
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/pagedlist"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="0dp"
        android:layout_height="256dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:layout_marginEnd="8dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>

//R.layout.item_concert_list
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/name"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginBottom="8dp"
        android:text="TextView"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</android.support.constraint.ConstraintLayout>
```
####3.新建DataSource实现类
```java
import android.arch.paging.PositionalDataSource;
import android.support.annotation.NonNull;
import android.util.Log;

import java.util.ArrayList;
import java.util.List;

public class PositionPageDataSource extends PositionalDataSource<Concert> {

    @Override
    public void loadInitial(@NonNull LoadInitialParams params, @NonNull LoadInitialCallback<Concert> callback) {
        int requestedLoadSize = params.requestedLoadSize;
        Log.d("ldo","PositionPageDataSource loadInitial "+requestedLoadSize);
        List<Concert> subList = getSubList(0, requestedLoadSize);
        callback.onResult(subList,0,40);//初始化加载从index == 0开始 40假设为数据总数
    }

    @Override
    public void loadRange(@NonNull LoadRangeParams params, @NonNull LoadRangeCallback<Concert> callback) {
        Log.d("ldo","PositionPageDataSource loadRange");
        callback.onResult(getSubList(params.startPosition,params.loadSize));
    }

    public List<Concert> getSubList(int start, int end) {
        Log.d("ldo", "getSubList");
        ArrayList<Concert> strings = new ArrayList<>();
        for (int i = 0; i < end; i++) {
            strings.add(new Concert(start, "" + i));
        }
        return strings;
    }
}

//Factory
import android.arch.paging.DataSource;

public class DataSourceFactorty extends DataSource.Factory<Integer,Concert> {

    @Override
    public DataSource<Integer,Concert> create() {
        return new PositionPageDataSource();
    }
}
```
####4.新建Adapter
```java
import android.arch.paging.PagedListAdapter;
import android.support.annotation.NonNull;
import android.support.v7.util.DiffUtil;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;


public class ConcertAdapter extends PagedListAdapter<Concert, ConcertAdapter.ViewHolder> {

    ConcertAdapter() {
        super(showDiffCallBack);
    }

    static DiffUtil.ItemCallback<Concert> showDiffCallBack = new DiffUtil.ItemCallback<Concert>() {

        @Override
        public boolean areItemsTheSame(@NonNull Concert oldItem, @NonNull Concert newItem) {
            return (oldItem).id == (newItem).id;
        }

        @Override
        public boolean areContentsTheSame(@NonNull Concert oldItem, @NonNull Concert newItem) {
            return oldItem.equals(newItem);

        }
    };

    @NonNull
    @Override
    public ConcertAdapter.ViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        return new ViewHolder(LayoutInflater.from(parent.getContext()).inflate(R.layout.item_concert_list, null));
    }

    @Override
    public void onBindViewHolder(@NonNull ViewHolder holder, int position) {
        Concert concert = getItem(position);
        holder.v.setText(concert.getId()+"");
    }

    public class ViewHolder extends RecyclerView.ViewHolder {
        TextView v;

        public ViewHolder(@NonNull View itemView) {
            super(itemView);
            this.v = itemView.findViewById(R.id.textView);
        }

    }
}
```
####5.新建Fragment
```java
import android.arch.lifecycle.LiveData;
import android.arch.lifecycle.Observer;
import android.arch.paging.LivePagedListBuilder;
import android.arch.paging.PagedList;
import android.os.Bundle;
import android.support.annotation.NonNull;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import com.example.android.displayingbitmaps.R;


public class PagedListFragment extends Fragment {

 
    
    private RecyclerView recyclerView;
    public static final int INITIALLOADSIZE = 40;//初始加载数量
    public static final int RELOADSIZE = 20;//往下滑动加载数量
    private LiveData<PagedList<Concert>> data;
    private ConcertAdapter adapter;


    public static PagedListFragment newInstance() {

        return new PagedListFragment();
    }

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {

        View inflate = inflater.inflate(R.layout.paged_list_fragment, container, false);
       
        recyclerView = inflate.findViewById(R.id.recyclerView);
        return inflate;
    }
    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
    
        
        PagedList.Config config = new PagedList.Config.Builder()
                .setPageSize(INITIALLOADSIZE / 2) // 分页加载的数量
                .setEnablePlaceholders(false) // 当item为null是否使用PlaceHolder展示
                .setInitialLoadSizeHint(INITIALLOADSIZE) // 预加载的数量, 与分页加载的数量成倍数关系
//                    .setPrefetchDistance(5)
                .build();
        data = new LivePagedListBuilder(new DataSourceFactorty(), config).build();//可以放在ViewModel层处理
        data.observe(this, new Observer<PagedList<Concert>>() {
            @Override
            public void onChanged(@Nullable PagedList<Concert> concerts) {
                Log.d("ldo","onChanged "+concerts.size());
                adapter.submitList(concerts);//调用PagedListAdapter中的方法绑定
            }
        });
        adapter = new ConcertAdapter();
        recyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
        recyclerView.setAdapter(adapter);
    }

}
```
---
有一个大概认知之后简单介绍一下各个部分
- DataSource
数据处理核心, 最终回调到PagedList
- LivePagedListBuilder
关联DataSource和观察数据
- PagedList
可以理解为存储数据的集合容器, 不需要我们过多关注
- PagedListAdapter
配合recycleview使用

需要注意的是adapter.submitList(concerts)使用父类的方法设置集合, 采取自己设置list然后notifyDataChaged的方式, PositionPageDataSource里面的方法不会回调

---
####不同DataSource实现类的区别
目前有三种DataSource实现方式, 上面使用的是PositionalDataSource, 想必你已经看到效果了
- PositionalDataSource
适用于上拉加载更多的场景, 如京东商城列表浏览, 知乎等
- ItemKeyedDataSource
适用于下一part内容依赖当前某个key来获取, 如获得下一part的专辑列表, 需要传入当前歌手来获取
- PageKeyedDataSource
适用于有明显上下页的场景, 如支持上一章,下一章跳转可以使用此种方式
