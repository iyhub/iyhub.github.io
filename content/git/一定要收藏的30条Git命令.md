---
title: 一定要收藏的30条Git命令
date: 2023-03-18 16:37:12.321
updated: 2023-03-18 16:37:12.321
url: /archives/git-cmd
categories: ["日常开发", "基础" , "git"]
tags: 
---

# GIT命令
<pre>
- git clone：克隆仓库，作用:远端代码下载到本地
  示例: git clone xxx.com/yy.git
  示例解释: 从xxx.com/yy.git 克隆远端代码到当前目录
- git init：初始化仓库，作用:将本地未初始化的仓库标识为一个git 仓库
  示例: 在指定目录下执行git init 即可初始化
  示例解释: 在当前目录下初始化一个新的Git仓库
- git push：推送到远程仓库，作用：将本地仓库中的代码推送到远程仓库中。
  示例: git push origin master
  示例解释: 将本地master分支的代码推送到远程仓库origin中
- git pull：从远程仓库拉取最新的代码，作用：将远程仓库中的代码拉取到本地仓库中。
  示例: git pull origin master
  示例解释: 从远程仓库origin的master分支拉取最新代码到本地
- git branch：列出分支，作用：列出本地仓库中的所有分支。
  示例: git branch
  示例解释: 列出本地仓库中的所有分支
- git checkout：切换分支或还原文件，作用：切换本地仓库中的分支或还原工作区的文件。
  示例: git checkout branchname
  示例解释: 切换到名为branchname的分支
- git merge：合并分支，作用：将指定分支合并到当前分支中。
  示例: git merge branchname
  示例解释: 将名为branchname的分支合并到当前分支中
- git status：查看仓库状态，作用：查看工作区、暂存区和本地仓库中的文件状态。
  示例: git status
  示例解释: 查看当前Git仓库的状态
- git log：查看提交历史，作用：查看本地仓库中的提交历史记录。
  示例: git log
  示例解释: 查看本地Git仓库中的提交历史记录
- git remote：管理远程仓库，作用：添加、删除或查看远程仓库的信息。
  示例: git remote add origin xxx.com/yy.git
  示例解释: 添加一个名为origin的远程仓库，地址为xxx.com/yy.git
- git stash：暂存当前更改，作用：将当前工作区中的修改暂时存储，方便在其他分支上工作。
  示例: git stash
  示例解释: 暂存当前工作区中的修改
- git tag：标记提交，作用：为指定的提交打上标记。
  示例: git tag tagname
  示例解释: 为当前提交打上标记
- git diff：比较文件差异，作用：比较当前工作区中的文件和本地仓库中的文件的差异。
  示例: git diff filename
  示例解释: 比较当前工作区中的filename文件和本地仓库中的filename文件的差异
- git reset：取消暂存或还原到历史版本，作用：将文件从暂存区移回到工作区，或者将当前分支回退到指定的历史版本。
  示例: git reset HEAD filename
  示例解释: 将暂存区中的filename文件移回到工作区
- git rm：删除文件，作用：从本地仓库中删除指定的文件。
  示例: git rm filename
  示例解释: 删除本地仓库中的filename文件
- git mv：移动或重命名文件，作用：将指定的文件移动到新的位置或者重命名。
  示例: git mv oldfilename newfilename
  示例解释: 将旧文件名oldfilename改为新文件名newfilename
- git fetch：从远程仓库获取最新的提交，作用：从远程仓库获取最新的提交，但不会合并到本地仓库中。
  示例: git fetch origin
  示例解释: 从远程仓库origin中获取最新的提交
- git rebase：将当前分支的提交移到另一个分支之后，作用：将当前分支的提交移动到另一个分支之后，以便使分支更加整洁。
  示例: git rebase branchname
  示例解释: 将当前分支的提交移动到名为branchname的分支之后
- git cherry-pick：将指定的提交应用到当前分支上，作用：将指定的提交应用到当前分支上，以便使分支更加整洁。
  示例: git cherry-pick commitid
  示例解释: 将指定的commitid提交应用到当前分支上
- git submodule：管理子模块，作用：将一个Git仓库作为另一个Git仓库的子模块进行管理。
  示例: git submodule add xxx.com/yy.git
  示例解释: 将xxx.com/yy.git添加为当前Git仓库的子模块
- git config：配置Git环境，作用：配置Git的全局或本地环境变量。
  示例: git config --global user.name "your name"
  示例解释: 配置Git的全局环境变量，设置用户名为"your name"
</pre>
