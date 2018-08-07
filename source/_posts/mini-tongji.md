---
title: 小程序 - 启动统计/来源统计
date: 2018-07-27 17:16:15
---
#### 1. 场景
- 场景是我们给外部小程序导流量，发现我们出去的量跟对方拿到的量差异很大, 对方称在【app onshow】里上报的事件统计。 

- 我们自己的事件统计没有问题，跟我们后台带userid的用户人数对的上。

- 测试使用微信API上报事件来源统计:  如果query 里面有关键字(custom_source的值)：【 app:onShow】上报统计量(uv)为A， 跟 page/index/index 【onReady】 上报统计量(uv)定义为B,  结果是A*4 ~= B 

#### 2. 分析
- 分析：为何这么大差距?  onshow事件没有被触发？ api没上报成功？

- 在同样的两个位置加上了阿拉丁的事件分析:  结果还是 A*3.5 = B

- 后来发现线上错误日志：app.onshow :  app.aldstat.sendEvent(阿拉丁的统计方法) 错误,  怀疑app.aldstat在apponshow的时候还没初始化, 于是
  ``` 
   setTimeout(() => {
      let app = getApp()
      app.aldstat.sendEvent('普通来源show-' + sourceQuery, '来源')
    }, 1)
  ```
- 然后神奇的事情发生了，同样的位置 A~=B。 然而微信api上报还是差4倍(没有加异步setTimeout)，证明微信自定义分析api上报这个方法在app.onshow事件中直接调用（wx.reportAnalytics）, 存在数据遗漏问题, 怀疑wx.reportAnalytics可能也存在初始化问题，或许可以也要setTimeout做个延迟上报操作。

#### 3. 解决方案
- 经过验证，困扰许久的问题，解决了。另外阿拉丁数据延迟俩小时。

```
onShow (options) {
  let sourceQuery = options.query.custom_source
  if (sourceQuery && sourceQuery !== 'undefined') {
    setTimeout(() => {
      // 微信后台统计
      wx.reportAnalytics('track_source', {
        source: sourceQuery
      })
    }, 2000)
    setTimeout(() => {
      // 阿拉丁统计
      let app = getApp()
      app.aldstat.sendEvent('普通来源-' + sourceQuery, '来源')
    }, 50)
  }
}
```

