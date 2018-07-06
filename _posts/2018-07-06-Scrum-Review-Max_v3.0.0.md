---
layout:     post
title:      "Scrum Review Max_v3.0.0"
subtitle:   " \"Scrum Review\""
date:       2018-07-06 13:00:00
author:     "Jeorch"
header-img: "img/post-bg-scrum.jpg"
catalog: true
tags:
    - Scrum
---

> “Yeah It's Scrum Review. ”


## 前言

对20180506～20180627(6周)1.5个月的Scrum周期进行review。

## 完成Max基于Spark的算法，并与线下进行对数

  - 使用表驱动法实现多市场多算法逻辑的映射，这一块之后可以考虑合并到老齐的builder里
  - 使用DataFrame和RDD实现Max算法不难，消耗时间的还是对数过程，出现的问题大多是匹配文件不规范或者线下更改了文件而线上未更新

## 完成Spark数据与MongoDB数据库同步

  - 启动Max后端项目时，执行一次ScheduleTimer，同步本地数据到mongo并同步mongo数据到本地，之后每24h同步一次数据
  - MongoDB服务器定期聚合job，使用linux周期指令crontab结合mongodb聚合脚本实现

## 完成Max数据查询，暂时以IndexRDD实现
  - 在RDD建索引上花了不少功夫，主要是优化搜索时间和搜索数据量上，之后使用elasticsearch替代

## 配置Redis AOF，实现历史数据恢复  
  - 当开启redis的Append-only-file后，如果不小心删除了某些重要的键或者不小心使用flushall清空了redis，可以找到appendonly.aof文件，然后删除文件中最近的一些命令(误删命令)，重启redis后，数据即可恢复

## 优化数据存储【Redis && MongoDB】
  - 能从一种缓存中load出数据，就不要多用另外一种缓存做多余的重复的工作
  - redis里应该存储的是单次job的一些kv，不会影响到下次任务的kv，并设置过期时间

## 完成Max数据导出和三个公司的交付接口
  - 利用运行时反射实现 满足不同公司的交付格式需求

## 完成Max运维中心  
  - 匹配表的展示与匹配文件的替换
  - 动态更新MongoDB中market_table的数据

*PS:在更新多重嵌套格式的mongodb数据时遇到了一些麻烦，还是JsValue和DBObject的互相转换，但是新的markt_table因为每个公司(或市场)的匹配文件不同，导致collection中不同document出现不同key，不能像以前一样在程序⾥按照指定的key进⾏解析，而是要动态的解析所有kv，Google爸爸告诉我，可以使⽤toString解决*
例如:
  1. j2d: DBOject.apply(Jsvalue.toString)
  2. d2j: Json.parse(dbo.toString)

## 测试与对接
  - Max算法测试与压力测试
  - 搜索接口对接
  - 运维中心对接

## 端午节与年中会议
  - 热忱表达（敢说）
  - 勇于思辨（敢想）
  - 让不可能成为现实（敢做）
  - 不止于

## 未完成的
  - 没有自己尝试kafka
  - 没有掌握macros
  - 没有掌握JsonAPI

## 反思
交给我的任务在周期内都已完成，但是自己却意识到项目进度的确前行缓慢！跨部门的协同工作是会影响到项目的进度，但是不能以此为借口，我慢了就是我慢了！年中会议表示下半年将开拓23个新的Max客户，Max做了也足一年了，项目上线or失败，自己要有危机意识！对项目负责、对团队负责、对公司负责。其实就是对自己负责！

## Keep Running
