---
title: 虚拟机如何安装VMware Tools？
date: 2022-08-09 03:13:10
categories:
- Coding
- Linux
tags:
- VMWare
id: 14
---

## 问题

用 VMware 安装完 Ubuntu Linux 系统后，重启进入到 Ubuntu 界面，这时发现整个窗口很小，没有铺满 VMware 的窗口，而且也不会跟随 VMware 窗口的缩放而变化。示图如下：

<!--more-->

![](https://img.arctee.cn/one/202205271150837.png)

## 分析

在 Ubuntu 界面上可以看见一个虚拟光驱：VMware Tools，实际上就是没有安装该辅助工具导致的。

如果没有看到 VMWare Tools，可以自己重新挂载到 Ubuntu 上，然后进行安装操作。

## 解决

### 挂载 VMWare Tools 虚拟光驱（可选项）

点击 VMware 的「虚拟机」菜单，找到 「安装VMware Tools」，点击安装即可。

因为我这里已经安装了，所以显示的「取消VMware Tools安装」。

![](https://img.arctee.cn/one/202205271157325.png)

### 安装 VMware Tools 工具

我这里采用的是命令行安装方式，因为我喜欢命令行的操作。如果是新手，平时接触 Linux 不多，也可以把这当作学习 Linux 命令行的实践。（关于如何学习 Linux 的操作，后续有时间我会出一篇详细新手教程，进度随大家反馈热度而定）

#### 一、进入 VMware Tools 挂载目录

如果是界面化操作，直接从桌面上双击就 OK 了。

![](https://img.arctee.cn/one/202205271210370.png)

但是采用命令行操作的话，需要先找打该虚拟光驱挂载在哪个文件目录下（挂载等名词目前不懂没关系，可直接忽略我的文字说明，直接根据截图操作）。

**1. 打开 Terminal，寻找挂载目录**
  
```bash
ls
# 当前目录下没发现相关文件
```

![](https://img.arctee.cn/one/202205271219707.png)

再去根目录下看看：

```bash
cd /

ls

# 发现有个 cdrom，进去看看
cd /cdrom 

ls
# 里面是空的，说明不对，继续寻找
```

![](https://img.arctee.cn/one/202205271223308.png)

一般来说，硬盘、光驱一般挂载在 /mnt 目录下，我们进入看下：

```bash
cd /mnt

ls
# 也是空的
```

![](https://img.arctee.cn/one/202205271228686.png)

**不要气馁，这就是试错学习的过程。遇到任何问题，首先要自己分析解决，一步步尝试，如果过了几十分钟仍未
解决再去百度。千万不要走来就百度，那就失去了宝贵的学习机会，而且是效果极好的学习实践。**

上面的路径都排除了，那么就只剩一个地方了，那就是 /media 目录：

```bash
# 进入 /media 
cd /media

ls

# 继续进入用户名命名的目录
cd csto

ls
## 终于找到了
```

![](https://img.arctee.cn/one/202205271234195.png)

**2. 进入虚拟光驱**

```bash
cd VMware\ Tools/

ls 
```
`VMwareTools-10.3.23-16594550.tar.gz` 这个文件就是我们后续需要安装的。

#### 二、解压 VMware Tools 压缩包

**1. 先解压**

```bash 
tar -xvzf VMwareTools-10.3.23-16594550.tar.gz

# 竟然出错了
```

![](https://img.arctee.cn/one/202205271241923.png)

解压失败原因：Read-only，即该文件权限为只读。为了方便，我们接下来就将该文件的权限改为 r+w+x (读+写+可执行，数字表示是 777)

```bash 
# 先看下当前权限
ll 

# 改变权限
chmod -R 777 VMwareTools-10.3.23-16594550.tar.gz

# 再看下改完的权限
ll 

```

改前权限：

![](https://img.arctee.cn/one/202205271248504.png)

改变权限：

![](https://img.arctee.cn/one/202205271249596.png)

改后权限：

![](https://img.arctee.cn/one/202205271250008.png)

**文件权限竟然没有变化，那是为什么呢？**

实际上这个问题很常见，也很重要，一般初学者容易卡在这里。往往只关注了文件本身，而忽略了文件所在的目录。

解压操作是需要将解压出的文件写入到该文件所在目录的，尽管压缩文件所有权限都打开，可是所在目录如果没有写入权限的话，依然无法执行解压操作。

为了验证这个原因，我们可以来试试，比如在该目录下创建一个文件：

```bash
touch file
```
看到只读提示信息，证明我们前面的问题分析是正确的。

![](https://img.arctee.cn/one/202205271302098.png)

该光驱是 VMware 虚拟出来的，仅仅存储了为 Linux 系统提供的辅助工具，权限只有只读，而不能写入。

那么好办，我们只要**将该文件拷贝到其他目录下（比如用户根目录）然后更改权限再解压即可**。

**2. 拷贝到 /home/csto**
  
```bash
cp VMwareTools-10.3.23-16594550.tar.gz /home/csto

tar -xzvf VMwareTools-10.3.23-16594550.tar.gz /home/csto
```

![](https://img.arctee.cn/one/202205271317504.png)

**3. 更改文件权限**

```bash
chmod -R 777 VMwareTools-10.3.23-16594550.tar.gz /home/csto
```

![](https://img.arctee.cn/one/202205271318429.png)

**4. 开始解压**

```bash
tar -xzvf VMwareTools-10.3.23-16594550.tar.gz /home/csto
```

![](https://img.arctee.cn/one/202205271320225.png)

解压完得到 vmware-tools-distrib 文件夹，安装文件就在这里面了。

#### 三、最后一步：正式安装

![](https://img.arctee.cn/one/202205271324941.png)

```bash
# 进入目录
cd vmware-tools-distrib

ls

# 开始安装
./vmware-install.pl
# 报错
```

![](https://img.arctee.cn/one/202205271328399.png)

提示需要用超级用户才能执行安装操作。

```bash
sudo ./vmware-install.pl
```

![](https://img.arctee.cn/one/202205271329167.png)

**安装过程中第一次询问一定要输入`yes` 然后再按回车，后续的所有询问直接按回车选择默认设置即可。**

![](https://img.arctee.cn/one/202205271332707.png)

#### 四、安装完成

![](https://img.arctee.cn/one/202205271337654.png)

此时，你会发现 Ubuntu 的界面已经自动适配 VMWare 了。

![](https://img.arctee.cn/one/202205271340335.png)

接着，重启 Ubuntu：

```bash
reboot
```

**问题解决，大功告成。**