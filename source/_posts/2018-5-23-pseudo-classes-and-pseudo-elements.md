---
title: 试试讲明白pseudo-classes和pseudo-elements
date: 2018/5/23 22:25:07
cover: 
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: 伪类和伪元素
tags:
- CSS

categories:
- CSS
---
<!-- toc -->
## introduction
最近在看基础，发现对伪类和伪元素的概念并不理解的透彻，而普通百度的文章都是写者自己的理解，因此觉得还不如直接阅读文档规范来的准确，下面就是今天的所得。

本文档主要用来解决下面几个问题：
1. 什么是pseudo classes 和 pseudo elements?
2. pseudo classes 和pseudo elements 两者的差别是什么？
3. 两者的实际应用是什么？开发中我们需要注意些什么？

## pseudo classes
首先来看下w3c规范(CSS3)里面的定义：
> The pseudo-class concept is introduced to permit selection based on information that lies outside of the document tree or that cannot be expressed using the other simple selectors.

w3c文档中说明了pseudo class的概念被引入的理由：用于选择不存在于DOM树当中的信息或者那些不能够通过常规选择器得到的信息。
1. 不存在于DOM树当中的信息，比如`:active` or `:visited`， 用于标书link的当前状态。
2. 不能够通过常规选择器获取的信息。比如`:first-child` or `nth-child()`等包含一些逻辑条件的选择器。

>A pseudo-class always consists of a "colon" (:) followed by the name of the pseudo-class and optionally by a value between parentheses.

这句话描述了pseudo class的基本语法：
```
selector:pseudo-class {
    property: value;
}
selector:pseudo-class() {
   property: value;
}
```

>Pseudo-classes are allowed in all sequences of simple selectors contained in a selector. Pseudo-classes are allowed anywhere in sequences of simple selectors, after the leading type selector or universal selector (possibly omitted). Pseudo-class names are case-insensitive. Some pseudo-classes are mutually exclusive, while others can be applied simultaneously to the same element. Pseudo-classes may be dynamic, in the sense that an element may acquire or lose a pseudo-class while a user interacts with the document.

这段话主要描述了pseudo-class的一些注意事项：
1.常规选择器可以以任何顺序使用伪类，伪类可以出现在任何位置
2.伪类是不区分大小写的
3.某些伪类是相互排斥的，而某些又是可以同时作用于同一个元素的、
4.伪类可能是动态的，比如用户在和document交互时元素会获得或失去一个伪类

**下面主要描述常用的pseudo classes**:

### dynamic pseudo-classes
> Dynamic pseudo-classes classify elements on characteristics other than their name, attributes, or content, in principle characteristics that cannot be deduced from the document tree.

规范中对于dynamic pseudo-classes的意义有了很好的描述：更多的看中元素的特性而不是他们的name这类的值。

dynamic pseudo-classes主要分为两类：
1. `:link`和`:visited`
2. `:hover` ，`:active` 和`:focus`

具体各个伪类的含义可以看[规范](https://www.w3.org/TR/selectors-3/#dynamic-pseudos)

### the target pseudo-class
`:target`用于匹配文档(页面)的URI中某个标志符的目标元素。

### structural pseudo-classes
常用的比如：
1. `:nth-child(an+b)`
2. `:nth-last-child(an+b)`
3. `nth-of-type(an+b)`
4. `:first-child`
5. `:last-child`
6. `:empty`
这些伪类在实际开发中都会起到非常好的效果。具体使用方法看[规范](https://www.w3.org/TR/selectors-3/#structural-pseudos)

### UI element states pseudo-class
这类pseudo classes主要展现UI元素的状态，常用的包括`:checked`、`:disabled`、`enabled`.

## pseudo-elements
>Pseudo-elements create abstractions about the document tree beyond those specified by the document language. For instance, document languages do not offer mechanisms to access the first letter or first line of an element’s content. Pseudo-elements allow authors to refer to this otherwise inaccessible information. Pseudo-elements may also provide authors a way to refer to content that does not exist in the source document (e.g., the ::before and ::after pseudo-elements give access to generated content).

pseudo-elements创建了抽象，这些抽象不存在于文档语言中。举例来说：
1. 文档语言没有提供机制来访问一个元素内容的第一行或第一个字符。pseudo-elements允许开发者来访问到这些信息
2. pseudo-elements也提供一种方式来使开发者获取到不存在源文档中的内容，比如常用的`::before`和`::after`.

这段话说明了pseudo classes和pseudo elements之间的差别。伪元素创建了抽象的容器，这个容器不包含任何DOM元素，但是可以包含内容。并且我们也可以给伪元素添加样式(这也是我们为什么要叫其element)。
比如`::first-line`: 获取第一行内容并将其加入到抽象容器中。
比如`::before`和`::after`。

> A pseudo-element is made of two colons (::) followed by the name of the pseudo-element.
This :: notation is introduced by the current document in order to establish a discrimination between pseudo-classes and pseudo-elements. For compatibility with existing style sheets, user agents must also accept the previous one-colon notation for pseudo-elements introduced in CSS levels 1 and 2 (namely, :first-line, :first-letter, :before and :after). This compatibility is not allowed for the new pseudo-elements introduced in this specification.

伪元素以`::`开头来区分伪类和伪元素。但是需要注意的是，考虑到兼容性，CSS1和CSS2当中的已存在的伪元素仍然可以使用`:`单冒号的方式。

>Only one pseudo-element may appear per selector, and if present it must appear after the sequence of simple selectors that represents the subjects of the selector. Note: A future version of this specification may allow multiple pseudo-elements per selector.

这段话描述了注意事项：
一个选择器只能使用一个伪元素，并且伪元素位于选择器的最后(目前的的规范)。

## 总结
1. pseudo classes引入的目的主要是为了弥补常规选择器的不足，以便于获取更多的关于元素的信息。而pseudo elements本质上创建了一个可以存放内容的抽象容器。
2. CSS3中伪类和伪元素的语法不同
3. 伪类和伪元素的使用规则不同，可以同时使用多个伪类，而伪元素在一个选择器当中只能出现一次，并且是在选择器的最后。

## reference
1. [w3c document - pseudo class ](https://www.w3.org/TR/selectors-3/#pseudo-classes)
2. [differ between pseudo classes and pseudo elements](https://stackoverflow.com/questions/8069973/what-is-the-difference-between-a-pseudo-class-and-a-pseudo-element-in-css)
3. [MDN, pseudo classes](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes)
4. [CSS3伪类和伪元素的特性和区别](https://www.cnblogs.com/ihardcoder/p/5294927.html)


