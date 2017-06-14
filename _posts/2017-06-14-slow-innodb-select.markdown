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

## The server interface is slow to respond

前几天一个项目的服务器要到期了，客户买了新服务器，需要我去把项目迁移到新服务器上，在迁移完毕之后，测试接口的时候
却发现其中一个接口的响应时间特别长，等待时间达到了九十多秒。项目的代码是完全相同的，数据库也是sql文件原封不动的导入
到新数据库的，按理说不应该出现这种情况。

项目详情：

* Windows Server 2008 企业版
* apache 2.4.9
* PHP 5.5.12
* Mysql 5.6.17

## “The process of checking out errors”
