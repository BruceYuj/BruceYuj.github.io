---
title: 深入CSS之属性赋值
date: 2019/2/12 10:23:07
cover: 	https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/2019-2-13-deep-into-css-assign-property-values.png
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: CSS中的属性赋值
tags:
- CSS

categories:
- CSS
---
<!-- toc -->

## 阅读注意事项

1. 本篇文章的依赖主要是**CSS specification**。
2. 本篇整体比较细节和理论，可能会看起来枯燥，我尽量讲的逻辑简单些。个人认为有时候阅读枯燥的理论文章是有必要的。
3. 写这篇的文章主要目的在于完善自己的知识体系。因此希望大家能够多多指出文章中不恰当之地方。

## introduction
我们在前端开发中写CSS时，如果观察仔细的话，很多时候浏览器渲染出来的元素的属性值和我们在CSS代码中指定的有差异。尽管这些差异可能对于人类视觉的感官没有什么差别，但是作为一个开发者，还是有必要了解这些差异的源头的。

**那么，导致属性值差异的原因是什么呢？换句话说，浏览器内部是如何来决定最后渲染的属性值的？**
让我们一同来畅游[CSS specification](https://www.w3.org/TR/2011/REC-CSS2-20110607/cascade.html#computed-value)。

首先，提出本篇文章想要解决的几个问题：
1. *user agent* 内部是如何得出最终的属性值的？
2. *specified value*, *computed value*, *used value*, *actual value* 这几个值究竟代表着什么？
3. 子元素属性对父元素的继承的细节是怎样的？
4. *user agent* 究竟是如何决定样式表规则的优先级的？

## 属性赋值
我们知道， 在 *user agent* 解析完文档并生成DOM树后， 它需要给DOM树中的每一个元素的所有属性进行赋值。

属性赋值计算的具体的过程主要分为4个步骤：
1. **specified value**:  根据指定得到 *specified value*
2. **computed value**: 然后 *specified value* 被解析成可以用于继承的值，也就是 *computed value*. 
3. **used value**: 如果有必要的话，在这一阶段将 *computed value* 转成绝对值，也就是 *used value*.
4. **actual value**: 最后根据本地环境的限制， *used value* 会被转换成 *actual value*.

下面，我们详细的理解每一个计算步骤。

### Specified Values
对于计算的第一步， *user agent* 会给每一个元素的每个属性都赋予一个 *specified value* . 那么这个 *specified value* 从何而来呢？ 我们来看看规范给出的机制：
 1. 如果 **cascade(层叠，详细解释看cascade小节)** 可以产生一个值，那么 *specified value* 就等于这个值。
 2. 否则，如果该属性是可以继承的并且该元素不是DOM树的根元素，那么 *specified value*就等于父元素的 *computed value* .
 3. 否则，使用该属性的 *initial value*. 而 *initial value* 来自于规范中属性的定义。

规范中属性的定义如下：
> **width**
Value:  	`<length> | <percentage> | auto | inherit`
Initial:  	auto
Applies to:  	all elements but non-replaced inline elements, table rows, and row groups
Inherited:  	no
Percentages:  	refer to width of containing block
Media:  	visual
Computed value:  	the percentage or 'auto' as specified or the absolute length

### Computed Values
1. 对于计算的第二步，*specified values* 通过层叠被转换成 *computed values*。也就是说，计算 *computed values* 时不需要 *user agent* 去渲染文档。**举个例子， 在css代码中使用 `em` 和 `ex` 单位的资源在这个阶段可以被计算为 `pixel`或者绝对长度。**

2. 如果*user agent* 无法将 *specified values* 转换为 绝对值的话，那么*computed values* 就直接使用 *specified values*.

**注意**：一个属性的*computed value*由规范中属性定义的 *computed value*一行决定(比如上一小节`width`属性定义中的*computed value* 一行)。
另外，即使属性不适用（于当前元素），其*computed value* 也存在，具体定义在规范属性定义中的`Applies To`行。然而，有些属性可能根据属性是否适用于该元素来定义元素属性的*computed value*

### Used Values
我们从上面知道，*user agent* 在计算 *computed values* 时会尽可能的不格式化文档。但是，有些时候，某些属性值只能在布局完成时确定。**举个简单例子，如果一个元素的 `width: 50%`, 而具体的 width计算需要获得 containing block的width。**

而在计算的第三步，*used value*  就是将 *computed value* 和 *相关的依赖* 结合来得到一个绝对值（如果本身 *computed value* 本身没有其他依赖，已经是绝对值，那么 *used value* 就等于 *computed value*）。

### Actual Values
通常来讲， *used value* 就是最终用来去渲染的值。但是，有些时候*user agent* 无法在给定的环境当中使用该值。
比如，user agent可能只能使用整数像素来渲染 border，如果used value是浮点数，就不得不对该值进行近似处理。
而 *actual value* 就是经过近似处理后的可以使用的 *used value*。

到这里，我们已经基本对元素属性值的计算过程有了详细的了解。

## 继承
属性值的继承主要出现在获取 *specified value* 的第二步。也就是说 在层叠无法给出一个值的情况下，如果该 属性是可以继承的并且该元素不是DOM树的根元素，*specified value*就等于父元素的 *computed value* 。
在继承发生的情况下， 子元素属性继承 父元素的 *computed values*。换句话讲，该子元素的这个属性的 *specified value* 和 *computed value* 都等于其父元素该属性的 *computed value* 。

分析下面的代码：
```html
<style>
body {font-size: 10pt}
h1 {font-size: 130%}
</style>
<body>
	<h1>A  <em>large</em>  heading</h1>
</body>
```
 根据规范：
 1. `h1 element`的 `font-size`属性的*computed value* 为 `13pt`（`130% * 10pt`）. 
 2.  `em element`的 `font-size`属性值可以继承，因此`em element`的 `font-size`的 *specified value* 和 *computed value* 都为 `13pt`. 如果*user agent* 没有可用的`13pt`字体，那么 `font-size` 的 *actual value* 可能是其他值，比如 `12pt`.

**Note**: 继承遵循DOM树, 并且不会被 *anonymous boxes* 给截断。

### 层叠中的 `inherit` 值
每一个属性也可能有一个`inherit` 层叠值。 `inherit` 层叠值可以用来显示实现属性值的继承，也就是说它可以用在一些本身不是继承的属性上。另外需要注意的是如果`inherit`被设置在根元素上面，该属性会被赋值为*initial value*.

## The Cascade(层叠)
样式表通常可能有三个来源：author, user 和 user agent。我们分开来了解一下：
- **author**: author根据文档语言约定来给源文档指定样式表。比如，在HTML当中，样式表可以包含在文档中(`<style></style>`)或者来自外部链接(`<link rel="stylesheet" type="text/css" href="theme.css" />`)。
- **user**：用户可能会给特殊的文档指定样式信息。比如，用户可以指定一个包含样式表的文件或者user agent提供一个界面来让用户生成用户样式表。
- **user agent**:  user agent必须应用一份默认样式表(或者表现的它做了一样)。user agent的默认样式表需要按照满足文档语言的一般表现预期的方式来呈现文档语言元素（比如对于可视化浏览器，HTML中的`em` 元素应该按照斜体来表示）。CSS2中推荐的 [默认样式表](https://www.w3.org/TR/2011/REC-CSS2-20110607/sample.html)

**那么问题来了，当不同来源的样式表相互重叠时，我们该运用哪一个样式表呢？**
上述的3种样式表将在一定范围内重叠，并且他们按照重叠互相影响。
**因此，CSS层叠给每一个样式规则赋予了权重，权重最高的规则将会被使用。**

默认情况下：
 - author style sheet的规则比 user style sheet的规则的权重更高。但是对于 `!important`规则，优先级是相反的。
 - 所有author style sheet和user style sheet的规则都比 UA的默认样式表的规则的权重更高。

### cascading order(层叠顺序)
为了找出一个 `element property`的值，user agents按照下面的步骤来排序：
1.  找出target media type下面，所有适用于该元素和目标属性的声明。
2. 根据importance(normal or important)和origin(author, user, user agent)来升序优先级排序：
    1. user agent 声明
    2. user normal 声明
    3. author normal 声明
    4. author important 声明
    5. user important 声明
3. 对于相同importance 和 origin的规则按照选择器的 `specificity`来排序。更特殊的选择器规则将会覆盖一般的。另外，对于pesudo-elements 和 pesudo-classes分别按照*normal elements* 和 *normal class* 对待。
4. 最后，按照指定的顺序(也就是出现的先后顺序)来排序：也就是说，在权重，origin和specificity相同的情况下，出现在后免得被指定的声明将会被采用(通过`@import`引入的样式表声明将被认为在样式表自身的所有声明之前)。

### `!important` 规则
我一直觉得想要真正的理解一门技术，一个技术的知识点。我们得带入自己到技术开发者的环境中，了解技术出现的历史背景、想要解决的问题。
CSS规范作者一直尝试在author和user样式表之间建立平衡。因此，默认情况下，author样式表的规则会覆盖user样式表的规则。
但是为了平衡，`!important`声明比normal声明优先级更高。如果author样式表和user样式表都包含`!important`声明，那么user的`!important`声明会覆盖author的`!important`规则。这样的目的是给与用户在表现上的特殊需求的控制，来提高用户的访问效果。

**Note**: 赋予简写属性(比如`background`)`!important`规则，等价于给所有的子属性都赋予`!important`规则。

### 计算选择器的 specificity
CSS规范当中在计算选择器的`specificity`时候采用了`a, b, c, d`四个变量，优先级为降序。下面是具体变量的计算方式：

- 如果声明来自元素的`style`属性而不是选择器样式规则，那么`a=1`，否则为0
- 计算选择器中ID属性的数量(=b)
- 计算选择器中其他属性和伪类的数量(=c)
- 计算选择器中元素名和伪元素的数量(=d)

这里，需要强调的是，选择器的`specificity`只根据选择器的形式来定。
举个例子， `[id=p33]`形式的选择器只被算作是一个属性选择器(`a=0, b=0, c=1, d=0`),尽管id属性在源文档的DTD中被定义为ID。

下面，我们给出详细的例子：
```css
 *             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
 li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
 li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
 h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
 ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
 li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
 #x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
 style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
```
```html
<head>
    <style>
        #x97z{color: red;}
    </style>
</head>
<body>
    <p id="x97z" style="color: green;"></p>
</body>
```
上面的html代码根据层叠规则3，`style="color: green;"`的特殊性要高于选择器` #x97z{color: red;}`, 因此p元素的color值为green。

**note**: 这里对于HTML中的表现型属性（比如color）被翻译成相应的特殊性为0的CSS规则不做过多的讨论。

## 总结
到这里为止，我们应该可以很好地回答在introduction当中提出的4个问题了。

## reference
1. [6 Assigning property values, Cascading, and Inheritance](https://www.w3.org/TR/2011/REC-CSS2-20110607/cascade.html#computed-value)