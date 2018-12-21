---
title: 详细理解CSS中的outline
date: 2018/10/23 21:23:07
cover: https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/181023-CSS-outline.png
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: CSS3中的outline
tags:
- CSS

categories:
- CSS
---
<!-- toc -->
## introduction
讲到outline，也就是我们通常所说的描边,我们要分成两个部分来理解:
 - 一个是shorthand CSS property: `outline`, 由`outline-style`、`outline-width`和`outline-color`组成
 - 一个是独立的`outline-offset`

通常来讲，outline会被用在visual objects上面，比如buttons, active form fields等元素(make them stand out).

## syntax
首先很简单的是，我们知道`outline`由`outline-style`、`outline-width`、`outline-color`组成。
`outline`的initial value:
- `outline-color`: invert, for browsers supporting it, currentColor for the other
- `outline-style`: none
- `outline-width`: medium; 0 if `outline-style` is none.


1. outline-width允许的值和`border-width`一致
2. outline-style允许的值和`border-style`一致(除了缺少`hidden`).

**Note**:在CSS3当中，`outline-style`允许auto值。`auto` value允许user agent 去渲染一个定制的outline style(比如platform默认的style或者CSS表现不出来的style：a rounded edge outline with semi-translucent outer pixels that appears to glow).但是本规范并没有定义`outline-color`该如何表现在`outline-style`为auto的时候。并且UAs可能把`auto`解释为`solid`。

3. `outline-color`允许所有的colors。初始值是`inert`（确保focus border is visible）,但是需要注意的是很多UA是不支持`invert`值得，这个时候`outline-color`的initial value 就是`currentColor`。

### outline and border
上面我们在理解outline的属性的时候，发现，`outline`的值和`border`是大同小异的。下面我们说一说不同点。

outline和border的不同点：
- outlines不会占据空间
- outlines可能不是矩形的
- UAs 经常在elments的`:focus` state渲染outlines


1. 首先我们来了解一下第一点：outline不会占据空间。
这里的空间指的是布局空间。the outine is always on top并且不会影响box的position和size,也就不会导致reflow。

2. 第二点，outlines may be non-rectangular.
举个例子，如果一个element被切割成多行，这个时候会产生多个element boxs.而outline的表现应该包裹住所有的element's boxes（an outline or minimum set of outlines ）。outline的每一个部分需要fully connected（这里和border的表现是不一样的，比如inline elements在line被borken的时候会open on some sides）.这也是为什么说outlines are not required to be rectangular.
另外,outline会follow the border edge（但是目前对于follow the border-radius各大浏览器还没有支持）.
```html?linenums
  <div>
    <span>
       love你love
    </span>     
  </div>
```
```css?linenums
    div {
      margin: 100px auto;
      width: 10px;
    }
    span {
      outline: 5px solid red;
      /*width: 15px;*/
      border: 5px solid green;
    }
```
![outline-non-rectangular-chrome](http://oxnuwmm3w.bkt.clouddn.com/181023/outline-non-rectangular.PNG)
这是chrome、IE11和edge下的表现。
![outline-non-rectangular-firefox](http://oxnuwmm3w.bkt.clouddn.com/181023/outline-non-rectangular-firefox.PNG)
这是firefox下的表现.

**Note**: outline的位置可能会受到desendant box的影响。
```html?linenums
<style type="text/css">
  .outline{
    width: 13px;
    height: 13px;
    outline: 1px solid red;
  }
</style>
<div class="outline"></div>
<script type="text/javascript">
  const el = document.querySelector(".outline")
  el.textContent = !!~navigator.appVersion.indexOf("Chrome") ? "Chrome" : "FireFox"
</script>
```
![enter description here](http://oxnuwmm3w.bkt.clouddn.com/181023/outline-desendant-box.png)
**Note**: This specification does not define the exact position or shape of the outline, but it is typically drawn immediately outside the border box.

### offsetting the outline: `outline-offset` property
我们知道，默认情况下，outline被drawn在`border-edge`之外。但是，我们可以通过修改`outline-offset`来改变位置。

- `outline-offset` 的initial value 为0.
- 允许正负值，负值时outline会收缩，正值时会扩张。

**Note**: 当outline的值为large negative value的时候，各个浏览器的表现是不一样的。
![enter description here](http://oxnuwmm3w.bkt.clouddn.com/181023/outline-chrome.gif)
这是chrome下的表现，在outline-offset逐渐变小的时候，outline会变成十字形(具体的边界是在border-box的值 + outline-width的值，超过这个边界，outline会消失)
![enter description here](http://oxnuwmm3w.bkt.clouddn.com/181023/outline-firefox.gif)
这是firefox下的表现
![enter description here](http://oxnuwmm3w.bkt.clouddn.com/181023/outline-edge.gif)
这是edge下的表现

**总结如下**：
  上面3个动图分别是3个主流浏览器`outline-offset`值发生变化时的表现。chrome和edge的表现比较符合我们正常的感官，而firefox的表现就比较奇怪了。
  
## 使用
1. 可以用来模拟双层边框
2. 用作focus indicators.
`outline`通常可以被用于focus indicators。这些可以用于无障碍使用(web content Accessibility Guidelines).比如people who use screen readers or people with limited mobility.

常见的focusable elements:
- Links
- Buttons
- Form Fileds / Controls (text fields, select boxes, radio buttons, etc.)
- Menu items
- Things triggerd by hober, like tooltips
- Widgets, like a calendar picker

那么如何设计一个useful and useable focus indicators可以看reference 2.

## reference
1.[outline, W3C](https://drafts.csswg.org/css-ui-3/#outline)
2.[to design useful and usable focus indicators](https://www.deque.com/blog/give-site-focus-tips-designing-usable-focus-indicators/)