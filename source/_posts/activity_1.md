---
title: 乐活动“月跑一善”数据结构说明
date: 2018-08-31 15:52:04
tags: 乐活动
notshow: true
---

#### 一、逻辑说明
>  主要逻辑为用户可以通过运动换来能量，选择学校+书籍，判断用户的能量余额达到了某书籍的能量值，消耗能量，对该学校进行捐赠1册，当学校捐赠册数达到1000册，任务完成, 赞助方对该学校进行线下捐赠

>  次要逻辑，每个月有一个静态主题展示，每个月的某一天用户的能量获得翻倍，由后台创建

#### 二、数据表结构
1. 用户记录表: 记录用户的能量，运动等数据
2. 学校记录表: 学校是可以后台动态创建的，记录学校的相关信息
```
  dbMongoose.Schema({
      id:{type:String,index:true},
      img:String,
      name:String,
      sort:Number,
      cur:Boolean, //是否显示当前
      number:Number,
      target_number:Number,
      longitude:String, //经纬度
      latitude:String,
      state:Number, //0 未开始  1进行中  2已结束
      create_time:Number,
  });
```
3. 书籍记录表: 书籍也可以后台动态创建，记录书籍的被捐赠数据等
```
 dbMongoose.Schema({
    id:{type:String,index:true},
    name:String,
    description:String,
    img:String,
    count:Number,
    state:{type:Number, index:true}, //0 已结束 1上线状态 2隐藏状态
    sort:Number,
    number_target:Number,
    create_time:Number,
  })
```
4. 捐赠记录表: 包含了学校+书籍+数量等主要信息
```
dbMongoose.Schema({
    school:String,
    school_name:String,
    book:String,
    book_name:String,
    number:Number,
})
```
5. 证书记录表: 用户的捐赠记录
6. 跑团表: 记录官方跑团id，用户如果是该团列表里的，则能量加倍



