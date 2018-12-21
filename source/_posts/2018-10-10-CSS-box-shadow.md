---
title: 详细理解CSS的 box-shadow
date: 2018/10/10 21:23:07
cover: 
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: CSS3中的box-shadow
tags:
- CSS

categories:
- CSS
---
<!-- toc -->

## introduction
本篇主要是对`box-shadow`属性的一个简单了解。主要参考资料是[CSS Backgrounds and Borders Module Level 3](https://www.w3.org/TR/2017/CR-css-backgrounds-3-20171017/),目前最新版本是CR-css-backgrounds-3-20171017版本，虽然是比较稳定的，但是仍然会有被deprecated的可能。

### 兼容性
从caniuse搜索`box-shadow`得到的结果可以看出来，所有主流浏览器都支持`box-shadow`属性(PC and Mobile),并且IE9向上同样支持。因此，我们使用该属性来实现我们想要的功能是非常安全的。

## 简单的语法
```tex?linenums
Name: box-shadow
Value:  none | <shadow>#
Initial:  none
Applies to: all elements
Inherited:  no
Percentages:  N/A
Media:  visual
Computed value: any <length> made absolute; any specified color computed; otherwise as specified
Animatable: as shadow list
```
上面是`box-shadow`的一些基本构成属性。
从文档可以得知，`box-shadow` 属性主要实现的效果是attaches one or more drop-shadows to the box，也就是给css box添加阴影效果。

**note**：该property支持a comma-separated list of shadows(顺序是从front to back),简单语法表示就是`box-shadow: none |<shadow>#`

下面是单一的shadow的正则：
`<shadow> = inset? &&<length>{2,4}&&color?`
上面的正则表示为如果我们要使用`box-shadow`属性：需要2 - 4 length values、 一个可选的color值、一个可选的inset关键字（省略的length值默认为0，省略的color值默认为`currentColor`）。 
- 1st `<length>`(必选): the horizontal offset of the shadow. 正值表示相对于box向右偏移；负值表示相对于box向左偏移
- 2st `<length>`(必选): the vertical offset of the shadow. 正值表示相对于box向下偏移；负值表示相对于box向上偏移
- 3st `<length>`(可选): 表示blur radius（模糊半径）.负值是不允许的。blur value默认为0(这时shadow的边缘是锋利的，相当于没有模糊)。
- 4st `<length>`(可选):表示 spread distance。 取正值时，阴影扩大；取负值时，阴影收缩。默认为0，此时阴影与元素box一样大。

下面来两个实际图片理解一下：
```css?linenums
width: 100px; height: 100px;
border: 12px solid blue; 
background-color: orange;
border-top-left-radius: 60px 90px;
border-bottom-right-radius: 60px 90px;
box-shadow: 64px 64px 12px 40px rgba(0,0,0,0.4),
                     12px 12px 0px 8px rgba(0,0,0,0.4) inset;
```
![](http://oxnuwmm3w.bkt.clouddn.com/181010/css-box-shadow-1.png)

```css?linenums
div {
width: 150px;
height: 150px;
background-color: #fff;
box-shadow: 120px 80px 40px 20px #0ff;
/* 顺序为: offset-x, offset-y, blur-size, spread-size, color */
/* blur-size 和 spread-size 是可选的 (默认为 0) */
}
```
![](http://oxnuwmm3w.bkt.clouddn.com/181010/css-box-shadow-2.png)
### shadow shape, spread, and blurring shadow edges
从上面来看，我们基本了解了如何来使用`box-shadow`。但是一些具体的使用细节仍然是模糊的。
比如：
 - 阴影的形状是怎样的？
 - spread value是如何影响阴影的？和`inset`关键字的关系是怎样的？
 - blur radius是如何表现的？实际的、准确的效果是怎样的？


我们根据`inset`关键字分两种情况来考虑:`outer box-shadow`和`inner box-shadow`
首先来看shadow shape:
1. `outer box-shadow`: 假设spread distance为0. outer box-shadow的形状完美覆盖元素的border box.
2. `inner box-shadow` 同样假设spread distance为0.inner box-shadow的形状完美覆盖元素的padding box.

![基本使用](http://oxnuwmm3w.bkt.clouddn.com/181010/css-box-shadow-3.png)

**note**：在上面的图片中我们可以看出,当spread value不为0时，box shadow会有相对应的扩大或收缩(不管是不是inset)。这个时候，我们希望box的形状仍然有同样的视觉效果(尤其是存在 border radius的时候)，css 规范规定了如何对corner radii进行增加或减少，来达到视觉上的效果。

下面主要讲讲`blur radius`这个属性。
当`blur radius`的值非0时表示shadow需要被模糊。css规范并没有规定具体的实现算法(通常都是Gaussian filter算法)，但是要求shadow达到的效果必须是合理的。
具体表现在视觉效果上：blur radius会创建一个明显的color transition(长度几乎是blur radius的两倍，中心点就是shadow的边缘) 。而具体的颜色变化是从shadow的内部endpoint(full shadow color)到shadow的外部endpoint（almost transparent）。

## 应用
1. 常见的立体阴影效果。但是并没有filter的drop-shadow效果更佳逼真
2. 模拟多重边框。[demo](https://codepen.io/lonekorean/pen/EdCjk)

## reference
1.[w3c, box-shadow](https://drafts.csswg.org/css-backgrounds-3/#shadow-blur)
2.[MDN, box-shadow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow#%3Cblur-radius%3E)
3.[box-shadow](http://www.css88.com/archives/9360)


