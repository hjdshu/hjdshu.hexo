---
title: 小程序爬坑-图片生成
date: 2018-08-08 09:20:58
tags: 小程序
---

图片生成，这个业务是小程序非常常见的需求了，但是根据不同的场景，往往都有会不同的解决方案

### 客户端-利用canvas生成
> 小程序的canvas原生能力跟web的canvas不一样，小程序的canvas提供的api有限，比如计算文字宽度的api是没有的, 官方文档地址:  https://developers.weixin.qq.com/miniprogram/dev/component/canvas.html#canvas

#### 小程序canvas的绘制注意事项
1. 适配: 绘制图片之前，先要搞清楚当前机器的真实尺寸，计算出单位标准
```
  //获取当前机器的尺寸
  let imageWidth = wx.getSystemInfoSync().windowWidth 
  let bil = imageWidth / 375 //拿到单位bil
  //根据这个单位来决定画布元素的大小
  ctx.drawImage(img_src, 0, 0, 340 * bil, 290 * bil) 
  //字体大小也是这样
  ctx.setFontSize(11 * bil) 
  ctx.setFillStyle('rgba(131,131,157,1)')
  ctx.fillText('长按识别小程序码', 179 * bil, 330 * bil) 
```
2. cx.drawImage
小程序里没有跨域问题，但是ctx.drawImage(imgsrc, x, y, w, h), imgsrc必须为图片的本地路径，当服务器给你一个远程地址的时候，必须使用 wx.downloadFile, 拿到res.tempFilePath，才能绘制

### 服务器生成
> 当绘制的精确度要求很高，图片的内容复杂一些，而由于各种机型对于canvas的支持都有差异的时候，这个时候用服务端来生成或许是一个不错的选择。