---
title: Hexo主题CyanStyle优化
date: 2022-08-10 00:02:22
toc: true
categories:
- Coding
- Geek
tags:
- Hexo
- CyanStyle
id: 41
---

> 因为网上没有找到 Hexo 对应于 CyanStyle 主题的魔改教程，所以记录下我的优化过程，供后来者借鉴。

## 一、增加评论功能

实现效果图：

![](https://img.arctee.cn/one/202208131112951.png)


我用的是 [waline](https://waline.js.org/) 评论组件，后端部署在 [vercel](https://vercel.com) 上面，具体教程网上很多，官方的教程[看这里](https://waline.js.org/guide/get-started.html#vercel-%E9%83%A8%E7%BD%B2-%E6%9C%8D%E5%8A%A1%E7%AB%AF)。

<!--more-->

同时配置了[邮件通知](https://waline.js.org/guide/server/notification.html)功能，在 vercel 中设置几个变量即可，可以申请个 163 邮箱来发送通知邮件，建议不要使用自己的主邮箱。

后端部署很容易，重点其实在前端的部署上面。我对前端也不懂，在网上看了一些教程，但是找不到完全针对 CyanStyle 主题的配置，大多数都是针对 NexT 这种烂大街主题的。通过一番摸索后，最终成功配置好评论功能，其实也很简单。

### 修改 comment.ejs 文件

找到 comment.ejs 文件，路径：`.\themes\cyanstyle\layout\_partial\comment.ejs`

如果是 html 文件的话，直接把 waline 提供的代码粘贴上去就行了：

官方代码：

```html

<head>
  <!-- ... -->
  <script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script>
  <link
    rel="stylesheet"
    href="https://unpkg.com/@waline/client@v2/dist/waline.css"
  />
  <!-- ... -->
</head>
<body>
  <!-- ... -->
  <div id="waline"></div>
  <script>
    Waline.init({
      el: '#waline',
      serverURL: 'https://your-domain.vercel.app',
    });
  </script>
</body>

```

但是我们这里是 ejs 文件，不能全部照抄，摘取部分复制粘贴即可。

因为我已经修改过了，最原始的代码不知道是什么样子的，但是你只要配置成我这样的即可：

```html

<% if (!index && post.comments && theme.waline_comment){ %>
</br>
</br>
</br>
</br>  <!-- nav与comment保持四行间距 -->
<section id="comments">
  <div id="waline"></div>
    <script src="//unpkg.com/@waline/client@v2/dist/waline.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/@waline/client@v2/dist/waline.css"/>

  <script>
      Waline.init({
        el: '#waline',
        serverURL: '<%= theme.waline.serverURL %>',
        
        placeholder: '<%= theme.waline.placeholder %>',
        requiredFields: '<%= theme.waline.requiredFields %>',
        pageSize: '<%= theme.waline.pageSize %>',
        avatar: '<%= theme.waline.avatar %>',
        copyright: false,
        
      });
  </script>
</section>
<% } %>

```

- 将代码复制到 `comment.ejs` 后，会发现文章下方的 Pre-Next 翻页标签紧挨着评论框，所以添加了四个 `</br>`，加大间距，使整体布局比较美观。
- `if (!index && post.comments && theme.waline_comment)` 里原来是 `theme.comment`，我改为了`theme.waline_comment`，所以需要修改主题配置文件 `_config.cyanstyle.yml`。
- `Waline.init()` 里必须包含 `el` 和 `serverURL` 两项，其他非必须项可以不填。如果填了，那么在 `_config.cyanstyle.yml` 文件里也必须配置。
- `copyright` 也打算在 `_config.cyanstyle.yml` 文件中进行配置的，但是不知什么原因，无法生效，所以直接在 `Waline.init()` 中进行赋值：`false`。设置为 false，就不会在评论框下方显示 `power by waline` ，同样是为了美观。如果你不在意美观度，建议默认 true 即可，支持下 waline。

### 修改主题配置文件

1. 以防升级主题或者更换新主题时导致配置文件被覆盖掉，建议将 theme 文件夹下的 `_config.yml` 复制到项目文件夹下，文件名改为 `_config.cyanstyle.yml`，与网站配置文件 `_config.yml` 同级。

2. 添加以下内容：

```yml
# Miscellaneous

## Waline [Waline](https://waline.js.org) 评论
waline_comment: true
## el 和 path 会在页面自动生成，不必加入
waline:
  serverURL: https://comment.pokeai.cn/ # Waline 的服务端地址（必填）
  meta: [nick,mail,link] # waline comment header info
  avatar: robohash # Gravatar头像展示方式。默认值：'mp'【不必填】
  requiredFields: [nick,mail]  # 设置评论者属性必填项。默认值：[]（即匿名）【不必填】
  placeholder: 快来评论吧~  # 评论占位提示
  pageSize: 10 # 评论列表分页，每页条数。默认值：10【不必填】
  copyright: false # 是否显示页脚版权信息。默认值true【不必填】
  
  visitor: # 文章访问量统计。默认值: false【不必填】
  login: # 登录模式状态，默认值：enable（'enable': 启用登录、'disable': 禁用登录，用户只能填写信息评论、'force': 强制登录，用户必须注册并登录才可发布评论）【不必填】
  wordLimit: # 评论字数限制，填入单个数字时为最大字数限制。默认值：0（即不限制）【不必填】
  
  avatarCDN: # 设置Gravatar头像CDN地址。默认值: https://sdn.geekzu.org/avatar/【不必填】
  avatarForce: # 每次访问是否强制拉取最新的评论列表头像。默认值：false【不必填】
  highlight: # 代码高亮显示。默认值：true【不必填】
  mathTagSupport: # 是否注入核外样式以兼容 <math> 显示，默认值：false【不必填】
  emojiCDN: # 设置表情包 CDN。默认值：https://img.t.sinajs.cn/t4/appstyle/expression/ext/normal/【不必填】--即将废弃


```
 
其中一些设置无法生效，比如 placeholder，原因未知。


## 二、修复音乐播放插件

实现效果图：

![](https://img.arctee.cn/one/202208130444666.png)

原播放插件其实也是可以的，但有几个局限性：

- 需要在项目文件夹下存储歌曲源文件，占用内存;
- 只能播放一首歌曲，无法播放歌曲列表；

如果不在乎这些局限性，可以直接使用主题集成的播放插件即可，另外完成如下配置：

1. 存储歌曲源文件

在 `source` 文件夹下新建文件夹 `asset`，即 `source\asset`，然后在其中放入歌曲即可，编译后 public 文件夹下就会生成对应的 asset 文件夹，歌曲也在其中，这样就可以在 `_config.cyanstyle.yml` 的 `music:` 下指定歌曲路径了。

2. 修改主题配置文件`_config.cyanstyle.yml`

```yml
music: ./asset/123.mp3  # 假设歌曲文件名为 123.MP3

```

如果你和我一样，**想要用网易云音乐的歌曲源，那么就看下面的步骤：**

### 修改music插件代码

插件路径：`themes\cyanstyle\layout\_widget\music.ejs`

为了不破化原有的音乐插件，我没有修改其代码，而是新建了一个文件 `music_new.ejs`，代码如下：

```html

<!--去[网易云音乐](https://music.163.com/)搜索喜欢的音乐，点击生成外链播放器， 复制代码直接放到博文末尾即可，
height设为0可隐藏播放器，但仍然可以播放音乐，auto设成0可手动播放，默认是1自动播放。-->

<% if (theme.music && theme.music.enable){ %>
  <aside class="widget">
    <h3 class="widget-title">Music</h3>
    <div class="widget-content">
      <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=100% height=52 
      src="//music.163.com/outchain/player?type=0&id=<%= theme.music.id %>&auto=<%= theme.music.autoplay?1:0 %>&height=32"></iframe>
    </div>
  </aside>
<% } %>

```

- 其中 `src="<%= theme.music %>">` 需要在主题配置文件中指定播放源

### 修改`_config.cyanstyle.yml`

```yml

# Sidebar
widgets:
- music_new

## 网易云音乐插件
music: 
  enable: true
  autoplay: true
  id: 2818034763  

```

- 如果要更换播放源，直接更改 `id` 即可；`autoplay: true`：自动播放。

### 遗留问题

翻页会重置播放器的状态，无法不间断播放音乐。

网上介绍用 pjax 解决，我目前没有尝试。


## 三、增加返回文章顶部按键

实现效果图：

![](https://img.arctee.cn/one/202208130443069.png)

原主题没有返回文章顶部的功能，阅读到文章下方时想要回到前面只能拖动滚动条，很麻烦，所以我加上了该功能。

找到 `footer.ejs` 文件添加以下代码即可:

路径：`themes\cyanstyle\layout\_partial\footer.ejs`

```html

<script src="../../source/js/top.js" type="text/javascript"></script>
<!--返回顶部开始-->
<div id="full" style="width:0px; height:0px; position:fixed; right:8%; bottom:100px; z-index:100; text-align:center; background-color:transparent; cursor:pointer;">
  <a href="#" onclick="goTop();return false;">
    <img src="/css/images/top.png" border=0 alt="返回顶部">
  </a>
</div>
<!--返回顶部结束-->

```

- 按键图片可以去[阿里图标库](https://www.iconfont.cn/)找一个，尺寸为 48*48，然后放在 `/css/images`路径下，图片名修改为 `top.png`。

### 遗留问题

返回顶部图标一直展示在页面右下角，无法自动隐藏。

## 四、网站底部增加统计功能

实现效果图：

![](https://img.arctee.cn/one/202208130447883.png)

其中有两部分：

1. 网站运行时间统计
2. 网站访问量统计

步骤与第3点增加返回文章顶部按键类似，只要修改 `footer.ejs` 文件。

避免与上述重复，这里我直接贴上 `footer.ejs` 最终的代码，你只要复制粘贴过去覆盖原代码即可。 

```html

<footer id="colophon" role="contentinfo">
    <p>&copy; <%= date(new Date(), 'YYYY') %> <%= config.author || config.title %>
    All rights reserved.</p>
    <p>Powered by <a href="http://hexo.io/" target="_blank">Hexo</a></p>
</footer>

<script src="../../source/js/top.js" type="text/javascript"></script>
<!--返回顶部开始-->
<div id="full" style="width:0px; height:0px; position:fixed; right:8%; bottom:100px; z-index:100; text-align:center; background-color:transparent; cursor:pointer;">
  <a href="#" onclick="goTop();return false;">
    <img src="/css/images/top.png" border=0 alt="返回顶部">
  </a>
</div>
<!--返回顶部结束-->


<!--底部添加运行时间-->
<div class="run_time" style=" text-align:center;">
    <span id="timeDate">载入天数...</span><span id="times">载入时分秒...</span>
    <script>
      var now = new Date(); 
      function createtime() { 
          var grt= new Date("08/06/2022 00:00:00");  // 此处修改为你的建站时间或者网站上线时间 
          now.setTime(now.getTime()+250); 
          days = (now - grt ) / 1000 / 60 / 60 / 24; dnum = Math.floor(days); 
          hours = (now - grt ) / 1000 / 60 / 60 - (24 * dnum); hnum = Math.floor(hours); 
          if(String(hnum).length ==1 ){hnum = "0" + hnum;} minutes = (now - grt ) / 1000 /60 - (24 * 60 * dnum) - (60 * hnum); 
          mnum = Math.floor(minutes); if(String(mnum).length ==1 ){mnum = "0" + mnum;} 
          seconds = (now - grt ) / 1000 - (24 * 60 * 60 * dnum) - (60 * 60 * hnum) - (60 * mnum); 
          snum = Math.round(seconds); if(String(snum).length ==1 ){snum = "0" + snum;} 
          document.getElementById("timeDate").innerHTML = "本站已安全运行 "+dnum+" 天 "; 
          document.getElementById("times").innerHTML = hnum + " 小时 " + mnum + " 分 " + snum + " 秒"; 
      } 
      setInterval("createtime()",250);
    </script>
    </br>
    </br>
    <!--添加不蒜子统计-->
    <script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"> </script>
    <span class="post-meta-item-icon">
        <i class="fa fa-user"></i>
    </span>
    <span id="busuanzi_container_site_uv">
        🤵访问人数：<span id="busuanzi_value_site_uv"></span>人
    </span>
    <span class="post-meta-divider">|</span>
    <span class="post-meta-item-icon">
        <i class="fa fa-eye"></i>
    </span>
    <span id="busuanzi_container_site_pv">
        👀访问量：<span id="busuanzi_value_site_pv"></span>次
    </span>
    </br>
    </br>
  </div>

```

- 其中 `<i class="fa fa-..."></i>` 部分可以删除，因为该主题无法使用 Font Awesome，如何去配置我就懒得花时间了，所以直接复制 🤵 👀 这两个图标过去了。


## 五、修改文章底部翻页标签

实现效果图：

![](https://img.arctee.cn/one/202208100141940.png)

可能你也会遇到这两个部分显示不正常，我没有去深究原因，直接改为字符图标即可。（如果你知道什么原因，欢迎在文章底部留言。）

具体修改这个文件：`themes\cyanstyle\layout\_partial\article.ejs`

我当时也不知道该修改哪个文件，就搜索 `prev` 关键词，然后在 `themes\cyanstyle\layout\_partial` 下遍历文件，看哪个文件里面有该关键词，就找到了 `archive.ejs` 文件

修改如下：
![](https://img.arctee.cn/one/202208100145089.png)

局部代码：

```html

<% if (page.total > 1){ %>
    <nav id="pagination">
      <nav id="page-nav">
        <%- paginator({
          prev_text: '« ' + theme.prev,
          next_text: theme.next + ' »'
        }) %>
      </nav>
    </nav>
  <% } %>

```

## 六、文章目录导航栏

实现效果图：

![](https://img.arctee.cn/one/202208130544136.png)

> 主要摘自下面链接1的文章，增加了`list_number`参数，让目录不显示编号。

### 查阅 hexo 文档

在 Hexo 官网 `文档>自定义>辅助函数>` 最下面，可以找到`toc`这个函数，看其介绍能知道它就是来实现文章目录的。

![](https://img.arctee.cn/one/202208130419250.png)

### 修改文章模板

首先编辑文章显示页面的模板，也就是`themes\cyanstyle\layout\_partial\article.ejs` 文件。为了将目录生成在正文之前，我们首先在这个文件中找到 `<%- post.content %>`，并在这一行之前加入如下代码：

```html

<!-- Table of Contents -->
<!--目录-侧边栏-->
<% if (!index && post.toc && theme.toc.right){ %>
  <div id="toc-right" class="toc2-article" style="position:fixed; right:1%; top:50px; background-color: white;">
    <strong class="toc-title">目录</strong>
    <%- toc(post.content, {list_number: false}) %>
  </div>
<% } %>
<!--目录-文章开头-->
<% if (!index && post.toc && theme.toc.top){ %>
  <div id="toc" class="toc-article" >
    <strong class="toc-title">文章目录</strong>
    <%- toc(post.content, {list_number: false}) %>
  </div>
<% } %>
</br>   

```

- 设置了两个目录栏，分别位于文章开头和侧边栏，可以在主题配置文件中开启或关闭任意一个。

这段代码的含义清晰明了，if 语句中有两个条件，`!index` 是为了不在首页的文章摘要中生成目录，`post.toc` 确保了只在显式地标记了 `toc: true` 的文章中生成目录，`theme.toc.top` 确定是否开启目录。若这三个条件满足，则创建一个目录的 <div>。

修改完这个文件之后，找一篇包含了多个子标题的文章，并在文章开头的 `front-matter`中添加一句 `toc: true`，在浏览器中访问这篇文章，应该可以看到文章的开头处已经有了带链接的目录。

### 修改主题配置文件

```yaml

# toc
toc:
  top: true
  right: true

```

### 编写CSS文件美化目录

我觉得原始样式看起来也还可以，就没有去美化了。如果你有美化需求可以看下面两篇文章：

参考链接：

- [https://xyzardq.github.io/2016/11/04/Hexo%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E7%9B%AE%E5%BD%95/](https://xyzardq.github.io/2016/11/04/Hexo%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E7%9B%AE%E5%BD%95/)
- [https://cloud.tencent.com/developer/news/264178](https://cloud.tencent.com/developer/news/264178)

### 遗留问题

这里设置的目录位于文章开头，不便于阅读中途查看，最好是放在侧边栏。

但是侧边栏小组件不会随文章下翻而滚动，查看也不方便。

所以有以下几个解决方案：

1. 将目录放在侧边栏的最后一个组件里，并且该组件可以随文章滚动。
~~2. 因为文章页面最右侧仍有空间，可以将目录放在最右侧空白处的顶端，那么就不会与侧边栏的位置冲突了，而且也不用考虑滚动问题，一直显示即可，就像返回顶部图标一样。~~

~~![](https://img.arctee.cn/one/202208130441652.png)~~

