---
layout:     post
title:      "Spark Dataframe 一些常规操作"
subtitle:   " \"Hello Spark, Hello Dataframe\""
date:       2018-07-02 16:00:00
author:     "Jeorch"
header-img: "img/post-bg-spark-dataframe.jpg"
catalog: true
tags:
    - Spark
    - Dataframe
    - 大数据
    - Scala
---

> “Yeah It's Spark. ”


## 前言

本文将直接介绍一些我在工作中处理DataFrame的常规操作，需要一点点Spark基础。
传送门：[Spark SQL, DataFrames and Datasets Guide](http://spark.apache.org/docs/latest/sql-programming-guide.html)

## DataFrame选择需要的列[select]
*select函数只需传入需要的column的name即可*

        df.select(col: String, cols: String*)

        eg:
        max_df.select("PHA_ID", "SALES")

## DataFrame选择需要的列[selectExpr]
*selectExpr函数是select的变种，除了select已有的功能，还可以解析sql表达式*

        df.selectExpr(exprs: String*)

        eg:
        df.selectExpr("colA", "colB as newName", "abs(colC)")
        df.select(expr("colA"), expr("colB as newName"), expr("abs(colC)"))

        max_df..selectExpr("concat(PRODUCT_NAME,APP2_COD,PACK_DES,PACK_NUMBER,CORP_NAME) as min1", "PHA_ID", "DOI as DOIE")

## DataFrame过滤操作[filter]
*filter函数可以解析sql表达式参数*

        df.filter(condition: Column)
        df.filter(conditionExpr: String)

*PS:使用filter操作多个条件时，filter仅满足了第一个条件，就不管第二个条件了，听起来像是短路与或者短路或，其实不然*
  - && 短路与　只要当前项为假，它就不往后判断了，直接认为表达式为假
  - || 短路或　只要当前项为真，它也不往后判断了，直接认为表达式为真

实战代码：
*过滤max数据中f_units和f_sales均不为零的数据集，使用下面的操作，却得到了f_units不等于零f_sales有等于零的错误数据集*

        max_df
            .withColumn("temp_tag", when($"f_units" =!= 0 and $"f_sales" =!= 0, 1).otherwise(0))
            .filter($"temp_tag" === 1)

*解决方法：利用withColumn和when函数结合，创建一个tag标签列来划分数据集*

        max_df.filter($"f_units" =!= 0 and $"f_sales" =!= 0)

*PS: where 等同于 filter*

## DataFrame聚合操作[groupBy]
*groupBy函数返回值为RelationalGroupedDataset，该函数常与agg函数搭配使用*

        df.groupBy(cols: Column*).agg(Map("salary" -> "avg", "age" -> "max"))

        eg:
        max_df
            .groupBy("Date", "Province", "City", "MARKET", "Product")
            .agg(Map("f_sales"->"sum", "f_units"->"sum", "Panel_ID"->"first"))

## DataFrame新建一列[withColumn]

        df.withColumn(newColName: String, col: Column)

        eg:
        max_df
            .withColumn("temp_tag", when($"f_units" =!= 0 and $"f_sales" =!= 0, 1).otherwise(0))


## DataFrame改列名[withColumnRenamed]

        df.withColumnRenamed(existingName: String, newName: String)

## DataFrame删除指定列[drop]

        df.drop(colNames: String*)        

## DataFrame连接操作[join]

        笛卡尔积：df1 join df2
        左连接：df1.join(df2: DataFrame, condition: Column, "left")
        右连接：df1.join(df2: DataFrame, condition: Column, "right")
        内连接：df1.join(df2: DataFrame, condition: Column, "inner")

        eg:
        max_df.join(city_df, max_df("city") === city_df("city"), "left")

## 一些常用的Column操作
*[when|otherwise|concat|substring|round]*

        when函数常和otherwise搭配使用:
        when(condition: Column, value: Any).otherwise(value: Any)

        concat可以把一些列的值连在一起：
        concat(exprs: Column*)

        substring可以取指定列值的定长子段：
        substring(str: Column, pos: Int, len: Int)

        round可以对列值规范数据精度：
        round(e: Column, scale: Int)

## 用户自定的UDF和UDAF
*将在另一篇博文中单独介绍*
