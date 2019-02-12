---
title: 深入前端之replaced element
date: 2019/2/12 10:23:07
cover: https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190212-deep-into-front-end-replaced-element.png
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: HTML中的replaced element
tags:
- Front End

categories:
- Front End
---
<!-- toc -->

## introduction
在阅读[CSS2 visual formatting model details](https://www.w3.org/TR/2011/REC-CSS2-20110607/visudet.html#inline-width)时，里面出现了大量的关于`replaced element`和`non-replaced element`的概念。`visual formatting model details`中将element在`inline-level and block-level`和`replaced and non-replaced`两个不同的维度展开描述。因此，如果我们想要深入理解CSS的内部世界，那么理解`replaced element`的概念就是必不可少的了。

那么本篇笔记主要解决下面几个问题：
1. 什么是`replaced element`?  
2. 什么是`intrinsic dimensions`?
3. 实际开发中如何计算`replaced element`的尺寸?
4. 对于前端来讲，HTML中具体有哪些elments可以被认为是`replaced element`?

## 什么是`replaced element`
w3c css2.2中对`replaced element`有一段简短的描述(来源于[3.conformance: Requirements and Recommendations](https://drafts.csswg.org/css2/conform.html)):
> An element whose content is outside the scope of the CSS formatting model, such as an image, embedded document, or applet. For example, the content of the HTML IMG element is often replaced by the image that its “src” attribute designates. Replaced elements often have intrinsic dimensions: an intrinsic width, an intrinsic height, and an intrinsic ratio. For example, a bitmap image has an intrinsic width and an intrinsic height specified in absolute units (from which the intrinsic ratio can obviously be determined). On the other hand, other documents may not have any intrinsic dimensions (for example, a blank HTML document).
 
 从这段基本的描述中，我们可以总结出什么：
 1. `replaced element`的content处于CSS formatting model控制之外。
 2. 常见的`replaced element`比如`image`, `embedded document`, `applet`
 3. `replaced element`通常包含`intrinsic dimensions`: *intrinsic width, intrinsic height, intrinsic ratio*。但是并不是所有的`replaced element`都同时包含这三个属性值。
 4. 推论：`replaced element`之所以冠以`replaced`的形容词，是因为这类element的内容会被指向的外部资源给replace掉。
 
上述的描述中引入一个新的名词`intrinsic dimensions`. 那么，我们来看一看什么事`intrinsic dimensions`.
 
## 什么是`intrinsic dimensions` 
从文字意义上来看，`intrinsic dimensions`表示的是*内部尺寸*。

另外，w3c CSS2.2中同样对`intrinsic dimensions`给出了一段简短的描述:
> The width and height as defined by the element itself, not imposed by the surroundings. CSS does not define how the intrinsic dimensions are found. In CSS 2 only replaced elements can come with intrinsic dimensions. For raster images without reliable resolution information, a size of 1 px unit per image source pixel must be assumed.

从上面的定义中我们可以总结出几点：
1. CSS2中只有`replaced element`有`intrinsic dimensions`.
2. `replaced element`的`width`和`height`都由`element`自己定义.

但是，我们光从上面的定义中很难真正的理解`intrinsic dimensions`.
因此我们再来看CSS3，[CSS Image Values and Replaced Content Module Level 3 5.1小节](https://www.w3.org/TR/css3-images/)有更加精确的定义：
> The term intrinsic dimensions refers to the set of the intrinsic height, intrinsic width, and intrinsic aspect ratio (the ratio between the width and height), each of which may or may not exist for a given object. 
These intrinsic dimensions represent a preferred or natural size of the object itself; that is, they are not a function of the context in which the object is used. CSS does not define how the intrinsic dimensions are found in general.
Raster images are an example of an object with all three intrinsic dimensions. SVG images designed to scale might have only an intrinsic aspect ratio; SVG images can also be created with only an intrinsic width or height. CSS gradients, defined in this specification, are an example of an object with no intrinsic dimensions at all. Another example of this is embedded documents, such as the` <iframe>` element in HTML. An object cannot have only two intrinsic dimensions, as any two automatically define the third.
If an object (such as an icon) has multiple sizes, then the largest size (by area) is taken as its intrinsic size. If it has multiple aspect ratios at that size, or has multiple aspect ratios and no size, then the aspect ratio closest to the aspect ratio of the default object size is used. Determine this by seeing which aspect ratio produces the largest area when fitting it within the default object size using a contain fit; if multiple sizes tie for the largest area, the wider size is chosen as its intrinsic size.

上面的definition主要包含4大块内容：
1. 术语`intrinsic dimensions`通常指代`intrinsic height`， `intrinsic width`, `intrinsic aspect ratio(width / height)`三种值的集合。但是对于不同`replaced element`中的`object`来讲，这三种value不一定都会存在。
2. `intrinsic dimensions`表示的是引入`object`的自己的`natural size`,也就意味着这是和`context`无关的。通常来讲，CSS并不会定义如何获得`intrinsic dimensions`.

这里如果仔细的话，我们这里使用了`element`和`object`两个名词。使用两个不同的名词是有一定的目的的。首先我们在[html specific replaced element](https://html.spec.whatwg.org/multipage/rendering.html#replaced-elements)中了解到`html`中的`replaced element`可以是`<audio>`,`<img>`,`<canvas>`,`<embed>`,`<iframe>`,`<input>`, `<object>`, `<video>`这几种。另外我们在什么是`replaced element`小节中推论到，`replaced element`之所以冠以`replaced`的形容词，是因为这类element的内容会被指向的外部资源给replace掉。所以这里就使用了`element`和`object`两个名词，`object`表示引入的外部资源本身，资源本身会有自己固定的的`dimensions`。
**Note**：但是，这并不意味着我们无法使用`CSS`来控制`html element`的外部展示，CSS控制`replaced element`的布局和`intrinsic dimension`两者并不冲突。具体细节可以看下面`concrete object size`的算法。

3. 主要是几个例子。`raster image`这个`object`同时拥有3个`intrinsic dimensions value`。`SVG iamges`可能只拥有一个`intrinsic aspect ratio`或者`intrinsic width`或者`intrinsic height`.`CSS gradient`3种`intrinsic dimension value`都没有。
4. 主要是一些边界情况，比如如果一个`object`有多个`size`。那么该选择哪一个？规范中是`largest size`将会作为该`object`的`intrinsic size`.如果有多个`aspect ratio`值，那么最接近`ratio of the default object size`的`aspect ratio`将会被选择。如果`default object size`使用`contain fit`的话，产生最大area的`aspect ratio`将会被选择。

**但是，这些定义对于我们实际开发当中有什么用呢？ 换个角度来讲，就是我们在实际开发中，`replaced element`的布局与尺寸该如何来计算呢？**

要给出上面的答案，我们可能还需要知道更多相关的名词：
> **specified size**
The specified size of an object is given by CSS, such as through the ‘width’ and ‘height’ or ‘background-size’ properties. The specified size can be a definite width and height, a set of constraints, or a combination thereof.
**concrete object size**
The concrete object size is the result of combining an object's intrinsic dimensions and specified size with the default object size of the context it's used in, producing a rectangle with a definite width and height.
**default object size**
The default object size is a rectangle with a definite height and width used to determine the concrete object size when both the intrinsic dimensions and specified size are missing dimensions.

有上面的定义可以知道：
1. **specified size**: *specified size*由CSS指定，比如通过`width`、`height`、`background-size`属性。*specified size*可以是有限的width和height, constraints集合或者两者的结合。
2. **concrete object size**:  *concrete object size*由*object's intrinsic dimensions*, *specified size*和*default object size*这三者决定，产生一个有*definite width and height*的矩形。
3. **default object size**: *default object size*是一个有*definite height and width*的矩形。

理解了上面三种`size`的定义，我们开始来看看在规范的[5.2 CSS Object Negotiation](https://www.w3.org/TR/css3-images/#object-negotiation)中**Objects in CSS 的size**是如何被决定并且渲染的：
>1. When an image or object is specified in a document, such as through a ‘url()’ value in a ‘background-image’ property or an src attribute on an `<img>` element, CSS queries the object for its intrinsic dimensions.
>2. Using the `intrinsic dimensions`, the `specified size`, and the `default object size` for the context the image or object is used in, CSS then computes a `concrete object size`. (See the following section.) This defines the size and position of the region the object will render in.
>3. CSS asks the object to render itself at the concrete object size. CSS does not define how objects render when the concrete object size is different from the object's intrinsic dimensions. The object may adjust itself to match the concrete object size in some way, or even render itself larger or smaller than the concrete object size to satisfy sizing constraints of its own.
>4. Unless otherwise specified by CSS, the object is then clipped to the concrete object size.

1. 第一步，当`image or object`在document当中被指定（比如`background-image`通过`url()`或者`<img>` element通过`src`来指定），CSS会先去查找`object`的`intrinsic size`.
2. 第二步，浏览器根据`intrinsic dimensions`,  `specified size`, 和 `default object size`来计算出`concrete object size`(具体计算方法看下面).
3. 第三步，CSS通知`object`根据`concrete object size`来渲染自己。CSS并没有规定当`concrete object size`和`intinsic dimensions`不一样的时候该如何渲染。`object`可能会自己调整自己来适应`concrete object size`或者为了满足自己的sizing constraints来大于或小于`concrete object size`。
4. 第四步，除非CSS另有指定，不然`object`会被剪切为`concrete object size`大小。

下面我们展开来讲解一下第二步- 如何确定`concrete object size`。在规范[5.3. Concrete Object Size Resolution](https://www.w3.org/TR/css3-images/#concrete-size-resolution)中介绍了`Default Sizing Algorithm`:
1. 如果`specified size`是有限的`width`和`height`，那么`concrete object size`的值就是`specified size`.
2. 如果`specified size`只有`width`或`height`其中一个。那么`concrete object size`的相同属性的值为`specified size`提供。另外一个值由以下方式来计算：
    1. 如果该object拥有`intrinsic aspect ratio`, 那么`concrete object size`将使用`intrinsic aspect ratio`来计算。
    2. 否则的话，我们再看缺少的dimension是否在`object`的`intrinsic dimensions`中存在，如果存在，则取用该值。
    3. 否则的话，`concrete object size`缺少的dimension则取用`default object size`.
3. 如果`specified size`没有任何的constraints：
    1. 如果object有`intrinsic height`或者`intrinsic width`，那么该`concrete object size`由`intrinsic size`来决定。
    2. 否则的话，`concrete object size`则被当做是相对于`default object size`的`contain constraint`。

另外，我们需要了解两个常见的`specified sizes`：`contain constraint`和`cover constraint`.两者都是通过`constraint rectangle`和`object's intrinsic aspect ratio`来决定`concrete object size`的大小。
- `contain constraint`: `concrete object size`将会被设置为一个根据`object's intrinsic aspect ratio`计算出来的最大的rectangle，并且该rectangle的width或height都各自比`constraint rectangle`的`wdith`和`height`小或相等。
-  `cover constraint`:`concrete object size`会被设置成一个根据`object's intrinsic aspect ratio`计算出来的最小的rectangle，并且该rectangle的width或height都各自比`constraint rectangle`的`width`和`height`大或相等。 
- 在上面两种情况下，如果object没有`intrinsic aspect ratio`，那么`concrete object size`就是specified constraint rectangle.




## HTML中有哪些元素可以被称为`replaced element`
从上面的内容，我可以知道`replaced element`的定义和计算细节。那么HTML中有哪些元素可以被称为`replaced element`呢？
这个我们得从[HTML standard 14.4 replaced element](https://html.spec.whatwg.org/multipage/rendering.html#replaced-elements)中来看。
规范中主要将`replaced element`分为两大类：
1. **Embedded content(嵌入内容)**:
   - 首先，`<embed>`, `<iframe>`, `<video>` 这三种元素被看作是`replaced elements`.
   - 对于`<canvas>`元素来讲，如果`<canvas>`中代表的是*embedded content*,那么该`<canvas>`元素也被当作*replaced element*(比如`<canvas>`中的content是*bitmap*)。否则的话，`<canvas>`会被看作是普通的*rendering model*中的元素。
   - 对于`<object>`元素来讲，和`<canvas>`一样得分为两种情况。`<object>`中展示的是*image, plugin, nested browsing context(比如iframe)* 时被看做是*replaced element*.其他情况下被认为是普通元素。
   - 对于`<audio>`来讲，通常browser会对这类元素有对应的*user interface*，如果`<audio>`的*user interface*被展示，那么`<audio>`就会被认为是`replaced element`.

2. **Images**: *HTML*当中中*images*主要分为2种：`<img>`和`<input type="image">` 
*user agent*主要按照下面的规则来render上面两种元素：
   - 如果该元素展示的*image*: 那么*user agent*将认为该元素是`replaced element`，并且根据CSS来render。
   - 如果该element不展示*image*,但是该element包含`intrinsic dimensions`（比如来自dimension attributes 或CSS rules）- 该element被认为是`replaced element`:
      - *user agent*认为image在将来会是肯用的并且会在适当的时候被render
      - 或者 element没有`alt attribute`
      - 或者当前`document`处于`quirks mode`.
   - 如果`<img>` element表示的是一些文本并且user agent不希望其发生改变： 该元素被看做是`non-replaced phrasing element`
   -  如果`<img>` element表示的是nonthing,并且user agent不希望其发生改变：该元素被看做是`empty inline element`
   -  如果`input element`不展示image并且user agent不希望其发生改变：那么该element被看做是`replaced element`（元素包含一个button，button的内容是element的可替换内容）。

**Note**：1.  对于`embed, iframe, img, object`元素或者`<input type="image">`元素的`align attribute`等于`center or middle`时，该element的`vertical-align`将会被设定为value（element的vertical middle和parent element baseline 对齐）。

## 总结
到这里为止，我们基本已经能够来回答在*introduction*中提出的4个问题。对于更多的细节（比如`width`和`height`如何计算等等）可以看specification。

## reference
1. [3.conformance: Requirements and Recommendations](https://drafts.csswg.org/css2/conform.html)
2. [CSS Image Values and Replaced Content Module Level 3](https://www.w3.org/TR/css3-images/)
3. [HTML standard 14.4 replaced element](https://html.spec.whatwg.org/multipage/rendering.html#replaced-elements)
4. [Replaced Elements in HTML: Myths and Realities](https://www.sitepoint.com/replaced-elements-html-myths-realities/)