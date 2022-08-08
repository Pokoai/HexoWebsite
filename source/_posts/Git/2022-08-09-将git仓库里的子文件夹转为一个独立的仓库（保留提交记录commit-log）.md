---
title: 将git仓库里的子文件夹转为一个独立的仓库（保留提交记录commit log）
date: 2022-08-09 02:46:21
categories:
- Coding
- Git
tags:
- git
id: 11
---

> 转载：[https://blog.csdn.net/festone000/article/details/97947157](https://blog.csdn.net/festone000/article/details/97947157)

这两天在一个大项目里写了一些小功能，后来觉得这部分内容不必整合到大项目中去，于是就产生了这个不太常见的需求，一查居然还真可以。git 这回让我惊喜。

## 需求

1. 在一个大仓库里，如 bigProject.git 写东西，比如新建了一个A文件夹，里面是a项目，写了一大堆，但恰好不与此文件夹之外的相关。
2. 想把A文件夹里的内容转移出去成为一个独立的仓库，但又不想失去所有的提交记录(commit log)
3. 是否可行？

<!--more-->

---

## 答案： 可行

方法：
找到一个别人的做法，我按照方法1顺利地成功了。

链接如下：
[大笨重的git仓库拆分成灵活轻巧的模块小仓库](https://www.cnblogs.com/noxy/p/7192238.html)

关键部分摘录如下，以防原作者删除博文。

```git
# 这就是那个大仓库 big-project
$ git clone git@github.com:tom/big-project.git
$ cd big-project

# 把所有 `codes-eiyo` 目录下的相关提交整理为一个新的分支 eiyo
$ git subtree split -P codes-eiyo -b eiyo

# 另建一个新目录并初始化为 git 仓库
$ mkdir ../eiyo
$ cd ../eiyo
$ git init

# 拉取旧仓库的 eiyo 分支到当前的 master 分支
$ git pull ../big-project eiyo
```

