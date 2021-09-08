---
title: Android studio
date: 2021年9月8日 
tags: 
categories: 工具


---





### 单例优雅模式
```java
private $NAME$(){}

public enum Singleton {
    INSTANCE;
    private final $NAME$ instance;

    Singleton() {
        this.instance = new $NAME$();
    }

    public $NAME$ getInstance(){
        return instance;
    }
}

public static $NAME$ getInstance(){
    return Singleton.INSTANCE.getInstance();
}
```

### 遇到的问题

1. Q1 AS提示乱码

A1 双击shift - edit custom vm options - -Dfile.encoding=UTF-8 重启

