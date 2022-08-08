---
title: git rebase合并多个commit
date: 2022-08-09 02:39:58
categories:
- Coding
tags:
- git
id: 9
---

> 全文摘录自：[www.its404.com](https://www.its404.com/article/f786587718/79444844)

不论是在使用 Git 或者 SVN 的过程中，我们都难免遇到出现多次的 commit。而导致在 Chery Pick 到其它分支的时候遗忘掉，或者是带来了其他不必要的麻烦。那么，有办法解决这些多次类似的 commit 提交记录么？

答案当然是有的。在 Git 中可以将多个提交记录合并成一个记录。只需使用 `git rebase -i SHA-1`前6位(需要合并到该记录的前一个hash值)或者是 Head~ 合并的总条数。

例如：总共有3次相同功能模块的提交记录(例子全为手动，懒癌发作)。

**1.**

commit 8d384922678....

Author: admin <admin@admin.com>

Date: Mon Mar 5 11:27:32 2018 +0800

Hello功能完善

**2.**

commit 98d2b46f44e6....

Author: admin <admin@admin.com>

Date: Mon Mar 5 11:25:32 2018 +0800

Hello功能 bug修复

**3.**

commit a12wxs1268ee....

Author: admin <admin@admin.com>

Date: Mon Mar 5 11:23:32 2018 +0800

新增Hello功能

**4.**

commit b7e419e2e1f8d....

Author: admin <admin@admin.com>

Date: Mon Mar 5 11:21:32 2018 +0800

\*\*\* bug修复（这个提交记录是跟前面的 Hello功能记录不相关的）

其中1,2,3为相同功能的提交记录，所以想将这三条记录合并成一个记录即 使用命令

git rebase -i b7e419（这里需要填写合并到该记录的前一条SHA-1值）

之后便出现如下界面(下面所列出来的顺序跟git log上是相反的。最上面的是最早提交的)：

pick a12wxs1 新增Hello功能

pick 98d2b46 Hello功能 bug修复

pick 8d38492 Hello功能完善

还有一些带#加一大串的是，注释所用。在这里面，我们只需用到 squash 和 pick。这两个的意思分别是：

squash，提交记录，合并到前一个记录。pick，提交记录。

所以我们想将 98d2b46和8d38492都合并到a12wxs1中，只需将 98d2b46和8d38492 前面的pick 改成 squash或者s即刻。

之后，便可以按Esc键退出，并敲上:wq保存退出。等待几秒，进入下一个界面。

\# This is a Combination of 2 Commits

\# The first commit's message is:

新增Hello功能-

\# This is the 2nd commit message:

Hello功能 bug修复

\# This is the 3rd commit message:

Hello功能完善

这是要合并的三条记录，你可以保留最上面到 新增Hello功能.将剩余的4行提示信息删除。这样就只保留了一条信息。

同上面，按Esc，并输入:wq 回车即可。这时候就开始合并。如果在合并过程中出现类似问题：

git rebase -i resumeerror: could not apply 8d38492 ... Hello功能完善 When you have resolved this problem, run "git rebase --continue".If you prefer to skip this patch, run "git rebase --skip" instead.To check out the original branch and stop rebasing, run "git rebase --abort".Could not apply 8d38492 ... Hello功能完善。

使用git status查看当前哪几个文件冲突。然后，解决冲突。再次git add file（文件路径，包含冲突文件和已合并文件）,都add完成后，在git commit 即可。 之后便可以使用git rebase --continue继续合并。直到合并成功。合并成功后push到远端即可。

**这里要注意的是：**

当在1.2.3这三条记录中，插入了其他的提交记录。这时候，你需要将它们的位置调整下。再进行上面的步骤。例如：在1和2之间插入了一条其他的提交记录

**1.**

commit 8d384922678....

Author: admin <admin@admin.com>

Date: Mon Mar 5 11:27:32 2018 +0800

Hello功能完善

**其他.**

commit a25w2za6ge2....

Author: admin <admin@admin.com>

Date: Mon Mar 5 11:26:32 2018 +0800

新增\*\*\*功能

**2.**

commit 98d2b46f44e6....

Author: admin <admin@admin.com>

Date: Mon Mar 5 11:25:32 2018 +0800

Hello功能 bug修复

在使用git rebase -i b7e419后，如下界面：

pick a12wxs1 新增Hello功能

pick 98d2b46 Hello功能 bug修复

pick a25w2za 新增\*\*\*功能

pick 8d38492 Hello功能完善

再进行合并的时候，只需将 a25w2za 和 8d38492 位置互调。如下即可。

pick a12wxs1 新增Hello功能

s 98d2b46 Hello功能 bug修复

s 8d38492 Hello功能完善

pick a25w2za 新增\*\*\*功能-

其他步骤如上面所述。至此，合并大功告成。

O(∩\_∩)O哈哈~

注意：

上面说到的push，要强制push。不然会出现多个重复的提交记录。因为我是在主分支上进行操作的。所以需要强制push。