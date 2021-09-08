---
title: Git常用功能
date: 2021-02-25 17:35:05
tags: 
categories: 工具
---

### **添加代理**

增快访问速度

1.打开ipaddress.com网站

2.输入GitHub.com或者fastly.net查询IP

3.管理员打开C:\Windows\System32\drivers\etc\hosts

4.199.232.69.194 fastly.net

---

Q: github.io打不开

A: 按如上配置自己的域名 

```text
140.82.114.4	github.com
185.199.109.153 github.global.ssl.fastly.net
```

### 配置用户名,邮箱

git config --global user.email "email"

git config --global user.name "username"

### 查看邮箱

git config user.name

git config user.email

### **设置别名**

git config --global alias.co "checkout" //使用git co 代替 checkout

git config --global alias.st status

### **查看当前配置**

 git config --global --list

### 克隆一个仓库

​	git clone git://

<!--more-->



------



### **获取远程仓库并合并到本地分支**

​	git pull

​	git pull --rebase 

### **提交**

​	git commit 



​	**覆盖上次提交**

git commit –amend

 

### **创建新分支**,并切到该分支

​	git co -b bugFix 

​	

### **合并**

​	git merge bugFix 合并分支到master

​	git rebase master 类似于线性提交

 

将**master**强制指向head的第三级父提交(不能在当前分支操作当前分支)

​	git branch -f master HEAD~3 

​	

### **撤销**

​	git reset <hash> 本地

​	git revert <hash> 远程

​	

### **整理提交记录**

​	git cherry-pick <hash> 知道hash的情况比较方便

​	git rebase -i <HEAD~3>

​	

### 打**TAG**

​	git tag v1 <hash>

 



​	

### **上传到远程仓库**

​	git push //和push.default文件有关

​	

### **跟踪远程分支**

​	git co -b foo o/master

​	git branch -u o/master foo (原来的跟踪状态不会同步更新)

 



### **图形化界面**

gitk --all



### **对比**

Git log xx

Git log -p xx //比较内容

 

### **查看git地址**

git remote show origin

git remote -v

 

### **回退到指定**commit

```
git reset --hard commit_id
```





### **放弃本地强制和远程一致**

```
git fetch --all 
```

``` 
git reset --hard origin/dev
```

```
git pull
```

### 清除untrack files

- 删除当前目录下untrack文件，不包括文件夹和.gitignore中指定的文件和文件夹

```
git clean -f
```

- 删除当前目录下untrack文件和文件夹， 不包括.gitignore中指定的文件和文件夹

```
git clean -df
```

 ### 遇到的错误

\* git clone错误之"error: RPC failed; curl 56 OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 10054"

A1: git config --global http.sslVerify "false" //验证通过

A2: git config http.postBuffer 524288000 //暂未验证