---
title: Xutils3数据库升级
date: 2020-02-06 08:35:05
tags: 
categories: Android日记






---



>前言:xutils真是一款不错的android开发框架，在使用过程中减少了程序员很多的代码量。不过其中也有一小部分需要注意的地方。其它使用请看[xutils3详细用法](http://www.jianshu.com/p/95b6c4d7b7ce)

## 1.注解注意事项
不像黄牛刀的注解，xutils的注解是在运行时（ps：我也不懂），用xutils注解点击事件发现，在频繁的切换点击两个button的时候，只会响应一个button的点击，用findviewbyid的方式设置onclicklitsener，就不会有这个bug。

所以我们在用到点击功能的时候，恰当的用一下注解，目前只发现频繁点击会出现问题，不一定其它地方埋着什么。
## 2.数据库升级
当我们的业务在一天天完善的时候，之前建立的数据库字段可能需要做修改。我们如下配置数据库的代码
          

        DbManager.DaoConfig daoConfig =newDbManager.DaoConfig()
    
        .setDbName("myapp.db")//设置数据库名
    
        xutils.db.setDbDir(newFile("/mnt/sdcard/"))//设置数据库路径，默认存储在app的私有目录
    
        .setDbVersion(2)//设置数据库的版本号
    
        .setDbOpenListener(newDbManager.DbOpenListener() {//设置数据库打开的监听
    
            @Override
            public void onDbOpened(DbManager db) {//开启数据库支持多线程操作，提升性能，对写入加速提升巨大
            db.getDatabase().enableWriteAheadLogging();
    }
    })
        .setDbUpgradeListener(newDbManager.DbUpgradeListener() {//设置数据库更新的监听
    
            @Override
            public void onUpgrade(DbManager db,intoldVersion,intnewVersion) {
    
    }
    
    })  .setTableCreateListener(newDbManager.TableCreateListener() {//设置表创建的监听
            @Override
            public void onTableCreated(DbManager db, TableEntity table){
            Log.i("JAVA","onTableCreated："+ table.getName());
    }
    });

***
我们可以在setDbVersion（x）填上任意数字，当然根据我们自己的实际情况

在需要改字段的时候，我们可以填x+n的数字,然后在

    .setDbUpgradeListener(newDbManager.DbUpgradeListener() {//设置数据库更新的监听
    
    @Override 
    public void onUpgrade(DbManager db,intoldVersion,intnewVersion) {
    
      //不需要之前的数据
    
      db.delete(x.class);
    
      //需要之前的数据
    
      db.addColumn(x.class,"test");//新增的字段
    
      db.saveOrUpdate(db.findall（）);//当前表中有这条isId则更新数据，没有则添加
    
    }
    
    })

***
感谢android，感谢开源
