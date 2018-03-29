# Git使用说明
### 常用命令
创建分支： `$ git branch mybranch`

切换分支： `$ git checkout mybranch`

创建并切换分支： `$ git checkout -b mybranch`

更新master主线上的东西到该分支上：`$git rebase master`

切换到master分支：`$git checkout master`

更新mybranch分支上的东西到master上：`$git rebase mybranch`

提交：`git commit -a`

对最近一次commit的进行修改：`git commit -a –amend`

更新远程库到本地： `$ git fetch origin`

推送分支： `$ git push origin mybranch`

取远程分支合并到本地：` $ git merge origin/mybranch`


### 初始化git
    
    $ git init
    Initialized empty Git repository in C:/GitWorkspace/auto_deploy/.git/
### 查看分支
    $ git branch\
    >
    * dev   #有dev分支，*代表当前本地关联的是dev分支
    master  #有master分支

### 切换分支
    $ git checkout -b local   #创建并切换分支
    A       unit/generateca.py
    M       unit/kube-master.py
    Switched to a new branch 'local'

### 从远程拉取
    git fetch origin dev-remote:dev-local 从远程分支拉取,将远端分支dev-remote更新到本地分支dev-local
    fetch代码下来要git diff orgin/xx来看一下差异然后再合并
    git merge XXX

### 提交代码
    git commit XXX 
    git push -u origin dev 提交代码到dev分支


---
git checkout  remote-dev origin/dev
git merge  dev local
---