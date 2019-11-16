---
layout:     post
title:      "为你的github博客添加评论模块"
subtitle:   "Add comment module for your github block"
date:       2017-04-30 22:48:00
author:     "Echo"
header-img: "img/post-bg-miui6.jpg"
catalog: true
tags:
    - 小知识
---

> “Add comment module in your blog”

## Foreword

> The last blog in Apri.

既然已经建立好了博客，那么当然需要各位看客老爷指出博文中的错误与不足，但是邮件联系做不到评论的**持久化和公开化**，并且也不是很方便，为了及时收到大家的建议，我为博客添加了**畅言评论模块**。

其实[HUX老师](http://huangxuan.me/)在制作这篇博客主题的时候已经为其添加了[Disqus](http://www.disqus.com/)和[多说](http://dev.duoshuo.com/)的评论模块，二者都属于**第三方社会化评论系统**，但我在使用的时候却遇到了一系列尴尬的事情。


## Compare

> The merits and demerits between these modules.

### [ NEW ] Gitalk
***Updated on November 16, 2019***

[Gitalk](https://gitalk.github.io/)是一个基于 GitHub Issue 和 Preact 开发的评论插件。畅言广告太烦，昨天手机浏览博客的时候居然引导我打开支付宝抢红包...简直不能忍。这几年第三方评论模块基本倒的差不多了，也让Gitalk逐渐流行开来，目前该博客已经全站采用Gitalk。

优点：

* 支持多语言
* 无广告，极简，清爽无比
* 无干扰模式（设置 distractionFreeMode 为 true 开启）
* 快捷键提交评论 （cmd|ctrl + enter）

缺点：

* 只能用 Github 帐号登录，门槛较高

安装教程：[一款超好用的第三方评论插件--Gittalk](https://www.jianshu.com/p/4242bb065550)

### Disqus

[Disqus](http://www.disqus.com/)一直是 Jekyll 的完美搭配，作为社会化评论系统的鼻祖，
无论是用户量还是使用体验，Disqus 都是**一级棒**的。遗憾的是 Disqus 在国外，想翻墙使用还是挺麻烦的。

优点:

* 功能全面，稳定，用户量大，用户体验好

缺点：

* 国外的，访问慢，甚至被墙，最近连官网都进不去

### 多说

[多说](http://dev.duoshuo.com/)作为第三方社会化评论系统，在国内用起来确实不错（特别是在Disqus被墙的情况下），
**自定义样式**更是能让你玩出很多花样。我起初也准备用多说的，但是进官网的时候看到通知**多说即将关闭**，便直接放弃了。

优点:

* 功能丰富，且允许自定义样式

缺点：

* 由于盈利模式模糊，多说团队最近几乎无人维护，bug和垃圾评论越来越多
* 多说将于6月1日正式关闭

### 友言

[友言](http://www.uyan.cc/)也是一款不错的评论系统，口碑也挺好，代码也简易，一般情况下使用没什么问题。但当
github个人博客遇到友言的时候，尴尬的事情发生了，**Github page 不知啥时候开始强制要求HTTPS协议访问**，而**友言
的js引入只能通过 HTTP**，搞了半天最后报了个`Fix content`错误，只能另辟蹊径。

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
* 风格喜庆，看起来热闹，有评论欲望

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
形如：

```html
//包裹在if中
//使用时请将'#123;'替换为'{'，'#125;'替换为'}'，kramdown在此处的解析问题我还没想到合适的办法解决

#123; % if site.changyan_use % #125;
<div id="SOHUCS" sid="请将此处替换为配置SourceID的语句"></div>
<script charset="utf-8" type="text/javascript" src="https://changyan.sohu.com/upload/changyan.js" ></script>
<script type="text/javascript">
	window.changyan.api.config({
		appid: '你的appid',
		conf: '你的conf'
	});
</script>
#123; % endif %125; ;

```

这样，只要`_config.xml`文件中`changyan_use: `后**不为空**，即会调用该评论模块，不使用时置空即可。


## Afterword 

加个模块处处碰壁，很是尴尬，一些网友说不如自己搭建一个评论系统，听起来很厉害的样子，有时间可以捣鼓捣鼓~















