---
toc: true
title: 七牛云图床，年轻博主的图床
date: 2018-09-12 20:13:00
tags: [qiniu]
---
使用hexo搭建博客网站，需使用Markdown写文章。
网上有很多免费优秀的markdown编辑器，我用了一年的马克飞象，最近一周过期了无法继续同步印象笔记。而我不继续充值的原因是79一年的费用对我太贵，参加工作后花钱如流水，现在已经入不敷出，所以走上了免费之路。但我没有完全弃用马克飞象，它还是我写markdown文件的第一选择。无法使用markdown的同步功能以及图片上传功能，就只能另寻方法，七牛是一个好的选择。
<!--more-->
### 注册七牛云
https://www.qiniu.com/
注册七牛需要身份证号、手持身份证件照信息，填完信息之后三个工作日审批完成。我填完之后半个工作日就完成了验证
![](http://pexa42zzm.bkt.clouddn.com//18-9-12/43992768.jpg)

### 新建存储空间
审批完之后，点击 对象存储->新建存储空间
![](http://pexa42zzm.bkt.clouddn.com//18-9-12/43605580.jpg)
- 存储空间名称：起一个符合条件的
- 存储区域：选择离自己最近得区域
- 访问控制：**只能选择公开空间，因为博客的图片是要给自己网站使用的**

### 极简图床 插件上传图片
上传图片到七牛图床有多重方法，
1. 最原始的上传，找到官网进入自己的存储空间上传
2. 使用七牛云插件 [qiniu upload files](https://chrome.google.com/webstore/detail/qiniu-upload-files/emmfkgdgapbjphdolealbojmcmnphdcc) 上传， 具体可以参考：https://www.jianshu.com/p/44d818f781a7 （我尝试过不成果，提示400，incorrect region, please use up-z1.qiniup.com）
3. 使用极简图床网站
4. 使用dropzone上传

我采用的是[极简图床](https://jiantuku.com/)
同样的简单注册，之后会看到这个界面，没有的点击设置符号
可以自己选一种图床，这里介绍七牛云。进入到七牛云个人中心->密室管理，复制AK和SK
进入刚刚新建的存储空间，复制域名
![](http://pexa42zzm.bkt.clouddn.com//18-9-12/54148274.jpg)

如果想设置图片水印可以参考 https://shimo.im/docs/IUQsi7cDPOwL9aZc
OK，现在请开始你快乐的blog吧！

> 参考
> [1] [使用七牛云作为图床获取外链方式总结](https://juejin.im/post/5a71ac325188257350518a23)，掘金，Jaybo
> [2] [使用样式选项自定义图片水印](https://shimo.im/docs/IUQsi7cDPOwL9aZc)