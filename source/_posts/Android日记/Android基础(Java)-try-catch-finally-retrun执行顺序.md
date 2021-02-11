---
title: Android基础-try-catch等执行顺序
date: 2021-02-06 08:35:05
tags: 
categories: Android日记
---



> 口诀,catch优先return, 无论怎样finally
2019年9月19日
```java
  private void go() {
        tag = "ldo-";
        Log.d(tag, "go: start");
        try {
            Log.d(tag, "go: try 01");
            arrayList.get(0);
            Log.d(tag, "go: return 01");
            return;
        } catch (Exception e) {
            Log.d(tag, "go: catch01");
            e.printStackTrace();
            try {
                arrayList = new ArrayList<>();
                arrayList.add(1);
            } catch (Exception e1) {
                e1.printStackTrace();
                Log.d(tag, "go: catch02");
            }

        } finally {
            Log.d(tag, "go: finally");
        }
    }
```
方法第一次执行
猜猜看打印什么
嘟嘟嘟嘟
.
.
.
.
.
.
.
.
.
.
.
.
.
.
.

.
.
.
..
.
.
.

公布
```
01-01 01:24:02.567 17772-17772/ D/ldo-: go: start
01-01 01:24:02.567 17772-17772/ D/ldo-: go: try 01
01-01 01:24:02.567 17772-17772/ D/ldo-: go: catch01
01-01 01:24:02.570 17772-17772/ D/ldo-: go: finally
```

---
第二次执行
嘟嘟嘟
.
.
.
.
.
.
.
.
.
.
.
.
.
.
.
```
01-01 01:27:29.302 17772-17772/ D/ldo-: go: start
01-01 01:27:29.302 17772-17772/ D/ldo-: go: try 01
01-01 01:27:29.302 17772-17772/ D/ldo-: go: return 01
01-01 01:27:29.302 17772-17772/ D/ldo-: go: finally
```
别忘了会执行finally哦~

---
最后考大家一个, 大家加油哦 ~. ~
```java
 private void go() {
        tag = "ldo-";
        Log.d(tag, "go: start");
        try {
            Log.d(tag, "go: try 01");
            arrayList.get(0);
            Log.d(tag, "go: return 01");
            return;
        } catch (Exception e) {
            Log.d(tag, "go: catch01");
            e.printStackTrace();
            try {
//                arrayList = new ArrayList<>();
                arrayList.add(1);
                Log.d(tag, "go: try 02");
                Log.d(tag, "go: return 02");
                return;
            } catch (Exception e1) {
                e1.printStackTrace();
                Log.d(tag, "go: catch02");
                return;
            }finally {
                Log.d(tag, "go: finally 02");
            }

        } finally {
            Log.d(tag, "go: finally 01");
        }
    }
```
