---
title: 无法访问Github导致git pull失效的解决办法
date: 2022-08-09 02:48:20
toc: true
categories:
- Coding
- Git
tags:
id: 12
---

> 摘录自：[https://iymark.com/computer/github-linux-cannot-access.html](https://iymark.com/computer/github-linux-cannot-access.html)

报错内容如下：

```
git pull

fatal: 无法访问 'https://github.com/teddysun/lamp.git/'：Empty reply from server
```

然后我们 ping 一下 github.com，长时间无任何反馈：

<!--more-->

![](https://img.arctee.cn/one/202208090259308.png)

## 解决

### 修改系统的 hosts 文件

```
vi /etc/hosts

添加如下代码：
15.164.81.167 github.com
15.164.81.167 www.github.com
```

### 如何获取 Github可访问的 ip 地址

访问链接：https://ping.chinaz.com/github.com

ping 检测完成后，会发现所有国内服务器都是访问的 20.205.243.166 这个 ip

这时候，我们继续往下找其他地区测得的 ip 地址，比如：52.69.186.44

![](https://img.arctee.cn/one/202208090300011.png)

在 SSH 终端里 ping 一下该地址，如果可以访问，那么我们就可以用该 ip 来设置 github hosts 地址。

