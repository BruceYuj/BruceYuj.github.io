---
title: 深入CSS基础之box model
date: 2019/2/13 19:13:07
cover: 	https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190219-deep-into-css-box-model.png
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: CSS中的box model
tags:
- CSS

categories:
- CSS
---
<!-- toc -->

## 阅读注意事项

1. 本篇文章的依赖主要是**CSS2.1 specification 8** 和 **CSS Box Model Module Level 3**。
2. 本篇整体比较细节和理论，可能会看起来枯燥，我尽量讲的逻辑简单些。个人认为有时候阅读枯燥的理论文章是有必要的。
3. 写这篇的文章主要目的在于完善自己的知识体系。因此希望大家能够多多指出文章中不恰当之地方。

## introduction
box model应该是CSS当中最核心的概念之一了，本篇主要讲述CSS当中的box model(盒模型)。

在看本篇内容之前，可以先自己回答下面几个问题：
1. 什么是box model？ box model 包含了哪些内容？ 它的作用是什么？ 它出现在浏览器处理文档的哪一个步骤？ 后续步骤是什么？
2. collapse margin的细节是什么？

## 什么是box model(盒模型)
[CSS2.1 specification 8.box model](https://www.w3.org/TR/2011/REC-CSS2-20110607/box.html) 第一句话就很好的描述了它的定义：
> The CSS box model describes the rectangular boxes that are generated for elements in the document tree and laid out according to the visual formatting model.

从这句话中我们可以得出来`box model`是什么：
`box model`是用来描述DOM树中元素生成的矩形盒子的，并且UAs会根据 `visual formatting model` 来布局这些盒子。

### box dimension(盒尺寸)
![typical box](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190219-deep-into-css-box-model-box.png)

这一小节是用来描述具体的矩形盒子的，每个盒子会有4块区域：
- content area
- padding area
- border area
- margin area
每块区域的尺寸由各自相关的属性决定。另外 `padding`, `border`, `margin` 可以被分割成`top, bottom, left, right` 四个小区域，并由具体的属性来定义。

下面我们来引入新的名词(为了后面的内容)，**`edge`:  每个盒子的周长被称为`edge`，即边界**。相应的，我们可以知道每个box都会有四个`edge`.
- content edge or inner edge: content area 的尺寸默认由元素的`width` 和 `height` 两个属性，元素的内容以及它的containing block决定
- padding edge： padding edge定义了元素的padding box（包含content 和 padding area）
- border edge: border edge定义了元素的border box（包含content，padding 和 border area）
- margin edge or outer edge:如果 margin的width为0，那么margin area和padding area 一样

**Note**： 一个box的content area， padding area 和 border area的 background样式由该元素的 `background`属性决定。也就是说`background` 默认会一直延伸到 border area，而 margin总是透明的。在CSS3中，我们可以使用background新属性来修改默认情况。

> CSS Box model 3 新增加一段：
When a box fragments—is broken, as across lines or across pages, into separate box fragments—each of its boxes (content box, padding box, border box, margin box) also fragments. How the content/padding/border/margin areas react to fragmentation is specified in [css-break-3] and controlled by the box-decoration-break property.

这一段主要讲的是当一个盒子跨行或者跨页时会被分割成多个box片段，那么对于该盒子的content，padding，border，margin在不同的box 片段中该如何表现？这些内容定义在[css-break-3](https://www.w3.org/TR/css-break-3/)中，并且可以使用`box-decoration-break`来控制。
另外其实在CSS2.1也有描述这样的场景，在8.6中描述了inline box跨行时盒模型该如何表现。

#### margin属性: `margin-top`, `margin-bottom`, `margin-left`, `margin-right` 和 `margin`
这些属性都是初学时就已经用的很熟的。因此，这里只强调几个注意事项：
先看属性的定义
> **'margin-top', 'margin-bottom', 'margin-left', 'margin-right'**
Value:  	`<margin-width>` | inherit
Initial:  	0
Applies to:  	all elements except elements with table display types other than table-caption, table and inline-table
Inherited:  	no
Percentages:  	refer to width of containing block
Media:  	visual
Computed value:  	the percentage as specified or the absolute length

1. `value`可以是 `<margin-width>` ，包含3种具体的取值：
    - `<length>`: 指定固定的宽度，比如`3px`, `1em`等
    - `<percentage>`: 百分比取值在由`computed value`转换成`used value`的时候计算，基数是`generated box's containing block`的`width`(也就是该元素的包含块的width). 而如果containing block的width会依赖该元素，那么具体表现在CSS 2.1中未定义的。
    -  auto， 后面具体讲
2. `margin-top` 和 `margin-bottom` 对于 `non-replaced inline elements`不起作用。
3. 在CSS3当中，上面这几个属性被称为physical margin属性。并且介绍了相对应的logical margin属性,比如`margin-block-start`,`margin-block-end`(这些属性跟文档的`writing mode`相关)。（logical margin和physical margin控制的是同样的margin区域，只是不同的表现形式而已。CSS3加入这个特性是因为不同国家的文档排列形式是不一样的，比如阿拉伯语就是从右往左写的）

**Note**: CSS3中添加的`margin-trim`属性由于目前没有任何浏览器支持，这里不做介绍(当前时间为**2019年2月**)。

#### collapsing margins
在CSS当中，2个或多个盒子相邻的垂直方向margins会合并成一个margin，这种合并被我们称为`collapse`（塌陷），而合并后的margin被叫做`collapse margin`.
首先，我们先强调一些margin不会合并的例外情况：
1. 水平方向上的margin不会collapse
2. 对于相邻的垂直方向上的margin不会塌陷有两种情况：
   - root元素盒子的 margin不会产生塌陷(在HTML当中就是 html元素)
   - 如果一个带有clearance(间隙，指的是`clear`属性导致元素位置移动产生的间隙)的元素的top margin和 bottom margin 存在相邻的margin，该元素的相关margin会和紧挨着的兄弟元素的相邻外边距合并，但合并后的外边距不会再和父级块的下外边距合并

那么，如何判断两个盒子的margin是相邻的(adjoining)呢? 需要同时满足下面那几个条件：
- 两个盒子都属于in-flow(流内) `block-level boxes`，并且处于同一个`block formatting model`。
- 没有 `line box`, `clearance`, `padding`, `border`将它们间隔开（这里高度为0的`line box` 会被忽略)。
- 都属于垂直相邻的盒边界，也就是满足下面的其中一种情况：
   - 盒子的上外边距和其第一个 `in-flow` 孩子节点的上外边距
   - 盒子的下外边距与其下一个紧挨着的`in-flow`中的兄弟节点的上外边距
   - 盒子的下外边距（并且该盒子的height computed value为auto）与其最后一个`in-flow`孩子节点的下外边距
   - 盒子的上外边距和下外边距，满足条件是：该盒子没有建立新的`block formatting model`，`min-height`为0，height的computed value为`0`或者`auto`，没有`in-flow` 孩子节点。

**另外需要注意的是折叠外边距也能够与另一个外边距相邻，只需要其外边距的任意一部分与另一个外边距相邻就算。**

**Note**：由上面的collapse margin 的定义我们可以得出以下的推论：
   - 相邻外边距可以由不具备兄弟或祖先关系的元素之间生成
   - `floated box`的margin和任何其他盒子的margin都不会合并（包括它流内的孩子节点）- **因为 `float box`不是流内的。** 
   - 绝对定位的盒子的margin和任何其他盒子的margin都不会合并（包括它流内的孩子节点） - **同样是因为绝对定位的盒子是流外的。**
   - 建立新的`block formatting context`的元素的margin不会与它的流内孩子节点的margin合并（比如floated box）
   - `inline-block`盒子的margin不会与其他盒子的margin合并（包括它的流内孩子节点）- **创建了新的`block formatting model`并且不是`block-level box`.**
   - `in-flow block-level box`的下外边距会与相邻的`in-flow block-level`兄弟节点的上外边距合并，除非该兄弟节点存在`clearance`
   - `in-flow block element`的上外边距会与它的第一个流内孩子节点（该孩子节点是`block-level`盒子）的上外边距合并，这里需要该元素不存在`top border`，不存在`top padding`，并且孩子节点也没有`clearance`
   - 一个`height`为`auto`并且`min-height`为`0`的`in-flow block-level box`的`bottom margin`会与它的最后一个`流内block-level`的孩子节点的`bottom margin`合并，条件是该盒子没有`bottom padding`，`bottom border`并且其孩子节点的`bottom margin`没有与具有clearance的top margin合并
   - 盒子自身的外边距也会合并，条件是`min-height`为`0`, 既没有上下边框，也没有上下内边距，height为0或者auto，且不包含line box（并且所有流内孩子的外边距都会合并）

**那么合并后的margin该如何取值呢？**
- 当两个或者更多的margin合并时，collapsed margin的宽度为被合并的所有外边距中的最大值
- 如果存在负margin，那么就从正相邻margin中的最大值减去负相邻margin中绝对值最大的值
- 如果没有正margin，就用0减去相邻margin中绝对值最大的值

**下面我们讨论最后一种特殊情况：**
如果一个盒子的上下外边距相邻，那么有可能外边距合并会穿透该盒子。在这种情况下，该元素的位置取决于合并margin的其他元素之间的关系：
 - 如果该元素的margins和它父元素的top margin合并，那么该盒子的top border edge和它父元素的top border edge一样
 - 否则的话，要么该元素的父元素没有参与margin collapse，要么只有该父元素的bottom margin参与。那么该元素的上边框edge和该元素bottom border非0时相同。 

**需要注意的是，被折叠外边距穿过的元素的位置不影响其他外边距正要被合并的元素的位置，其上边框边界的位置仅仅是用于布局这些元素的后代元素。**

#### padding属性： `padding-top`, `padding-right`, `padding-bottom`, `padding-left` 和 `padding`
padding属性定义了盒子的内边距区域的宽度。
> 'padding-top', 'padding-right', 'padding-bottom', 'padding-left'
Value:  	`<padding-width>` | inherit
Initial:  	0
Applies to:  	除table-row-group，table-header-group，table-footer-group，table-row，table-column-group和table-column外的所有元素
Inherited:  	no
Percentages:  	参照包含块的宽度
Media:  	visual
Computed value:  	指定的百分比或者绝对长度

padding属性和margin属性主要由下面几个不同：
1. padding的取值没有`auto`
2. padding值不可以为负
3. 适用的元素范围不一样

**和margin一样的是，百分比的基数是`generated box's containing block`的`width`(也就是该元素的包含块的width)**.

**note**：这里的`padding-top`等4个属性仍然是physical 属性。在CSS3当中添加了logical padding - `padding-block-start`等。

#### border属性
> 'border-top-width', 'border-right-width', 'border-bottom-width', 'border-left-width'
Value:  	`<border-width>` | inherit
Initial:  	medium
Applies to:  	所有元素
Inherited:  	no
Percentages:  	N/A
Media:  	visual
Computed value:  	绝对长度，或者'0'，如果border style为'none'或者'hidden'的话

对于`border-width`需要注意：
1. `<border-width>`只有关键字和`<length>`这两种选择，关键字是`thin`, `medium`, `thick`，大小依次变大，具体大小由用户代理决定
2. 边框宽度不能为负

>'border-top-color', 'border-right-color', 'border-bottom-color', 'border-left-color'
Value:  	`<color>` | transparent | inherit
Initial:  	'color'属性的值
Applies to:  	所有元素
Inherited:  	no
Percentages:  	N/A
Media:  	visual
Computed value:  	从'color'属性取值的话，取'color'的计算值，否则，就按指定值

**这里需要注意的是： `border-color` 的初始值是 `color` 属性的值，我们有些时候可以利用这个特性来实现一些特殊的要求**。

### 双向环境(bidirectional context)中inline-level 元素的盒模型
对于每一个line box， UAs必须为每一个元素产生的inline box渲染margin，border，padding以visual order的方式（而非logical order）
主要分为两种情况：
- `direction: ltr` : 元素出现的第一个line box的最左端生成的盒子具有左外边距，左边框，左内边距，并且元素出现的最后一个line box最右端生成的盒子具有右内边距，有边框，右外边距
- `direction: rtl`：元素出现的第一个line box的最右端生成的盒子具有右内边距，有边框，右外边距，并且元素出现的最后一个line box的最左端生成的盒子具有左外边距，左边框，左内边距

## reference
1. [CSS Box Model Module Level 3](https://www.w3.org/TR/css-box-3/)
2. [CSS2.1 box model](https://www.w3.org/TR/2011/REC-CSS2-20110607/box.html#box-model)
3. [CSS Display Module Level 3](https://www.w3.org/TR/css-display-3/)
4. [CSS Flexible Box Layout Module Level 1](https://www.w3.org/TR/css-flexbox-1/)
