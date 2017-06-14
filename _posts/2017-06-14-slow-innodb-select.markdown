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

这是一句比较复杂的sql查询语句，涉及到