---
layout:     post
title:      "百万级数据调优小记"
date:       2017-06-14 11:07:00
author:     "Echo"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - mysql
    - 服务器
---

> “Innodb is not always suitable”

## A Server interface is slow to respond

前几天一个项目的服务器要到期了，客户买了新服务器，需要我去把项目迁移到新服务器上，在迁移完毕之后，测试接口的时候
却发现其中一个接口的响应时间特别长，等待时间达到了九十多秒。项目的代码是完全相同的，数据库也是sql文件原封不动的导入
到新数据库的，按理说不应该出现这种情况。

项目详情：

* Windows Server 2008 企业版
* apache 2.4.9
* PHP 5.5.12
* Mysql 5.6.17

## The process of checking out errors

#### 版本 & apache模块

我第一个想到的是**版本问题**，经检查，服务器、数据库以及语言的版本都与原服务器一致。

第二个想到的是**原服务器上 apache 添加了一些模块**，原封不动复制过去问题依然没有改善。


#### 代码断点测试

无奈，只能一步步开始测试代码的执行时间，为接口文件添加执行时间函数

```php
function getmicrotime()
{
    list($usec, $sec) = explode(" ",microtime());
    return ((float)$usec + (float)$sec);
}
```