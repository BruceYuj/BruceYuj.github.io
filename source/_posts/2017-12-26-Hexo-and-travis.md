---
title: 如何更'懒'的用Hexo和Travis CI搭建自己的blog
date: 2017/12/26 22:25:07
cover: http://oxnuwmm3w.bkt.clouddn.com/2017-12-26/Hexo-and-travis.jpeg
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: 使用travis CI和Hexo最'懒'的搭建自己的blog
tags:
- Hexo
- Travis CI
- Blog
categories:
- 博客
---

## 前言
社交是每个人的欲望，而作为一个技术人员，建立自己的技术博客也是一种重要的外界沟通。每篇博文都可以是自己一段时间的技术总结(最重要的是能够有规律的坚持下去)。
Anyway!首先看看一个新手在第一次尝试搭建技术博客时可能会选择的技术方案：
- wordpress + 云主机 + 插件(付费:目前aliyun 1核1G 600RMB左右,学生价120RMB左右，腾讯云类似)
- jekyll + github pages + markdown(免费: github默认支持jekyll,可以直接将原生文件放到github上,github自动编译)
- hexo + github pages + markdown(免费: 操作简单)

blog本质上是静态网站(无需太多交互)，核心是文章内容，因此上面的3种方式都可以选择。目前我搭建的方式是第三种。
ok,选择好了方式，直接在本地电脑搭建就好。具体的搭建方式网上一大堆，不过我建议直接看[官方文档](https://hexo.io/docs/index.html)即可，因为官方文档肯定是最新的。

**在这里强烈自荐我合作的主题*[hexo-theme-skapp](https://github.com/Mrminfive/hexo-theme-skapp),欢迎star和issue **

什么是hexo and travis CI？
两者集成需要注意什么问题？
详细步骤。