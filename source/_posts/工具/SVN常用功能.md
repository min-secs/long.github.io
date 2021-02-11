---
title: svn常用功能
date: 2021-02-06 08:35:05
tags: 
categories: 工具



---



1. checkout
   拷贝远程仓库(后续简称仓库)代码到本地
2. update
   拉取仓库代码,同步更新本地代码
3. commit
   提交本地代码到仓库
4. show log
   查看仓库更改记录
5. revert
   5.1 revert to this version
   还原仓库代码到指定版本,修改冲突后需要再次提交覆盖仓库
   常用于代码回退
   5.2 revert changes from this version
   去掉此次修改记录,如遇到了一个bug,增加了日志,提交了一个版本A,修复bug后提交了一个版本B,此时不想要之前的打印信息,可以选中版本A revert changes from this version
   需要在提交一份覆盖仓库