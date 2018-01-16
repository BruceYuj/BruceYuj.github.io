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
- wordpress + 云主机 + 插件(付费:目前aliyun 1核1G 330RMB/year左右,学生价120RMB/year左右，腾讯云类似)
- jekyll + github pages + markdown(免费: github默认支持jekyll,可以直接将原生文件放到github上,github自动编译)
- hexo + github pages + markdown(免费: 操作简单)

blog本质上是静态网站(无需太多交互)，核心是文章内容，因此上面的3种方式都可以选择。目前我搭建的方式是第三种。
ok,选择好了方式，直接在本地电脑搭建就好。具体的搭建方式网上一大堆，不过我建议直接看[官方文档](https://hexo.io/docs/index.html)即可，因为官方文档肯定是稳定最新的。

**在这里强烈自荐我合作的[hexo-theme-skapp](https://github.com/Mrminfive/hexo-theme-skapp),欢迎star和issue**

那么，下面重点来了，我们在使用一段时间的hexo之后，会发现一个非常蛋疼的问题：
每次想要更新博客，都需要到安装hexo的电脑上去更新,这真是反人类啊...

## 解决方案 
那么，我想要达到什么成果?
扩展多台电脑都可以更新博客

首先，我们先分析一下hexo，hexo本质上就是一个解析markdown文件的node程序，作用就是将markdown转换成对应的HTML文件，并提供了很多实用的封装命令。
那么，思路来了，github本身不就是一个版本控制的网站吗?那么我们可以将hexo作为我们博客的一个分支来同步到github上面。这样，一旦我们希望用另外一台电脑来更新博客，我们只需要将blog的代码库检下来即可。

但是，这样仍然会存在一些问题，比如我有一台win10的thinkpad笔记本，还有一台macbook pro，公司里面使用的ubuntu 16.04LTS. 我们每台电脑都git clone了github上面的代码库。然后，我们`npm install`了node的依赖包，这个时候我们可能会发现某些系统下hexo程序运行报错(对，没错我说的就是windows,比如node最基本的node-gyp)，虽然，我们可以通过各种补丁等等东西解决了系统问题，但是这个仍然非常的麻烦啊！

因此，我们我想能不能在多个系统的电脑上更新博客，而不需要考虑系统的兼容性问题。

下面就当当当的介绍下[travis CI](https://travis-ci.org/)



什么是hexo and travis CI？
两者集成需要注意什么问题？
详细步骤。