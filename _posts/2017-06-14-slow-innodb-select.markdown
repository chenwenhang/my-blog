---
layout:     post
title:      "百万级数据调优小记"
subtitle:   "Optimize for million data"
date:       2017-06-14 11:07:00
author:     "Echo"
header-img: "img/post-bg-os-metro.jpg"
catalog: true
tags:
    - mysql
    - 服务器
---

> “Gain experience from mistakes”

## A Server interface is slow to respond

前几天一个项目的服务器要到期了，客户买了新服务器，需要我去把项目迁移到新服务器上，在迁移完毕之后，测试接口的时候
却发现其中一个接口的响应时间特别长，等待时间达到了一百多秒！项目的代码是完全相同的，数据库也是sql文件原封不动的导入
到新数据库的，按理说不应该出现这种情况。

项目详情：

* Windows Server 2008 企业版
* apache 2.4.9
* PHP 5.5.12
* Mysql 5.6.17

## The process of checking out errors

#### 版本 & apache模块

我第一个想到的是**版本问题**，经检查，服务器、数据库以及语言的版本都与原服务器一致。

第二个想到的是**原服务器上 apache 添加了一些模块**，原封不动复制过去，但问题依然没有改善。


#### 代码执行时间测试

无奈，只能一步步开始测试代码的执行时间，为接口文件添加执行时间函数

```PHP
function getmicrotime()
{
    list($usec, $sec) = explode(" ",microtime());
    return ((float)$usec + (float)$sec);
}
```

通过`$time_start = getmicrotime();`和`$time_end = getmicrotime();`记录时间点，然后相减得出执行时间，放入到结果集中返回前台。
对几个部分测试结果如下：

![Time check](/img/in-post/post-4-time.png "Time")

可以看到`$time4`的执行时间极长，已经达到了104秒。而这部分代码对应的是一句sql的执行，sql语句如下：

```sql
SELECT
  depend.deviceId,
  depend.group_id,
  DATA.tmp,
  DATA.ph,
  DATA.o2,
  DATA.feed_switch_status AS feedStatus,
  DATA.o2_switch_status AS o2Status,
  DATA.recv_time AS recvTime,
  info. status,
  count(alarm.id) AS alarmCount,
  info.imei
FROM
  pond
LEFT JOIN device_depend AS depend ON depend.pondId = pond.pondId
RIGHT JOIN device_info AS info ON info.id = depend.deviceId
LEFT JOIN (
  SELECT
    device_id,
    group_id,
    max(id) AS id
  FROM
    device_data
  GROUP BY
    device_id,group_id
) AS dataTime ON
  dataTime.device_id = depend.deviceId AND 
  dataTime.group_id = depend.group_id
LEFT JOIN device_data AS DATA ON 
  DATA.id = dataTime.id
LEFT JOIN device_alarm AS alarm ON
  alarm.device_id = depend.deviceId AND 
  alarm.group_id = depend.group_id AND 
  alarm.ack_time IS NULL AND
  msg <> '溶氧过高报警'
WHERE
  pond.pondId =:pondId
GROUP BY
  depend.deviceId
;
```

#### sql 语句测试

这是一句比较复杂的sql查询语句，涉及到五个表以及一个临时表。数据量是**三百一十多万条记录**。

第一反应是**数据表没加索引**以至于查询效率低下，然而检查数据库的时候发现是有索引的，和原服务器上一样。

然后我觉得可能是**索引存在但未被正常使用**，先直接查看sql语句执行状况：

![Sql state](/img/in-post/post-4-state.png "state")

发现`Handler_read_key`和`Handler_read_next`值都高，说明索引应当被正常使用了，参数具体含义参考
[浅谈MySQL之 Handler_read_*参数](http://gfsunny.blog.51cto.com/990565/1558480)

为了确认索引正常使用，通过`EXPLAIN`命令得到下图结果：

![Explain sql](/img/in-post/post-4-explain.png "Explain")

先看`key`字段，这个字段是指**使用到的索引**，这里每一行均有值，且运用到了`device_group`、`INDEX_DEVICE_ID`等索引，说明
索引是起作用的。

再看`key_len`字段，这个字段是指**使用的索引长度**，索引长度也恰当并不算长，不会耗费太多时间。

最后看`type`字段，这个字段是指**连接类型**，影响也较大，比较复杂，有兴趣的朋友可以去[zhuxineli的专栏](http://blog.csdn.net/zhuxineli/article/details/14455029)
学习一下，这里不再赘述。

综合以上，sql语句本身和数据库索引都是没有问题的。

#### 数据库配置

接着我开始怀疑是数据库配置问题，我认为可能是内存分配给mysql的太小了导致每次查询都会产生大量磁盘I/O，延缓查询速度。

于是我去更改了`my.ini`的配置。添加了`max_connections = 1024`增大最大连接数、`key_buffer_size=128M`增大索引块缓冲区大小，并添加`innodb_buffer_pool_size = 512M`将`Innodb`的内存池大小设置为512M。具体参数配置参考[my.ini优化mysql数据库性能的十个参数](http://www.jb51.net/article/72577.htm)

再次测试，发现确实有一定改善，sql执行时间从一百多秒降低到七十多秒，但仍然没有从根本上解决问题，效率依旧低下。

#### 数据库引擎

> “Innodb is not always suitable”

最后，在我[师父](https://wss534857356.github.io/)的指引下找到了错误，发现是数据库引擎的问题。

mysql广泛使用的两个数据库引擎`innodb`和`myisam`差异还是很大的。新版服务器上的mysql使用的是`Innodb`引擎，`Innodb`是**不支持全文索引的（修正：mysql5.6后开始支持）**，本身也**不会存储表中的数据行数**，以至于每次查询都需要对全表三百多万条数据进行扫描，极为耗费时间，具体差异可以参考[一夜长风的专栏](http://blog.csdn.net/wlzx120/article/details/53924123)。

简答点说，`Innodb`对**数据表的更新友好**，当数据库需要**频繁大量地增删改**时，推荐`Innodb`，`Myisam`对**查询友好**，如果执行**大量查询操作**，推荐使用`Myisam`。

更换数据库引擎为`Myisam`，问题解决，其实网络上也有一些优化`Innodb`的方法，但服务器第二天就要到期了必须马上完成迁移，也就没有尝试了，有兴趣的朋友可以去看看。


## Sleepy...

> “We are never afraid of the competitors who are as smart as god but the teammates who are as incapable as pigs”

以下是吐槽……

昨晚一直忙到将近两点钟，也没有解决问题，如果不是[师父](https://wss534857356.github.io/)救急那就真的gg了。服务器几天前就迁移好了，问了测试说有没有问题，都说一切正常，一直到交接的时候人家跟我说接口响应速度太慢，也是醉了，大兄弟测试的时候就不能长点心么。
好困啊T_T...




