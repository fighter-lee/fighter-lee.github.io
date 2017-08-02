---
layout:     post
title:      "git命令操作"
subtitle:   "简单实用"
date:       2016-05-25
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

> (注：Windows环境)


1、创建一个文件夹test，并在test下创建一个a.md文件，然后在git bash下用cd命令进入到刚才创建的test文件夹，

2、初始化 git 仓库

    git init  
3、查看状态

    git status  

4、把a.md文件添加到本地Git仓库

	git add a.md  

5、设置下自己的用户名与邮箱

    git config —global user.name "fighter-lee"   
    git config —global user.email "fighterlee1@gmail.com"  

6、正式提交文件

    git commit -m ‘first commit’  
    -m 代表是提交信息

7、查看所有产生的 commit 记录

    git log  

8、把本地 test 项目与 GitHub 上的 test 项目进行关联（切换到 test 目录）

    git remote add origin git@github.com:JasonLi-cn/test.git  
    （查看我们当前项目有哪些远程仓库）
    
    git remote -v  

9、向远程仓库进行代码提交（前提是你已经配置好公钥和密钥，配置方法见第三部分）
    
    git push origin master  

###**提交时，可能出现的问题：**

    $ git push origin master  
    To github.com:JasonLi-cn/test.git  
     ! [rejected]master -> master (fetch first)  
    error: failed to push some refs to 'git@github.com:JasonLi-cn/test.git'  
    hint: Updates were rejected because the remote contains work that you do  
    hint: not have locally. This is usually caused by another repository pushing  
    hint: to the same ref. You may want to first integrate the remote changes  
    hint: (e.g., 'git pull ...') before pushing again.  
    hint: See the 'Note about fast-forwards' in 'git push --help' for details.  
说明在远程仓库中存在本地仓库没有的文件，所以需要先pull操作

    git pull origin master  

此时可能会遇到的问题：

    $ git pull origin master  
    From github.com:JasonLi-cn/test  
     * branchmaster -> FETCH_HEAD  
    fatal: refusing to merge unrelated histories  

解决方法：
    
    git pull origin master --allow-unrelated-histories  
然后就可以 push了！！！

###三、公钥和密钥配置方法
在Git bash中执行：

    ssh-keygen -t rsa  

会生成两个文件 id_rsa 和 id_rsa.pub ， id_rsa 是密钥，id_rsa.pub 就是公钥。

第一步先在 GitHub 上的设置页面，点击最左侧 SSH and GPG keys ，然后点击右上角的 New SSH key 按钮，

在 Key 那栏把 id_rsa.pub 公钥文件里的内容复制粘贴进去就可以了，

Title 那栏不需要填写，点击 Add SSH key 按钮。

SSH key 添加成功之后，输入

`ssh -T git@github.com`  

  进行测试，如果出现以下提示证明添加成功了。

    $ ssh -T git@github.com
    Hi JasonLi-cn! You've successfully authenticated, but GitHub does not provide shell access.  

###四、其它常用命令
    [plain] view plain copy
    git branch aaa 新建分枝aaa  
    git branch 查看分枝  
    git checkout aaa 切换到分枝aaa  
    git checkout -b aaa 新建并切换到aaa  
    git merge aaa 把aaa分支的代码合并过来(当前所在分枝，比如master)  
    git branch -d aaa 删除分枝aaa  
    git branch -D aaa 强制删除aaa  
    git tag v1.0 加版本号  
    git tag 查看版本号  
    git checkout v1.0 切换到版本v1.0  