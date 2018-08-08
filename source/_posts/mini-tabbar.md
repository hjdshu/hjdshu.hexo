---
title: 小程序tabbar的一些建议
date: 2018-08-07 17:25:50
categories: 小程序
tags: 小程序
---

### 一、原生tabbar配置

```
  config: {
    pages: ['pages/index/index', 'pages/me/me'], //配置在tabbar的页面必须要在pages里
    tabBar: {
      color: '#8a8da4', //文字颜色
      selectedColor: '#5fafff', //选中颜色
      backgroundColor: '#ffffff', //背景颜色
      borderStyle: 'black', // bar-top 的边框色(一般都是black)
      list: [
        {
          'pagePath': 'pages/index/index',
          'text': '首页',
          'iconPath': './static/images/spc1.png', //图标路径，只能为本地图片，不能为远程地址
          'selectedIconPath': './static/images/spc1_1.png' //被选中图标路径
        },
        {
          'pagePath': 'pages/me/me',
          'text': '我',
          'iconPath': './static/images/spc4.png', //图标路径，只能为本地图片，不能为远程地址
          'selectedIconPath': './static/images/spc4_1.png' //被选中图标路径
        }
      ]
    }
  }
```

### 二、自己实现tabbar

- 自己写tab，我大概总结为两种。 
- 举例你有个两个tab页面， A-B， 一种方案是， AB在一个page上，js实现根据tab切换view视图。 这样会带来两个问题，第一，一旦你以后变成多个tab，比如4个，5个，你页面的状态管理会非常复杂。 第二，页面是同一个path，PVUV数据统计也很难统计和区分（path就跟URL, 实际上两个页面是同一个url，这种方案强烈不推荐。

- 第二种方案是，写一个tab组件，分成两个page，每个page加上自己的tab组件。 但是这样的话，看起来好像也解决了统计和状态管理的问题。但是这种tab就不是真正意义上的tab了，切页面的时候，A->B实际上是一个从首页到内页的过程（到B页面会产生访问路径）,  当然有一种方案是使用reLunch跳转可以做到(不产生访问路径)，但是A-reLunch到B的时候，A页面的状态会丢失(从B再点击回到A的时候，页面数据需要重新获取和渲染)，这样也得不偿失。
- 综上，非常不推荐自定义tabbar

### 三、原生tabbar的好处

原生的tabbar是微信实现的tabbar，当tab来回切换的时候，不需要状态管理，页面数据不会销毁，也不用担心统计和交互上的问题, 另外原生tab微信还提供了显示和关闭红点的api，可以说是非常适用了。