---
layout: post
title: git
p: git
date: 2019-06-17 15:50:21
tags:
    - [ git ]
categories:
    - [git]
---

## git 
```
1、解决文件已经添加到暂存区进行回退

   git reset HEAD readme.txt(文件)  #取消暂存区文件修改
   git status  #查看当前文件状态
   git checkout -- readme.txt #回退到git add之前状态

   git log -n10 --oneline  #显示最近十次提交日志
   git diff old_id new_id --name-status #对比两次提交和文件修改及状态

2、查看git提交日志

   git log参数详解:
   -p  按补丁格式显示每个更新之间的差异。

   --stat   显示每次更新的文件修改统计信息。

   --shortstat   只显示 --stat 中最后的行数修改添加移除统计。

   --name-only   仅在提交信息后显示已修改的文件清单。

   --name-status   显示新增、修改、删除的文件清单。

   --abbrev-commit   仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。

   --relative-date  使用较短的相对时间显示（比如，“2 weeks ago”）。

   --graph    显示 ASCII 图形表示的分支合并历史。

   --pretty  使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。

   -(n)   仅显示最近的 n 条提交

   --since, --after   仅显示指定时间之后的提交。

   --until, --before   仅显示指定时间之前的提交。

   --author   仅显示指定作者相关的提交。

   --committer  仅显示指定提交者相关的提交。

   --grep  仅显示含指定关键字的提交

   -S  仅显示添加或移除了某个关键字的提交
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
   git log filename  #查看filename文件commit记录

   git log -p filename  #显示每次提交的diff

   git log  --pretty=format:"%cn committed %h on %cd"  #指定日志格式(%cn作者名字，%h缩略标识，%cd提交日期)

   git log  --author="manson"

   git log --pretty="%an - %s" --author=cuibobo --since="2018-10-01" --before="2018-12-27" --oneline #指定格式和日期进行日志查询
 
   git log #命令显示从最近到最远的提交日志，如果嫌输出信息太多，看得眼花缭乱的，可以试试加上--pretty=oneline

   git reset --hard HEAD^ #用HEAD表示当前版本（注意我的提交ID和你的肯定不一样），上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100

  git reset --hard commit_id   #通过commit id来回溯

  git reflog #Git提供了一个命令git reflog用来记录你的每一次命令

  HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令git reset --hard commit_id。

  穿梭前，用git log可以查看提交历史，以便确定要回退到哪个版本。

  要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本。

  git remote add origin git@server-name:path/repo-name.git  #添加远程仓库
 
  git push -u origin master #第一次推送master分支的所有内容；

  git push origin master #提交以后，推送最新修改

3、git使用分支

  Git鼓励大量使用分支：

  查看分支：git branch

  创建分支：git branch <name>

  切换分支：git checkout <name>

  创建+切换分支：git checkout -b <name>

  合并某分支到当前分支：git merge <name>

  删除分支：git branch -d <name>

4、合并和解决冲突

  git log —-graph —-pretty=oneline —abbrev-commit #查看执行日志

  git merge abort #放弃合并

  git merge —-no-ff -m “描述信息” 分支名  #解决Fast forward删除分支会丢失commit id信息，加-m选项描述信息，因为会产生新commit id

  总结:合并分支时，加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。

5、解决bug分支前隐藏当前工作目录

  工作场景:当前某个分支还未提交，需要解决线上bug:
  git stash  #把当前工作现场"储藏"起来,然后就可以修改bug

  git stash  list  #查看工作现场

  git stash apply  #恢复工作现场，恢复以后stash内容不删除，使用git stash drop来删除

  git stash pop #恢复工作目录同时并把stash内容也删除

  git stash apply stash@{0} #恢复指定stash
 
6、git log --pretty=format:"%h - %an, %ar : %s"

   %H		提交对象（commit）的完整哈希字串
   %h		提交对象的简短哈希字串
   %T		树对象（tree）的完整哈希字串
   %t		树对象的简短哈希字串
   %P		父对象（parent）的完整哈希字串
   %p		父对象的简短哈希字串
   %an		作者（author）的名字
   %ae		作者的电子邮件地址
   %ad		作者修订日期（可以用 -date= 选项定制格式）
   %ar		作者修订日期，按多久以前的方式显示
   %cn		提交者(committer)的名字
   %ce		提交者的电子邮件地址
   %cd		提交日期
   %cr		提交日期，按多久以前的方式显示
   %s		提交说明

   -(n)	仅显示最近的 n 条提交
   --since, --after	仅显示指定时间之后的提交。
   --until, --before	仅显示指定时间之前的提交。
   --author	仅显示指定作者相关的提交。
   --committer	仅显示指定提交者相关的提交。

   git log --pretty="%h - %s" --author=gitster --since="2008-10-01" --before="2008-12-27" --no-merges -- t/
   git log  --pretty=format:"%cn committed %h on %cd"


7、分支作用

  分支策略

  在实际开发中，我们应该按照几个基本原则进行分支管理：

  首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

  那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

  你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了。

8、tags创建和推送

git tag <tagname>用于新建一个标签，默认为HEAD，也可以指定一个commit id；

git tag -a <tagname> -m "blablabla..."可以指定标签信息；

git tag可以查看所有标签。

git push origin <tagname>可以推送一个本地标签；

git push origin --tags可以推送全部未推送过的本地标签；

git tag -d <tagname>可以删除一个本地标签；

git push origin :refs/tags/<tagname>可以删除一个远程标签
```