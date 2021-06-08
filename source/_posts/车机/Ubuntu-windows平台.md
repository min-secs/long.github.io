---
title: Ubuntu - windows使用
date: 2021-05-27 08:35:05
categories: 车机

---

Win10下商城搜索Ubuntu

* sudo apt-get update
* mkdir ~/bin
* curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
* chmod a+x ~/bin/repo
* 配置git

  1、进入自己的linux的终端账号 

​      2、在自己账号的根目录中创建.ssh目录

​      3、输入 ~/.ssh$ ssh-keygen -t rsa 一路回车生成公约文件id_rsa.pub

​      4、cat id_rsa.pub 将输出的内容拷贝到gerrit的[SSH Public Keys](http://39.108.176.27:9090/#/settings/ssh-keys)中，并添加到账号中



执行时会出现错误“Cannot get http://gerrit.googlesource.com/git-repo/clone.bundle”

解决方法：

    从git上clone repo
    
        git clone https://gerrit-googlesource.lug.ustc.edu.cn/git-repo
    
    然后将git-repo里面的repo文件复制到bin目录，然后chmod a+x ~/bin/repo.   再在同步源码的工作目录新建.repo文件夹，把git-repo重命名为repo复制到.repo目录下，重新执行repo init即可。
————————————————
版权声明：本文为CSDN博主「haolask」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/haolask/article/details/102826032