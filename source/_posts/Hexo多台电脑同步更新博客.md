---
toc: true
title: Hexo多台电脑同步更新博客
date: 2018-09-12 19:20:07
tags: [Hexo]
---
七月毕业参加工作，忙了一个多月之后，九月准备开始继续更新博客。但是有个问题，有时候人在公司想写博客，必须要写完之后同步到笔记上，再用家里的电脑提交到github，很麻烦，所以学习参考了几篇博客搞起多台电脑同步更新博客。
> 主要思路是 利用git分支实现同步
> Hexo生成的静态博客文件默认放在master分支上
> Hexo的源文件都放在hexo分支（新建的一个hexo分支）上，换新电脑时直接git clone hexo分支

<!--more-->
### 第一步：已有一台按照hexo教程配置好的电脑
能够有这个需求的同学肯定已经完成了这一步。不再赘述。
这里推一篇搭建hexo博客的文章：[GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)

### 第二步：创建新的git分支 hexo
master分支与hexo分支内容比较：
1. 使用`hexo d -g`命令会自动同步静态博客文件到master分支
2. 使用`git push origin hexo`将hexo分支同步到github，这时本地完整的环境都会同步
**master分支同步静态博客文件**
![master分支](http://pexa42zzm.bkt.clouddn.com//18-9-12/46907723.jpg)
**hexo分支同步blog源文件**
![Hexo分支](http://pexa42zzm.bkt.clouddn.com//18-9-12/66203428.jpg)


### 第三步：第二台电脑安装git，node，hexo
具体怎么安装git，node和hexo请google教程。
1. `git clone` hexo分支
2. `hexo new "新文章"`编写自己的博客
2. `hexo d -g` 编译生成静态博客文件，并部署到github page
3. `git checkout hexo` 切换到hexo分支
4. `git pull` 建议每次换电脑写文章前都先pull一下，同步另一台电脑更新的文章
5. `git add .` 提交本地新写的文章
6. `git commit -m "提交新文章"`commit
7. `git push` push到远端

OK，到这里你就可以开心的在多个机器上写代码了。

> 参考：
> [1] [利用Hexo在多台电脑上提交和更新github pages博客](https://www.jianshu.com/p/0b1fccce74e0), 简书，Michaelhbjian.




