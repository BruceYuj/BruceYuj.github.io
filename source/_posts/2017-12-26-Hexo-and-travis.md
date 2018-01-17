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
<!-- toc -->
## 前言
社交是每个人的欲望，而单从一个技术人员角度来讲，建立自己的技术博客也是一种重要的外界沟通。blog中的每篇博文都可以是自己一段时间的技术总结(最重要的是能够有规律的坚持下去)。
![](http://oxnuwmm3w.bkt.clouddn.com/2017-12-26/socialization.jpeg)
Anyway!首先我们来看看一个新手在第一次尝试搭建技术博客时可能会选择的技术方案：
- wordpress + 云主机 + 插件(付费:目前aliyun 1核1G 330RMB/year左右,学生价120RMB/year左右，腾讯云类似)
- jekyll + github pages + markdown(免费: github默认支持jekyll,可以直接将原生文件放到github上,github自动编译)
- hexo + github pages + markdown(免费: 操作简单)

blog本质上是静态网站(无需太多交互)，核心是文章内容，因此上面的3种方式都可以自行选择。目前我搭建的方式是第三种。
ok,选择好了方式，第一步肯定是在本地电脑搭建hexo。具体的搭建方式网上一大堆，这里我就不讲解如何搭建了，不过我建议直接看[官方文档](https://hexo.io/docs/index.html)即可，因为官方文档肯定是稳定最新的。

**另外,在这里强烈自荐我合作的[hexo-theme-skapp](https://github.com/Mrminfive/hexo-theme-skapp),欢迎star和issue**

前提讲完了，开始切入重点，我们在使用一段时间的hexo之后，会发现一个非常蛋疼的问题：
**每次想要更新博客，都需要到安装hexo的电脑上去更新,这真是太麻烦了...**
也就是说，最基础的跨电脑都会有困难。

## 解决方案 
首先，我们先来分析一下hexo:hexo本质上是一个解析markdown文件的node程序，作用在于将markdown转换成对应的HTML文件，并提供了很多实用的封装命令。

我们简单想一想，是不是可以直接版本控制来维护我们的blog程序，而github本身不就是一个版本控制的网站吗?

那么我们可以将hexo作为blog代码库的一个分支来同步到github上面。这样，一旦我们希望用另外一台电脑来更新博客，我们只需要将blog的代码库检下来即可。这样多台电脑就相当于多个开发共同开发维护blog代码库。

但是，这样用着一段时间后，仍然会存在一些小问题。
先举个例子：比如我有一台win10的thinkpad笔记本，还有一台macbook pro，公司里面使用的ubuntu 16.04LTS. 这3台电脑我都会维护更新我的博客。首先，这3台电脑都git clone了blog代码库。然后，我们`npm install`了node的依赖包，这个时候我们可能会发现某些系统下hexo程序运行报错(对，没错我说的就是windows,比如node最基本的node-gyp在win下安装就很麻烦)，虽然，我们可以通过各种补丁等等东西解决了系统问题，但是这个仍然非常的麻烦啊！

那能不能在多个系统的电脑上更新博客，而不需要考虑系统的兼容性问题？

下面就DuangDuangDuang的介绍下[travis CI](https://travis-ci.org/),travis CI是一种构建和测试的自动化工具。从其简称CI就可以理解，Travis CI提供的是持续集成服务(Continuous Integration).它可以绑定github上面的项目，只要有新的代码提交，就会自动抓取，然后travis会提供一个运行环境，执行测试，完成构建并部署到服务器。

通过travis CI，我们每次只要更新markdown文件，网站内容就能自动更新了。在这里就不具体的讲如何入门travis CI了，具体学习可以点击[travis CI官网](https://docs.travis-ci.com/)或者[阮一峰travis CI教程](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)
![](http://oxnuwmm3w.bkt.clouddn.com/2017-12-26/success.jpeg)
## 详细搭建步骤
我们在部署博客时，`hexo d`就可以搞定，但是问题在于Travis CI本身并没有对github库进行push操作的权限。如果我们直接将密钥直接放在开源库中，则相当于将代码库的提交权限开放给所有github的使用者，因此，我们需要一些加密操作。
### 1. travis本地环境搭建
```bash
brew install ruby 
```
首先本机安装ruby。

### 2. Deploy Key
```bash
$ ssh-keygen -t rsa -C "your_email@example.com"
```
首先通过`ssh-keygen`生成一个SSH key专门提供给github中blog库使用。在生成SSH key时，将passphrase留空，因为在travis中输入密码比较麻烦。
然后将制作完成的Public key复制到github blog代码库的Deploy key里面，如下：

### 3. 加密 private key
```bash
$ gem install travis // 安装travis命令行工具
$ travis login --auto // 命令行登录travis
```
```bash
$ travis encrypt-file ssh_key --add
```
这里假设private key的文件名为`ssh_key`,travis会加密产生`ssh_key.enc`,并自动在`.travis.yml`的`before_install`位置自动插入解密指令。
*注意*：这里在windows系统下使用`travis encrypt-file`命令加密生成的`ssh_key.enc`在travis执行时会在解密密钥时失败。具体原因在travis的issue中可以查看[File decryption fails on Windows](https://github.com/travis-ci/travis-ci/issues/4746).最简单的做法就是在linux或mac环境下生成该文件。

### 4. 设定 .travis.yml
首先我们在blog根目录下建立`.travis`文件夹和`.travis.yml`文件。
然后我们将生成的`ssh_key.enc`移动到本地blog项目的`.travis`文件夹下面，并在`.travis`目录下创建`ssh_config`文件，然后配置Travis上的SSH设定。
```
Host github.com
  User git
  StrictHostKeyChecking no
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes
```
因为我们刚刚修改了`ssh_key.enc`的位置，因此我们需要修改`.travis.yml`中刚刚插入的解密指令(不要照抄，不同的环境修改不一样)
```bash
- openssl aes-256-cbc -K $encrypted_06b8e90ac19b_key -iv $encrypted_06b8e90ac19b_iv -in .travis/ssh_key.enc -out ~/.ssh/id_rsa -d
```
该命令会利用openssl来解密private key,并将解密后的文件放在 `~/.ssh/id_rsa`，接着我们指定该档案的权限:
```bash
- chmod 600 ~/.ssh/id_rsa
```
然后将private key加入到系统中:
```bash
- eval $(ssh-agent)
- ssh-add ~/.ssh/id_rsa
```
接下来将`ssh_config`文件复制到`~/.ssh`目录下：
```bash
- cp .travis/ssh_config ~/.ssh/config
```
为了使git操作能够正常进行，我们需要事先设定git的使用者讯息：
```bash
- git config --global user.name "Bruce"
- git config --global user.email "your email address"
```

最后的结果大概如下：
```yaml
language: node_js
node_js: stable
cache:
  directories:
  - node_modules
before_install:
  # Decrypt the private key
  - openssl aes-256-cbc -K $encrypted_0c8703cca11f_key -iv $encrypted_0c8703cca11f_iv -in .travis/id_rsa.enc -out ~/.ssh/id_rsa -d
  # Set the permission of the key
  - chmod 600 ~/.ssh/id_rsa
  # Start SSH agent
  - eval $(ssh-agent)
  # Add the private key to the system
  - ssh-add ~/.ssh/id_rsa
  # Copy SSH config
  - cp .travis/ssh_config ~/.ssh/config
  # Set Git config
  - git config --global user.name "Bruce"
  - git config --global user.email "444048170@qq.com"
install:
  - npm install
script:
  - hexo g
after_success:
  - hexo deploy
branches:
  only:
    - hexo
```

## reference
1.[tommy351,continuous-deployment-to-github-with-travis](https://zespia.tw/blog/2015/01/21/continuous-deployment-to-github-with-travis/)
