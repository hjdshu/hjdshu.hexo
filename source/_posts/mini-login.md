---
title: 小程序一般登录流程
date: 2018-08-23 09:34:35
tags: 小程序
---

### 一、小程序原生能力
微信利用了自己的账号系统，给每个小程序主体提供了拿到用户的唯一识别码: open_id
``` javascript
  wx.login({
    success: function (res) {
      if (res.code) {
        callback(null, res.code)
        // 拿到微信给的临时登录凭证code, 用这个code可以去服务器换成open_id
      } else {
        callback('登陆失败!' + res.errMsg)
      }
    }
  })
```
code换open_id必须通过服务器处理，需要使用到app_secret, 文档地址：
https://developers.weixin.qq.com/miniprogram/dev/api/signature.html

### 二、一般登录流程
1. 服务器拿到code之后，去服务器换成open_id和token，然后会给一个token给前端, 前端通常会将此token缓存在storage里
2. 登录完成之后执行待完成的需要登录操作的函数

``` javascript
let login = () => {
  wx.request({
    url: 'login',
    code: code,
    success: (data) => {
      let token = data.token
      wx.setStorageSync('token', token)
      // 执行队列里的等待函数
      store.state.ajaxList.forEach((item) => {
        item()
      })
    }
  })
}
```
3. 由于小程序里不支持cookie，所以通常前端在请求头里带上token

``` javascript
      wx.request({
        url: requestUrl,
        data: data,
        method: method,
        header: {
          'Authorization': `Bearer ${token}`
        }
      })

```

### 三、登录状态保持，重试
逻辑通常是当我们认为登录过期时，执行登录操作，并将待完成的操作加入队列

``` javascript
let Rq = function (param, method, url, callback) {
  let token = getStorageSync('token', token)
  let makeRequest = () = > {
    wx.request({
      url: url,
      data: param,
      method: method,
      header: {
        'Authorization': `Bearer ${token}`
      },
      complete: function (res) {
        if (String(res.statusCode) === '401') {
          // 往队列里丢待执行的函数
          store.state.ajaxList.push(makeNowRequest)
          login()
          return
        }
        if (String(res.statusCode) !== '200') {
          tips(res.data.err)
          return
        }
        callback(null, res.data)
      }
    })
  }
  makeRequest()
}

```