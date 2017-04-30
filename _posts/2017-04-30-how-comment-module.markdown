---
layout:     post
title:      "为你的github博客添加评论模块"
date:       2017-04-30 22:48:00
author:     "Echo"
header-img: "img/post-bg-miui6.jpg"
tags:
    - 评论模块
    - 博客
    - github
---

> “Add comment module in your blog”

## Foreword

> The last blog in Apri.

既然已经建立好了博客，那么当然需要各位看客老爷指出博文中的错误与不足，但是邮件联系的做不到评论的**持久化和公开化**，并且
也不是很方便，为了及时收到大家的建议，我为博客添加了**畅言评论模块**。

其实[HUX老师](http://huangxuan.me/)在制作这篇博客主题的时候已经为其添加了[Disqus](http://www.disqus.com/)和[多说](http://dev.duoshuo.com/)的评论模块，二者都属于**第三方社会化评论系统**，但我在使用的时候却遇到了一系列尴尬的事情。


## Compare

> The merits and demerits between these modules.

### Disqus

自从使用 Jekyll 以来，一直使用[Disqus](http://www.disqus.com/)作为评论系统，作为社会化评论系统的鼻祖，
无论是用户量还是使用体验，Disqus 都是**一级棒**的。遗憾的是disqus在国外，想翻墙使用还是挺麻烦的。

优点:

* 功能全面，稳定，用户量大，用户体验好

缺点：

* 国外的，访问慢，甚至被墙，最近连官网都进不去

### 多说

[多说](http://dev.duoshuo.com/)作为第三方社会化评论系统，在国内用起来确实不错（特别是在Disqus被墙的情况下），
**自定义样式**更是能让你玩出很多花样。我起初也准备用多说的，但是进官网的时候看到通知多说**即将关闭**，便直接放弃了。

优点:

* 功能丰富，且允许自定义样式

缺点：

* 由于盈利模式模糊，多说团队最近几乎无人维护，bug和垃圾评论越来越多
* 多说将于6月1日正式关闭

### 友言

[友言](http://www.uyan.cc/)也是一款不错的评论系统，口碑也挺好，代码也简易，一般情况下使用没什么问题。但当
github个人博客遇到友言的时候，尴尬的事情发生了，**github page不知啥时候开始强制要求HTTPS协议访问**，而**友言
的js引入只能通过http**，搞了半天最后报了个`Fix content`错误，只能另辟蹊径。

优点：

* 配置代码简单

缺点：

* 不支持HTTPS

### 评论啦

评论啦也是在网上看到的一个评论系统，是不是在疑惑为啥之前段首关键字都变蓝而这段没有？呵呵，因为我**找不到官网（T_T）**。

优点：

* 尚不明确

缺点：

* 找不到官网

### 畅言

[畅言](http://changyan.kuaizhan.com/)是我的最后一根救命稻草，当然他也没有让我失望，配置好后感觉还不错，色调也挺喜庆的，
重要的是他**支持HTTPS**，**可以在guthub page上使用**。

优点：

* 使用简单
* 支持HTTPS
* 风格喜庆，看起来热闹

缺点：

* 需要ICP备案号，不备案只能用15天


## Install & configuration

> Here comes Changyan!

在此介绍下将畅言安装到`github+jekyll`的个人博客的方法：

进入[畅言官网](http://changyan.kuaizhan.com/)，点击注册，注册完毕会跳转至完善资料，其中最重要的一个步骤就是**填写ICP备案号**，不填写的话只能使用15天，填写完毕后点击进入后台。

在左侧边栏里找到**安装畅言**，在此介绍**通过代码安装**，**复制PC端安装代码**，会得到如下代码段：

```html
//changyan 代码

<div id="SOHUCS" sid="请将此处替换为配置SourceID的语句"></div>
<script charset="utf-8" type="text/javascript" src="https://changyan.sohu.com/upload/changyan.js" ></script>
<script type="text/javascript">
window.changyan.api.config({
appid: '你的appid',
conf: '你的conf'
});
</script>

```

如果只是想**对畅言单一支持**，引用以上代码至`_layouts`目录下的`post.html`的`<body></body>`标签中的合适位置即可（不同的主题可能路径略有差别）

如果是制作主题模板，即希望**对多种评论模块支持**的话，则需要修改`_config.xml`文件，在其中添加一行`changyan_use: `,然后将以上代码包裹在`if`标签中，
`{% if site.changyan_use %}`和`{% endif %};`形如：

```
//包括在if中

{% if site.changyan_use %}
<div id="SOHUCS" sid="请将此处替换为配置SourceID的语句"></div>
<script charset="utf-8" type="text/javascript" src="https://changyan.sohu.com/upload/changyan.js" ></script>
<script type="text/javascript">
	window.changyan.api.config({
		appid: '你的appid',
		conf: '你的conf'
	});
</script>
{% endif %};

```

这样，只要`_config.xml`文件中`changyan_use: `后**不为空**，即会调用该评论模块，不使用时置空即可。


## Afterword 

加个模块处处碰壁尴尬，很是尴尬，一些网友说不如自己搭建一个评论系统，听起来很厉害的样子，有时间可以捣鼓捣鼓~















