# gorm的使用

## 前言

  在我们的日常开发中，会经常使用到关系型数据库，比如mysql,sqlite等，会涉及到对相应的数据模型进行建表，然后封装CRUD操作，
  使用golang中基础包的数据操作往往会让我们觉得繁琐，因为要写的代码比较多，因此诞生了gorm。
  

## 功能方法介绍

   方法名称 | 作用
   -------|------
  gorm.Open| 打开数据库。。
  db.SigularTable(true)| struct结构体映射采用结构名称作为表明（单数）
  db.LogModel（true)|开启gorm日志，将相应翻译SQL打印出来
  db.AutoMigrate(&table1,...)|初始化表


[官方地址](https://gorm.io/docs/models.html)
