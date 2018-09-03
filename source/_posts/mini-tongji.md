---
title: 小程序 - 启动统计/来源统计
date: 2018-07-27 17:16:15
categories: 小程序
tags: 小程序
author: Jake
---

#### 一、场景
> 场景是给外部小程序导流量，发现我们出去的量(点击事件UV)跟对方拿到的量(来源统计UV)差异很大, 对方称在【app onShow】里上报的事件统计。 

> 经过对比数据，我们自己服务统计的用户跟微信统计的事件数据一致(点击事件)。

> 自测微信API上报事件来源统计:  APP生命周期上报【 app:onShow】上报统计量(uv)为A， 跟 页面的生命周期 (测试页面为首页)) 【page: onReady】 上报统计量(uv)定义为B,  如果按期预期A应该约等于B， 但是实测 B ~= 4*A， 数据相差4倍。

``` javascript
onShow (options) {
  let sourceQuery = options.query.custom_source
  if (sourceQuery && sourceQuery !== 'undefined') {
    // 微信自定义事件分析统计
    wx.reportAnalytics('track_source', {
      source: sourceQuery
    })
    // 阿拉丁事件统计
    let app = getApp()
    app.aldstat.sendEvent('普通来源-' + sourceQuery, '来源')
  }
}
```

#### 二、分析
> 分析：为何这么大差距?  onshow事件没有被触发？ 微信的api没上报成功？

> 在同样的两个位置加上了阿拉丁的事件分析:  结果还是 A(阿拉丁app onShow上报的事件统计)*3.5 = B(阿拉丁page onReady上报的事件统计)

> 发现线上错误日志：app:onshow :  app.aldstat.sendEvent(阿拉丁的统计方法) 错误,  怀疑app.aldstat在apponshow的时候还没初始化, 于是做了一个延迟上报处理

  ``` javascript
   setTimeout(() => {
      let app = getApp()
      app.aldstat.sendEvent('普通来源show-' + sourceQuery, '来源')
    }, 100)
  ```
> 然后神奇的事情发生了，同样的位置 A~=B。 然而微信api上报还是差4倍(没有加异步setTimeout)，证明微信自定义分析api上报这个方法在app.onshow事件中直接调用（wx.reportAnalytics）, 存在数据遗漏问题, 怀疑wx.reportAnalytics可能也存在初始化问题，或许可以也要setTimeout做个延迟上报操作。

#### 三、 解决方案
> 经过验证，困扰许久的问题，解决了。另外阿拉丁数据延迟俩小时
> 延迟1秒上报数据为确保数据更加准确

``` javascript
onShow (options) {
  let sourceQuery = options.query.custom_source
  if (sourceQuery) {
    setTimeout(() => {
      // 微信自定义事件分析统计
      wx.reportAnalytics('track_source', {
        source: sourceQuery
      })
    }, 1000)
    setTimeout(() => {
      // 阿拉丁统计
      let app = getApp()
      app.aldstat.sendEvent('普通来源-' + sourceQuery, '来源')
    }, 1000)
  }
}
```

