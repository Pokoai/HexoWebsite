---
title: Python项目实战一：weiboSpider
date: 2022-08-09 22:45:53
toc: true
categories:
- Coding
- Python
tags:
- 爬虫
- requests
id: 40
---

> 准备借助几个爬虫项目来掌握 Python 的使用，重点在于代码的组织架构、项目的开发流程、优化过程以及Python 的高级用法：如装饰器、面向对象等较为常用的部分，不深究很少用到的知识点。因为时间宝贵，大部分精力要花在 C++ 上面，Python 只是拿来玩玩的。

项目仓库：[https://github.com/dataabc/weiboSpider](https://github.com/dataabc/weiboSpider)

## 介绍

本项目利用 Python 爬取指定用户的所有微博，主要输出为 csv 格式或者输出到数据库中。

我选择这个项目的原因有几个：

1. 作者最开始发布的是单文件，一步步重构为多文件结构，契合我的学习目的。我写的微信公众号爬虫目前也是单文件模式，想学习完该项目的重构思想后，着手重构我自己的项目，便于大家使用。
2. issue 详细记录了项目的发展脉络，便于我学习；
3. 作者最初这个项目没有采用框架，是自己利用 requests 等模块搭建起来的，比较原生态，适合学习代码思想；同样契合我的微信公众号项目目前的状态。

<!--more-->

**需要重点关注学习的：**

1. 代码组织框架。
2. 设置与代码逻辑部分相分离的思想。这样便于后期修改配置，让用户使用更容易，只要修改配置文件部分即可，不用动代码部分。我之前采用的是利用全局变量来配置的，修改起来麻烦，而且代码耦合度也高。
3. 增量更新的思想。我的微信爬虫项目的增量更新方法是自己想的，应用上太狭窄，但又没时间去调研其他的增量更新方案，这个项目正好也有涉及，后面要好好学习作者是如何实现增量更新的。（而且我知道他最初是全量更新的，与我的项目进程很相似）
4. json 数据结构的解析。我没有专门学过，这里掌握下，后面就一劳永逸了。
5. MySQL 数据库的应用。本来我后期也要单独学习数据库的，该项目涉及到了，就先用它学习下如何应用数据库，以后再学理论知识很更轻松。

## 源码解构






## 其他经验

### git commit 规范

我之前提交 commit，文字介绍部分是随意写的，看该项目的 commit log，发现作者记录很规范，我借鉴过来，后面我自己的 commit 也按这个规范写。

```
fix: 修复...问题
style: 修改代码格式
feat: 增加...功能
update: README.md
refactor: 重构...
perf: 优化...
docs: add ...
```

## 总结