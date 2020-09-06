---
title: 如何在一台电脑上管理/切换多个github账户
date: 2020-07-05 19:19:01
tags: [git, 多账号配置]
categories: git
---

> 场景：比如个人person和公司work的多个github账号，这个时候在本地做操作，是无法用work账号操作person账号下的git仓库的。

当我用work提交属于person的代码的时候，会出现以下报错，提示无权限.
```git
$ git push origin master
ERROR: Permission to person/git-start.git denied to work.
fatal: Could not read from remote repository.
```
<!-- more -->

使用git remote -v可查看当前仓库的远程git地址。
```git
$ git remote -v
origin git@github.com:person/git-start.git (fetch)
origin git@github.com:person/git-start.git (push)
```

那么，一台电脑上如何管理多个github账户呢?

--- 

### 设置SSH密钥
创建多个SSH密钥，并保存在对应的文件中
```git
cd ~/.ssh
ssh-keygen -t rsa -C "work@163.com"

ssh-keygen -t rsa -C "person@163.com"

...
```
以上创建出id_rsa_work, id_rsa_work.pub和id_rsa_person, id_rsa_person.pub四份文件

---

### 将SSH密钥添加到Github账户

将密钥复制到剪切板
```git
pbcopy < ~/.ssh/id_rsa_work.pub
```
将生成的密钥中的公钥内容(即.pub文件)的内容添加到不同的github账户中，流程如下:
* 转到github的帐户设置
* 点击“SSH密钥”，然后“添加SSH密钥”
* 将密钥粘贴到“密钥”字段并添加相关标题
* 点击“添加密钥”，然后输入您的Github密码进行确认


### 创建config配置文件来单独管理密钥
> $ cd ~/.ssh/
$ sudo vim config


编辑config文件
```git
# work
Host work
   HostName github.com
   User git
   IdentityFile ~/.ssh/id_rsa_work

# person
Host person
   HostName github.com
   User git
   IdentityFile ~/.ssh/id_rsa_person
```

添加新的密钥
```git
$ ssh-add id_rsa_work
$ ssh-add id_rsa_person
```

查看当前的密钥列表，查看是否添加成功
```git
$ ssh-add -l
```

测试以确保Github识别密钥：
```git
$ ssh -T work
Hi work! You've successfully authenticated, but GitHub does not provide shell access.

$ ssh -T person
Hi person! You've successfully authenticated, but GitHub does not provide shell access.
```

---

### 试一下
在和远程库交互的时候，还有一点要注意，即git仓库地址的更改。
首先，回到命令行上，创建一个测试目录：
```git
$ cd ~/documents
$ mkdir git-start
$ cd git-start
```

使用work账号，向Github添加一个空白的“readme.md”文件和PUSH：
```git
$ touch readme.md
$ git init
$ git add .
$ git commit -am "first commit"
$ git remote add origin git@work:work/git-start.git
$ git push origin master
```

注意我们如何使用自定义帐户git@work，而不是git@github.com!
对于git@work:work/git-start.git。第一个work是在config文件里创建的Host，第二个work为你github的用户名。
再试一下person的PUSH和PULL操作，看是否成功

<strong>tips：</strong>更改远程仓库的命令
```git
$ git remote set-url origin git@work:work/git-start.git
```

总结，一台计算机上管理多个github账户的核心就是：
- ssh密钥
- config文件配置
- git仓库远程地址的配置


> ps: git常用命令
* git branch branchName(在本地创建一个命名为branchName的分支)
* git branch 查看当前自己所在的分支
* git branch -a 查看服务器的所有分支以及自己当前所在的分支
* git checkout branchName 切换本地分支
* git push origin branchName(把命名为branchName的本地分支推送到服务器)
* git checkout --track origin/branchName (切换为远程服务器上的命名为branchName的远程分支)
* 如果你的搭档要把他本地的分支给关联到服务器上命名为branchName的远程分支，请执行以下操作，git branch --set-upstream localBranchName origin/branchName  （为本地分支添加一个对应的远程分支 与之相对应）->这行命令用来关联本地的分支与服务器上的分支
* git push origin branchName（提交代码到远程服务器上命名为branchName的分支上）
* git pull origin branchName （从远程分支上拉取代码）
















