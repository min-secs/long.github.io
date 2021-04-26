---
title: 蓝鲸Record
date: 2021-02-06 08:35:05
categories: companies
---



# **一.** ***\*Password\****

 

## ***\*wifi密码\****

bw1308&&

 

## ***\*pin\****

123456

tannima

 

## ***\*邮箱\****

tanlong@bwic.cn

密码：Ttannima99

先登录阿里云绑定手机修改密码。~

https://qiye.aliyun.com

 

## ***\*confluence&jira账号：\****

账号：tanlong

密码：bwtanlong123

confluence地址： http://120.79.4.206:8090

jira地址： http://120.79.4.206:8080

 

## ***\*文档资料：\****

confluence培训资料：http://120.79.4.206:8090/x/_oAO

jira培训资料：http://120.79.4.206:8090/x/-IAO

 

 

## ***\*gerrit账号：\****

tanlong 

tlong@app6

 

## ***\*服务器IP：192.168.0.200\****

账号：tanlong

密码：bw123456

## ***\*Ubuntu\****

Boom

tannima

 

Ubuntu2 

Tan

 

## ***\*Jenkins\****

jenkins网址：http://120.77.175.62:8081
samba：\\192.168.0.200\release
谭龙
用户名：tanlong
密码：bwtanlong567

## ***\*ftp\****

  ftp://120.77.175.62/
  账号：a100_download
  密码：123456

 

乱码解决

选择时间区域, 设置兼容全球utf-8

 ## 公司百度网盘

（手机号是17774990251）

  密码：bw1303&&

 

# **二.** ***\*Git\****

**代理**

1.打开ipaddress.com网站

2.输入GitHub.com或者fastly.net查询IP

3.管理员打开C:\Windows\System32\drivers\etc\hosts

4.199.232.69.194 fastly.net

---

Q: github.io打不开

A: 按如上配置自己的域名 185.199.109.153 min-secs.github.io



**提交**

​	git commit 

​	

**创建新分支**,并切到该分支

​	git co -b bugFix 

​	

**合并**

​	git merge bugFix 合并分支到master

​	git rebase master 类似于线性提交

 

将**master**强制指向head的第三级父提交(不能在当前分支操作当前分支)

​	git branch -f master HEAD~3 

​	

**撤销**

​	git reset <hash> 本地

​	git revert <hash> 远程

​	

**整理提交记录**

​	git cherry-pick <hash> 知道hash的情况比较方便

​	git rebase -i <HEAD~3>

​	

打**TAG**

​	git tag v1 <hash>

 

**克隆一个仓库**

​	git clone git://

​	

**获取远程仓库并合并到本地分支**

​	git pull

​	git pull --rebase 

​	

**上传到远程仓库**

​	git push //和push.default文件有关

​	

**跟踪远程分支**

​	git co -b foo o/master

​	git branch -u o/master foo (原来的跟踪状态不会同步更新)

 

**设置别名**

git config --global alias.co "checkout" //使用git co 代替 checkout

 

**图形化界面**

gitk --all

**覆盖上次提交**

git commit –amend

 

**对比**

Git log xx

Git log -p xx //比较内容

 

**配置缩写**

git config --global alias.st status

 

**查看git地址**

git remote show origin

 

**回退到指定**commit

git reset --hard commit_id



**放弃本地强制和远程一致**

git fetch --all 

git reset --hard origin/dev

git pull

 

 ***\*遇到的错误\****

\* git clone错误之"error: RPC failed; curl 56 OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 10054"

A1: git config --global http.sslVerify "false" //验证通过

A2: git config http.postBuffer 524288000 //暂未验证

 

# **三.** ***\*Gerrit\****

配置用户名,邮箱

git config --global user.email "email"

git config --global user.name "username"

查看邮箱

git config user.name

git config user.email

gerrit中配置邮箱

 

忽略文件

/build

/src/test/

/src/androidTest/

/node_modules/

*.iml

*.json

 

提交前配置格式化

​	工程项目仓库，使用 npm init --yes

​	commitizen init cz-conventional-changelog --save --save-exact

 

git cz  //配置提交 git commit --amend //二次提交,未push之前

 

//提交审核 对应分支 master

git push origin HEAD:refs/for/Platform_Dev  

 

 

 

 

# ***\*四.AndroidStudio\****

\1. 修改 classpath 'com.android.tools.build:gradle:3.5.3'

 

2.gradle-wrapper.properties distributionUrl=https\://services.gradle.org/distributions/gradle-5.4.1-all.zip

 

3.阿里云镜像maven { url 'http://maven.aliyun.com/nexus/content/groups/public/' }

 

​    maven { url 'http://maven.aliyun.com/nexus/content/repositories/jcenter' }

\4. 本地缓存jar包

***\*c盘/用户/用户名/.gradle/caches/modules-2\****

AS各方库,版本号要一致

\5. 定义Live temp生成时间

Setting/Editor/Live Templates 

定义变量 $data$ 选择Edit variables date()函数

***\*项目根目录\****

gradlew -q :mediaplayer:dependencies //查看依赖

Q: INSTALL_FAILED_TEST_ONLY: installPackageLI

A: AS3.0以上打包的apk会自动添加test_only=false属性,

android.injected.testOnly=false //gradle.pro中添加

 

 

 

 

 

 

 

Q: INSTALL_FAILED_NO_MATCHING_ABIS: Failed to extract native libraries, res=-113

A: app.gradle

Android{

...

splits {
  abi {
    enable true
    reset()
    include 'arm64-v8a' , 'x86'
    universalApk true
  }
}

 

}

 

导出javadoc

-encoding utf-8 -charset utf-8

# **四.** ***\*访问数据库\****

cd  /data/user_de/0/com.android.providers.media/databases

cd /data/data/com.android.providers.media/databases

ls

sqlite3 external.db 进入数据库

.tables //列出所有表

 .mode column  显示的列会对齐

.header on //显示列名

 select * from 表

 

# **五.** ***\*Ubuntu\****

a100-user-debug

 

Q: 启动Ubuntu黑屏

A: 关闭 3D加速器

 

Q: git安装失败，git : 依赖: liberror-perl 但无法安装它

A: sudo apt-get update

sudo apt-get install git

 

Q: 安装vmTools提示空间不足

A: 拷贝到桌面在提取

 

Q: E: 无法获得锁 /var/lib/apt/lists/lock - open (11 资源临时不可用)

A: sudo rm /var/lib/apt/lists/lock

 

Q: 正在读取软件包列表... 有错误

A: sudo rm /var/lib/apt/lists/* -vf
sudo apt-get update //耗时较久

 

Q: E: 无法获得锁 /var/lib/dpkg/lock-frontend - open (11: 资源暂时不可用

A: sudo rm /var/cache/apt/archives/lock

sudo rm /var/lib/dpkg/lock

sudo rm /var/lib/dpkg/lock-frontend //验证通过

 

Q: git commit 无法保存

A: ctrl+x 然后按Y

# **六.** ***\*Vim\****

* 查看

vi xx.sh +22 //跳到指定行

* 删除

当前行 dd

全选ggvG 按d删除

* 创建文件

touch + xx.tt

* 查找

/xx  查找某字符, n 向下找

* 替换

:s/from/to/  替换当前行第一个

:s/from/to/g  替换当前行所有

* Mk转bp

out/soong/host/linux-x86/bin

如果没有androidmk  使用m -j blueprint_tools

androidmk  android.mk > android.bp

* 查找

  find prebuilts/sdk/ -name Android.bp|xargs grep "name.*androidx" 

# **七.** ***\*Shell\****

#### 查看cpu信息

![img](file:///C:\Users\admin\AppData\Local\Temp\ksohtml30816\wps2.jpg) 

adb shell

cat /proc/cpuinfo  AArch64对应arm64-v8a

 

#### 查找文件

busybox find . -name filename

 

#### 抓取日志

adb pull /data/misc/bwlog/

 

#### 查看当前占用

Lsof | grep 路径

 

#### 查看settings 属性

Settings.System.**getString**(context.getContentResolver(), CustomViewUtils.**NAME_BW_THEME_COLOR**);

settings get SYSTEM bw_theme_color

#### 查看当前源

settings get system bw_current_source_name

#### 查看文件大小

 busybox du -h -d 1

#### 模拟发送广播

adb shell am broadcast -a com.android.action.xx --es test_string "this is test string" --ei test_int 100 --ez test_boolean true

**模拟mode**

./vendor/bin/hw/VehicleSimulateMcu 0 11 7 0 19 4;./vendor/bin/hw/VehicleSimulateMcu 0 11 7 1 19 4;

# 八. 烧录

文件含义

# 九. python

使用镜像

pip install xx -i https://pypi.tuna.tsinghua.edu.cn/simple/

# 十. RDS

AF: 自动搜寻当前电台的其他频道, 比如从深圳开到了北京, 当前频点无信号,自动搜索同电台的另一个频道

TA: 电台信息等, 自动搜索同类型电台