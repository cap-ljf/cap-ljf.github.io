---
toc: true
title: python爬虫之新浪微博评论
date: 2018-03-19 21:29:40
tags: [爬虫,新浪微博]
---

今天继续进行毕设工作，昨天完成今日头条爬虫之后，今天开始编写微博评论的爬虫。

    环境：
    ubuntu16.10
    python3.5
    
<!--more-->

一开始我用老一套爬虫方法"chrome登录网站，F12->network查找相应接口"，废老劲找到了评论接口，结果返回结果是html格式。于是google微博api，没想到找到这篇[博客](https://www.jianshu.com/p/92de44f0376a)，于是开启了轻松愉快的爬虫。

## [0] 寻找 weibo 评论接口
爬虫最重要的就是找到合适的接口，这是我目前对爬虫的理解，找到合适的接口能给你减少一半以上的工作。`微博`作为流量数一数二的平台，早就公开了开发者接口。那我们就来看看怎么使用weibo 开发者 api。

## **[1] 微博开放平台创建应用**
登录[微博开放平台](http://open.weibo.com)，注册个人或公司看自己需求，我注册的个人。
![Alt text](https://app.yinxiang.com/shard/s15/res/2b4877c5-9e3b-4c4c-af61-167835268cbf)
微连接-->其他-->随意填写信息，完成新应用创建.
然后就可以在我的应用中看到自己刚刚创建的应用。点击`应用信息`
![Alt text](https://app.yinxiang.com/shard/s15/res/624bbf2a-3107-4a44-8395-ceee0761e5a0)
我们需要的是这个新应用的两样东西：`App Key`和`App Secret`
点击`高级信息`，将`https://api.weibo.com/oauth2/default.html`填入`授权回调页`和`取消授权回调页`。
到这里我们已经完成开发者的申请。

## **[2] 微博api 接口**
在[微博API](http://open.weibo.com/wiki/%E5%BE%AE%E5%8D%9AAPI#.E8.AF.84.E8.AE.BA)页面找到评论接口：
![Alt text](https://app.yinxiang.com/shard/s15/res/1b38a717-564f-4aff-9537-e9303a6ed457)

## **[3] 编写简单测试代码**

首先安装 `sinaweibopy` 模块，我本来想尝试用python3装，但是遇到错误：![Alt text](https://app.yinxiang.com/shard/s15/res/aa7be1dc-5bdf-493a-b5d7-24d85f804c5a)。google 百度 都没有找到解决方法。结合[博客：](https://www.jianshu.com/p/92de44f0376a)和错误信息，觉得是这个模块并不支持python3，于是用python2.7进行了测试。

测试代码参考文章[开头链接](https://www.jianshu.com/p/92de44f0376a)。

## **结尾**
上面是很好的方法，但我并没有使用这种方法，而是爬去这两个接口：
- 热评接口：`'https://m.weibo.cn/single/rcList?format=cards&id=' + 单条微博id + '&type=comment&hot=1&page=' + 页码
`最新评论接口：`'https://m.weibo.cn/api/comments/show?id=' + 单条微博id + '&page=' + 页码`

我采用的方法：scrapy + redis +（cookies、ip、UserAgent池）+ Mysql。详情下回揭晓。




