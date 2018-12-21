---
title: ES6基础 - destructing
date: 2018/8/4 21:23:07
cover: https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/180804-ES6-Destructing.png
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: es6中的赋值解构
tags:
- ES6

categories:
- JavaScript
---
<!-- toc -->

## introduction
本文主要为ES6当中的destructing的基础知识，简单但是实用。

ES6允许我们按照一定的模式从数组和对象当中提取值，对变量进行复制，这个过程被称为destructing(解构)。
而在javascript当中，解构主要用于下面几种情况：
- 数组的解构赋值
- 对象的解构赋值
- 字符串的解构赋值
- 数值和布尔值的解构赋值
- 函数参数的解构赋值

以前我们给几个变量赋值:
```javascript
let a = 1;
let b = 2;
let c = 3;
```
而在ES6允许下面这种写法:
`let [a, b, c] = [1, 2, 3];`

上面的代码就是数组的解构赋值，本质上讲，这种写法叫做“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。

## 数组的解构赋值
```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["a", "b", "c"];
third // "c"

let [head, ..tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ["a"];
x // "a"
y // undefined
z // []
```
了解了正常情况下的解构，我们了解几种正常边界之外的情况：
```javascript
let [bar, foo] = [1]
bar // 1
foo // undefined
```
这种情况等号左边需要赋值的变量多于右边的值，这种情况并不会报语法错误，多余的变量会被赋予undefined。

另外一种情况：
```javascript
let [a, [b], d] = [1, [2, 3], 4]
a // 1
b // 2
d //4
```
这种情况属于不完全解构，左边的所有变量都有值。

**注意**： 只要某种数据结构具有Iterator接口，就可以采用数组的形式解构赋值。
```javascript
function* fibs() {
  let a = 0;
  let b = 1;
  white(true) {
   yield a;
   [a, b] = [b, a + b];
  }
}
let [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
```

### 默认值
destructing允许指定默认值，但是需要注意的是，只有当一个数组成员严格等于(`===`)undefined时，默认值才会生效
```javascript
let [x = 1] = [undefined]
x // 1

let [x = 1] = [null]
x // null
```

更加复杂一点：默认值可能是一个表达式，那么该表达式是惰性求值的
```javascript
function f() {
    console.log('a');
}
let [x = f()] = [1]; // 这里f函数不会执行
```

默认值可以引用解构赋值的其他变量，但是前提是该变量已经声明。
```javascript
let [x = 1, y = x] = [];  // x = 1; y =1;
let [x = y, y = 1] = []; // ReferenceError: y is not defined.

```

## 对象的解构赋值
数组的元素是有顺序的，而对象本身是没有顺序的。这也是对象的解构不同的地方，变量必须与属性同名，才能够取到值。
```javascript
let {foo, baz} = {foo: 'a', bar: 'b'}
foo // 'a'
baz /// undefined
```

但是有时候，我们要变量名和属性名不一样该怎么写？
```javascript
let {foo: baz} = {foo: 'a'}
baz // 'a'
```
也就是说，对象的destructing是先找到同名的属性，然后再赋给对应的变量。真正被赋值的是后者。
上面代码的foo是模式，而baz是变量。

**实际情况当中，我们的解构会比较的复杂，比如讲对象和数组结合**

另外，对象的解构也可以制定默认值：
```javascript
var {x = 3} = {}
x // 3
```
和数组相同的是，对象解构默认值生效的条件同样是属性值严格等于`undefined`.
```javascript
var {x = 3} = {x: undefined}
x // 3
```

下面来看一看边界值之外的情况：
```javascript
// Uncaught TypeError: Cannot destructure property `bar` of 'undefined' or 'null'.
let {foo: {bar}} = {baz: 'baz'};
```
这是由于`foo = undefined`现在，取undefined的子属性坑定会报错。

```javascript
let x;
{x} = {x : 1};
```
很明显，这里`{x}`会出现二义性，js引擎会把`{x}`读成代码块，所以会报`syntax error`
那么该如何修改呢？`({x} = {x:1};)`就可以了。

## 字符串的解构赋值
```javascript
const [a,b,c] = 'hello';
a // 'h'
b // 'e'
```
这里可以解构是因为字符串被转化为类似数组的对象。

```javascript
let {length : len} = 'hello';
len // 5
```
类似数组的对象都会有一个`length`属性。

## 数值和布尔值的解构赋值
解构赋值的规则是，只要等号右边的值不是数组或对象，就会先将其转为对象。但是由于`undefined`和`null`无法转为对象，所以对齐赋值解构会报错。
```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true

let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```

## 函数的参数的解构赋值
```javascript
function add([x, y]) {
    return x + y;
}

add([1, 2]) // 3
```
函数参数的解构也可以是使用默认值。
```javascript
function move ({x = 0, y = 0} = {}) {
    return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

但是，下面这种写法会出现不一样的结果：
```javascript
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```
`undefined`就会触发函数参数的默认值。

## 圆括号问题
解构赋值用起来会很方便，但是解析起来并不容易。对于编译器来讲，一个式子到底是模式还是表达式，是没有办法从一开始就知道的，必须等到解析到或解析不到等号才能知道。

由此，圆括号可能会带来歧义问题，ES6表示只要可能导致解构的歧义就不能使用圆括号。

因此，建议不要在模式中防止圆括号。

### 具体的几种情况
1. 变量声明语句不能使用
`let [(a)] = [1];`

2. 函数参数不能使用

```javascript
function f( [(z)] ) {return z;}
```
函数参数也属于变量声明

3. 赋值语句中的模式

`([a]) = [5];`

那么什么时候可以使用圆括号呢？
只有一种情况，赋值语句的非模式部分，可以使用
`[(a)] = [1]; // correct`

## 解构的实际开发用途
(1). 交换变量的值
```javascript
let x = 1;
let y = 2;
[x, y] = [y, x];

x // 2
y // 1

```

(2). 从函数返回多个值
我们知道函数只能返回一个值，如果我们希望返回多个值，只能将值放在数组或对象里面返回。使用destructing，可以很容易的取出这些值。
```javascript
// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

(3). 函数参数的定义
```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

(4). 快速提取JSON数据
```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

(5). 函数参数的默认值
```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```
和在函数体内部写`var foo = config.foo || 'default foo';`各有利弊。

(6). 遍历Map解构
可以很方便的获取键名和键值。
```javascript
const map = new Map();
map.set('first', 'hello');

for (let [key, value] of map) {
    console.log(key + "is" + value); 
} 
// first is hello
```

(7). 输入模块的指定方法
这个是模块化开发经常用到的加载方法

`cosnt {SourceNode} = require('source-map');`
## reference
1.[es6 destructing](http://es6.ruanyifeng.com/#docs/destructuring)

