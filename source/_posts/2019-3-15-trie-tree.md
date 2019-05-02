---
title: 算法之trie tree
date: 2019/3/25 15:13:07
cover: 	https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree.png
author:
  nick: BruceYJ
  link: https://www.github.com/BruceYuj
editor:
  name: BruceYJ
  link: https://www.github.com/BruceYuj
subtitle: trie tree
tags:
- Algorithm

categories:
- Algorithm
---
<!-- toc -->

---
## introduction
最近在做ASTR，而algorithm是该项目的task 1. 所以开始重新刷LeetCode题。相比其他ACM题库而言，LeetCode的难度属于初、中级，更加的偏向于职场，而弱化了一些高级数学相关的东西。

这段话是我对于刷LeetCode的一些看法。很多人觉得算法对于实际工作过程中的用途不大，有点类似于“面试造火箭，工作拧螺丝”。但是真的用途不大吗？我觉得并非如此，如果你能够发现里面的数学之美的话。其一，一个优秀的算法可以极大的提高你的程序运行效率，尤其在某些极端情况下面；其二，学习了这些算法，可以极大的提高我们的逻辑思维；其三，这对于面试还是有很多好处的。

我做LeetCode的方法是：
1. 选取某一到特定的题目（一般是净点赞多的优先）
2. 做题，并研究其抽象出来的原理
3. 举一反三运用于这一类题

**Note**： 特别强调，刷题不是目的，会做某一到具体的题更不是完成结果。学会每道题背后的原理，并能够理解、完成所有的这一类问题，以及将其运用于自己的工作当中，这才是我们真正需要达到的目标。

举个例子：
1. 选取题目leetcode 5:  Longest Palindromic Substring
2. 由该题目联想到经典的两类问题： longest common substring problem 和 longest common subsequence problem。然后，联系到longest common substring problem的一般解决思路，generalized suffix tree，在细化到 trie tree。
3. 搜索相关的问题，并解决。

## Trie tree
首先*Trie* 来自于单词*retrieval*, 通常发音为 /ˈtraɪ/ (as "try").
> In computer science, a trie, also called digital tree, radix tree or prefix tree, is a kind of search tree—an ordered tree data structure used to store a dynamic set or associative array where the keys are usually strings. 

Trie tree，又被称为字典树或前缀树。从名字我们可以推断出其可以被用来查找字符串。

我们先来看一个例子：
1. 给定一个字符串集合`cat, cash, app, apple , aply, ok`，来构建一颗字典树， 如下图：

![Trie tree 1](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree-1.jpg)

由此我们引出字典树的特点：
1. Trie tree用边来表示字母
2. 有相同前缀的单词公用前缀节点。那么我们可以知道，在单词只包含小写字母的情况下，我们可以知道每个节点最多有26个子节点
3. 整棵树的根节点是空的
4. 每个单词结束的时候用特殊字符表示(比如上图的$)，在代码中可以单独建立一个bool字段来表示是否是单词结束处

### 基本操作
最简单的两个操作为: **insert** 和 **search**

1. insert: 插入一个新单词
> 从图中可以直观看出来，从左到右扫描新单词，如果字母在相应根节点下没有出现过，就插入这个字母；否则沿着字典树往下走，看单词的下一个字母。

问题1： 字母往哪个位置插？
有两种编码方式。第一种可以按照输入顺序对其进行编码，这里相同字母的编码可能不同：
![trie tree 2](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree-2.jpg)

第二种编码方式:
因为每个节点最多26个子节点，我可以可以按他们的字典序0-25编号，这里相同字母的编码相同
![enter description here](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree-3.jpg)

通常来讲，我们来实现这个数据结构会有两种方式：
- 数组模拟
- 类的形式

通常，虽然第二种方式更加的浪费空间，但是我会更加的喜欢用第二种方式。比如在处理下面这几个问题时更加方便：
- 查询某个单词是否存在字典树中。我们只需要在节点中添加属性表示即可。
- 查询某个前缀出现的次数。我们仍然只需要在节点中添加属性即可。


因此，我们来看实际的代码(javascript版):
```javascript
  var TrieNode = function() {
    this.isEnd = false;
    this.links = new Array(26);
}
TrieNode.prototype.containsKey = function(ch) { // 当前节点的子节点中是否包含该字符
    return this.links[ch.charCodeAt(0) - 'a'.charCodeAt(0)] !== undefined; 
}
TrieNode.prototype.get = function(ch) { // 获取当前节点相关字符的子节点
    return this.links[ch.charCodeAt(0) - 'a'.charCodeAt(0)];
}
TrieNode.prototype.put = function(ch, node) { // 插入当前相关字符的子节点
    this.links[ch.charCodeAt(0) - 'a'.charCodeAt(0)] = node;
}
TrieNode.prototype.setEnd = function() { // 设置当前节点是否为单词结尾
    this.isEnd =true
}

/**
 * Initialize your data structure here.
 */
var Trie = function() {
   this.root = new TrieNode();
};

/**
 * Inserts a word into the trie. 
 * @param {string} word
 * @return {void}
 */
Trie.prototype.insert = function(word) {
    let node = this.root;
    for (let i = 0; i < word.length; i++) {
        let currentChar = word[i];
        if(!node.containsKey(currentChar)) {
            node.put(currentChar, new TrieNode());
        }
        node = node.get(currentChar);
    }
    node.setEnd();
};

/**
 * Returns if the word is in the trie. 
 * @param {string} word
 * @return {boolean}
 */
Trie.prototype.search = function(word) {
    let node = this.root;
    for (let i = 0; i < word.length; i++) {
        let currentChar = word[i];
        if(node.containsKey(currentChar)) {
            node = node.get(currentChar);
        } else {
            return false;
        }
    }
    return node.isEnd();
};

/**
 * Returns if there is any word in the trie that starts with the given prefix. 
 * @param {string} prefix
 * @return {boolean}
 */
Trie.prototype.startsWith = function(prefix) {
    let node = this.root;
    for (let i = 0; i < word.length; i++) {
        let currentChar = word[i];
        if(node.containsKey(currentChar)) {
            node = node.get(currentChar);
        } else {
            return false;
        }
    }
    return true;    
};

/** 

 */
  var obj = new Trie();
  let words = ["Trie","insert","search","search","startsWith","insert","search"]
  for (let i = 0; i < words.length; i++ ) {
    obj.insert(words[i]);
  }
```
很明显，上面的写法比较偏向于工程化，比较完整类型的，上面的代码可以更加的优化,我们用对象来模拟：
```javascript
var Trie = function() {
   this.root = {};
};

/**
 * Inserts a word into the trie. 
 * @param {string} word
 * @return {void}
 */
Trie.prototype.insert = function(word) {
    let node = this.root;
    for (let ch of word) {
      if (!(ch in node)) node[ch] = {}
      node = node[ch]
    }
    node['$'] = true // 表示单词结束位置
};

/**
 * Returns if the word is in the trie. 
 * @param {string} word
 * @return {boolean}
 */
Trie.prototype.search = function(word) {
    let node = this.root;
    for(let ch of word) {
      if(ch in node) node = node[ch]
      else return false
    }
    return node['$'] === true;
};

/**
 * Returns if there is any word in the trie that starts with the given prefix. 
 * @param {string} prefix
 * @return {boolean}
 */
Trie.prototype.startsWith = function(prefix) {
  let node = this.root;
    for(let ch of prefix) {
      if(ch in node) node = node[ch]
      else return false
    }
    return true; 
};
```
我们来看看两段代码的运行效率：
![execute result](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree-result.png)

### 实际应用
我们下面来看看实际开发中trie tree的运用：
1. autocomplete
![enter description here](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree-208_GoogleSuggest.png)

2. spell checker
![enter description here](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree-208_SpellCheck.png)

3. IP routing(longest prefix matching)
![enter description here](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree-208_IPRouting.gif)

4. T9 predictive text
![enter description here](https://myblog-1257043911.cos.ap-chengdu.myqcloud.com/posts/190325-trie-tree-208_T9.jpg)

## reference
1. [Trie, wiki](https://en.wikipedia.org/wiki/Trie)
2. [leetcode 208. Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/submissions/)
