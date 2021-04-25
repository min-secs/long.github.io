---
title: MVP小白入门,只需5步
date: 2020-02-06 08:35:05
tags: 
categories: Android日记

---



>前言:因为公司氛围没有用,一直很火的mvp模式像鬼一样听说过没见过,今天闲来无事了解一下,发现由普通的MVC改起来还是挺行云流水的,但像有些弹窗不知道写在P层还是V层
关于MVP的概念我就不详细说了,记住关键的一点就是将view与逻辑分离
在谷歌推荐写法下,M层被弱化,数据处理放在了P层
####1.定义一个P接口
```java
public interface BasePresenter {
    boolean something();
}
```
####2.定义一个V接口
```java
public interface BaseView<P extends BasePresenter> {
    void setPresenter(P presenter);
}
```
####3.定义一个合约类管理上述两个接口,Presenter用于定义界面的逻辑与数据,View 用于定义对应的界面显示
```java
public class DateRecordContract {
    interface Presenter extends BasePresenter {
        //获取今日数据
        void getTodayData();
        //获取所有数据
        void getAllData();
        //检查数据是否有效
        void checkData();
        //插入一条数据
        void insertDateBean();
        //开始计时
        void startTiming();
    }

    interface View extends BaseView<Presenter> {
        //显示计时界面
        void startAnim();
        //刷新界面
        void refreshUI(List<DateRecordBean> data);
        //停止计时界面
        void stopAnim();
        //刷新一个界面
        void refreshOneDate(DateRecordBean bean);
        //保存输入的文本
        void saveInputtext(String str);
        //获得当前的文本
        String getCurrenttext();
        //隐藏输入法
        void hideInput();
    }


}
```
####4.定义一个P层实现类,最好放在上面的接口同一个包下
```java
public class DateRecordPresenterImpl implements DateRecordContract.Presenter {
    private DateRecordContract.View view;
    private SharedPreferencesHelper spHelper;

    public DateRecordPresenterImpl(DateRecordContract.View view,SharedPreferencesHelper sp) {
        this.view = view;
        view.setPresenter(this);
        spHelper = sp;
    }

    @Override
    public void getTodayData() {
        //..处理数据
        List<DateRecordBean> list = ...;
        //..通知view刷新界面
        view.refreshUI(list);
    }

    @Override
    public void getAllData() {
        List<DateRecordBean> list = ...;
        //
        view.refreshUI(list);
    }

    @Override
    public void checkData() {

    }

    @Override
    public void insertDateBean() {
        long endTime = System.currentTimeMillis();
        DateRecordBean bean = new DateRecordBean();
        //..数据处理

        //处理完数据后通知view刷新界面
        view.refreshOneDate(bean);
    }

    @Override
    public void startTiming() {
        view.hideInput();
        view.startAnim();

        spHelper.put(SharedPreferencesHelper.isStartTime, true);

        view.saveInputtext(str);
    }
    @Override
    public boolean something() {
        //...自己的处理逻辑
        return isStartRecord;
    }
}
```
####5.定义一个View实现类,根据回调显示UI
```java
public class MainActivity extends AppCompatActivity implements DateRecordContract.View {
private DateRecordPresenterImpl presenter;
  @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        presenter = new DateRecordPresenterImpl(this,spHelper);
        //通知P层获得今日数据
        presenter.getTodayData();
    }


    @Override
    public void refreshUI(List<DateRecordBean> data) {
        //显示P层处理好的数据
        dapter.addBeans(data);
    }
}
```
---
以上就是简单的MVP入门,哪里出问题了直接在合约类查看,还是挺方便,也可以把MainActivity改为Fragment实现View,有些疑问是不知道把Dialog放在哪,目前还是放在了Activity中
2018年9月5日08:02:16
