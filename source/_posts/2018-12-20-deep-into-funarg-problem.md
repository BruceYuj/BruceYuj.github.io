---
title: 深入理解Computer Science中的funarg problem
date: 2018/12/20 22:23:07
cover: https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/181220-understanding-funarg-problem.png
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: CSS3中的outline
tags:
- Computer Science

categories:
- Computer Science
---
<!-- toc -->

## introduction
本篇主要是对computer science领域当中的Funarg problem以及涉及到的知识概念进行学习讨论。
很多人会有疑问，学习这些相关理论的意义在于何处，看起来对于自己的工作开发没什么明显的作用，但在我的理解当中这些理论就是CS世界的基石。举个简单的例子，世界上有N多种编程语言，但上升到语言学概念，很多语言的特性是类似的，构造也是类似的，解决的问题也是类似的。而通过学习这些理论会很好的让我们理解各个语言的特性的共同点和不同点，也会更好的理解自己所在的行业，而不是整天局限在前端、后端这种简单的职业划分或者某一种语言当中。

## terminology(术语)
本篇会牵扯到一些专业术语，而对于出现的这些术语我一般不会翻译成中文（对专业词汇的认识还是很有必要的）。
- first-class citizen
- first-class function
- -closure
- non-local variable
- stack-based memory allocation
- heap-based memory allocation
- call stack
- funarg problem
- tail call
- tail recursion


### first-class citizen 和 first-class function
>first-class object(first-class citizen): 在编程语言的设计当中，如果一个entity支持其他entity支持的所有操作，那么该entity可以被叫做first-class citizen,也就是常说的一等公民。这些操作通常包括 passed as an argument, returned from a function, modified 和 assigned to a variable.

我们来实际看一下，目前流行的编程语言对于first-class citizen的实现：
 - 在一些出现比较早的编程语言当中arrays和strings不是first-class. 比如C语言当中不支持array assignment,如果我们在C中将一个array传递给function，实际上只是该array的第一个element被传递(C支持以array pointer的方式来assignment).
- 在大部分语言中，data types 不是first-class，尽管在一些OO语言当中，class是first-class objects并且是metaclass的instance,比如Python。

> first-class function: 在该编程语言当中，function属于first-class citizen.

很明显，一个支持first-class function的编程语言，需要支持以下几种特性：
-  passing functions as arguments to other functions(function可以成为函数的参数)
- returning them as the values from other functions（function的返回值可以使function类型）
- assigning them to variables（function可以用来赋值）
- storing them in data structures（function可以保存在数据结构中）
**note 1**：在支持first-class function的语言当中，function的name没有什么特殊地位，它们只是function type的variables。  另外，支持first-class function对于function proagramming是必须的。

**note 2**: first-class function还会引申到另外一个概念: higher-order function(高阶函数)，详细了解可以查看[reference 6](https://en.wikipedia.org/wiki/Higher-order_function)

下面，我们来了解一下该如何test function for equality(比较function).首先，我们了解一下通常比较的几种方式：
- Extensional equality: 如果要使两个function `f`和`g`被认为extensional equality,只要他们对于all inputs都有相同的outputs(`∀x. f(x) = g(x)`)。举个例子，两个stable sorting算法的实现有很多种(比如insertion sort和merge sort)，不同sort算法的实现会被认为extensional equality因为对于所有的input都有相同的output.这对于语言实现来讲非常难，所以没有编程语言支持该特性。
- Intensional equality： 只要两个function有相同的internal structure，那么这两个function就被认为Intensional equality.比如Lisp1.5就在interpreted阶段实现了该特性。
- Reference equality: 这是目前大部分语言支持的equality。在这些语言当中，所有的function都会被赋值一个unique identifier(通常是function body的内存地址)，而两个function的比较就是对应的identifier的比较，只要两个identifier指向同一块内存地址，那么久认为两个function是相等的。

总结来说，这3种比较方式的比较严格程度依次增强.：extensional equality < intensional equality < reference equaltiy.

讲了这么多，我们看看目前编程语言对于first-class function的支持(图表更加直观)：
- functional programming languages: 比如 Scheme， ML, Haskell, Scala都从语言层面上支持first-class functions. 在最早的functional language - lisp被创造的时候，很多关于first-class function的概念还没有被很合理的理解，这也导致lisp在早期实现first-class function是通过dynamically scoped来实现的。但是在后来的Common Lisp和scheme这两个dialect当中已经通过lexically scoped来实现first-class functions了.
- 很多scripting language，比如Perl, Python, PHP, Lua,Javascript都支持first-class function.
- 对于imperative languages，主要分为两个家族，Algol family和C family。具体区别看下面的图表。
![first-class function](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/181220-first-class-function.png)
 
### non-local variable 和 closure
> closure(闭包): 简单来讲，在编程语言当中, a closure (also lexical closure or function closure) 是指用来实现lexically scoped name binding(由first-class function造成)的一种技术.

>non-local variable(free variable): 指该variable既不是定义在local scope当中，也不是global scope中，通常出现在nested or anonymous functions当中。

下面我们通过简单的javascript代码来直观看一下：
```javascript
var f, g;
function foo() {
  var x;
  f = function() { return ++x; };
  g = function() { return --x; };
  x = 1;
  alert('inside foo, call to f(): ' + f());
}
foo();  // 2
alert('call to g(): ' + g());  // 1 (--x)
alert('call to g(): ' + g());  // 0 (--x)
alert('call to f(): ' + f());  // 1 (++x)
alert('call to f(): ' + f());  // 2 (++x)
```
很明显，我们从上面代码中可以看出，f和g都是nested function.而两个function都在各自内部依赖外部lexical scope的变量`x`,这就形成了closure。而这段代码中的free variable就是`x`.

### stack-based memory allocation 和 heap-based memory allocation
> stack-based memory allocation: 首先，我们知道stack是一种LIFO(last in first out)的data structure, 而stack-based memory allocation就是指基于stack的一种内存分配方式。

> heap: 在CS领域当中，heap指的是专用的 tree-based data structure 。

我们现在从computing architecture层面上来看stack，stack就是指的memory的部分区域。在大部分现代操作系统当中，每一个thread都有一块memory被用作stack。当function 被执行时，相关的环境会被压入stack当中，执行完该调用后在pop出来。因为stack-based memory allocation的执行方式是LIFO，因此它非常简单，也就导致存取速度快于heap-based memory allocation。

**note**: 在很多small CPU当中，每个 thread被分配的 stack size很少，可能只有几bytes. 我们在开发过程当中，分配太多的stack memory可能会导致stack overflow.
**note**：一些processer families,比如X86，会有一些特殊的命令来让我们操作executing thread的stack。另外一些其他的processor families，比如PowerPC,不支持直接的stack support，而是通过delegate stack management来处理。

### tail call 和 tail recursion
> tail call: 在CS领域当中，我们可以将程序划分成一个个procedure,如果在这个procedure的最后一个action当中执行的是subroutine call,那么我们可以称之为tail call.

> tail recursion: 如果tail call在call chain当中都是进行的相同的subroutine，那么这种情况就被称为tail recursion，也就是尾递归。理论上尾递归只需要在call stack当中包含一个stack frame就足够了。

## Funarg problem
> funarg problem:在CS领域当中，funarg problem指的是在编程语言的实现当中, 如何来实现first-class functions这个特性。实现该特性的难点通常出现在nested functions当中，比如我们在nested function直接引用non-local variables。解决思路通常有两种, 从语言层面禁止nested function或者支持closure.

下面，我们简单看看语言当中是如何来实现该特性的。
- 在早期imperative languages: 不支持functions as result types(比如ALGOL 60 或者Pascal)；忽略 nested function(也就不会出现non-local variables)（比如C）。
- 早期functional language: Lisp采用dynamic scoping来实现该特性(non-local variable的值由运行时决定，而不是function被定义时决定).
- 更加合理的Lexically scoped first-class function被scheme引入。但这也导致语言需要处理references to functions,这意味着garbage collection需要成为一个必要的特性。


在CS当中，通常将funarg problem 分成 upwards funarg problem和downwards funarg problem.
- upwards funarg problem - 在function当中返回 function
- downwards funarg problem - 将function当成parameter(实参)传递给另一个function call.

### upwards funarg problem
首先，我们描绘一个简单的编程场景：
在很多的compiled language中，function的调用时通过call stack来实现的。
在一段程序执行中，我们在一个function(caller)执行当中调用了另外一个function(callee)。caller的local state(比如parameters和local variables)需要保存压入call stack当中，因为需要在callee执行完返回后来还原环境。在大部分compiled programs中，这些local state以stack frame(或者被叫做activation record)的数据结构存储。
以上面的情况来看，upwards funarg problem会出现是因为caller function需要依赖 callee function的当中的某些数据state，这就意味着callee在被调用完成后不能被deallocated(而这就和stack-based function call paradigm冲突)。

那么，作为一个语言设计者，我们该如何解决上述的问题：
1. 对于stack frames采取 heap-based memory allocation,并且依赖Garbage collection或者reference counting来deallocate stack frames。这种方案的缺点在于效率，会很明显的降低性能。
2. 一些efficiency-minded compilers会采取一种Hybird的方案。通过 static program analysis来检测程序中的function有没有upwards funargs,如果没有则给该stack frame分配stack memory；否则，分配heap memory.
3. 另外一个解决方案：在closure刚被创建的时候，将涉及到的non-local variables的值复制到closure当中。但是我们需要考虑一种情况，如果non-local variable是mutable variable将会导致该variable在所有的closures当中共享，这可能并不一定使我们想要的。因此，如果这些variables是constant的话，该方案是可接受的。 **ML语言**采用了这种方案（因为该语言中variables直接绑定到values，variables是不可改变的）。另外**Java**在anonymous class当中也采取了该方案，只允许在enclosing scope当中依赖 final variables.
另外一些语言允许开发者显式的选择这两种行为。比如PHP5.3当中，anonymous functions 可以在closure当中使用 `use() clause`来显式选择。比如Apple的 blocks anonymous functions 捕获variable默认是value，但是如果你想要共享state，你可以使用`__block` modifier来限定variable（该变量会被分配在heap上）。

### downwards funarg problem
首先，我们来简单的看一段javascript代码：
```javascript
let x = 10;
 
function foo() {
  console.log(x);
}
 
function bar(funArg) {
  let x = 20;
  funArg(); // 10, not 20!
}

// Pass `foo` as an argument to `bar`.
bar(foo);
```
对于函数 foo 来说，x 是自由变量。当 foo 函数被激活时（通过
funArg 参数） - 程序应该在哪里解析 x 的绑定？是创建函数的外部作用域还是调用函数的调用者作用域？正如我们所见，调用者即 bar 函数，也提供了 x 的绑定 - 值为 20. 
抽象来讲，downwards funarg problem就是指我们该如何确定绑定的正确环境：应该是创建时的环境，还是调用时的环境（dynamic scope还是lexical scope）.
通常对于大部分现代编程语言，都是lexical scope，也就是创建时的环境。并且downwards funarg problem会复杂化tail recursion的编译效率。

## Conclusion
综上所述，我们基本能够理解funarg problem是什么问题，有哪些解决办法。理解这些基本CS概念能够帮我们更好地理解各个编程语言（比如javascript、python、java等等）。

## Reference
1.[Wiki, Funarg problem](https://en.wikipedia.org/wiki/Funarg_problem)
2.[first-class citizen](https://en.wikipedia.org/wiki/First-class_citizen)
3.[first-class function](https://en.wikipedia.org/wiki/First-class_function)
4.[Stack-based memory allocation](https://en.wikipedia.org/wiki/Stack-based_memory_allocation)
5.[tail call](https://en.wikipedia.org/wiki/Tail_call)
6.[higher-order function](https://en.wikipedia.org/wiki/Higher-order_function)