---
title: 浅谈js内存泄漏
date: 2019-06-26 16:15:52
tags: [js内存泄漏, 内存泄漏, 垃圾回收]
---

内存泄露是每个开发者最终都要面对的问题，它是许多问题的根源：反应迟缓，崩溃，高延迟，以及其他应用问题。

<!--more-->
### 什么是内存泄漏

本质上，内存泄露可以定义为：应用程序不再需要占用内存的时候，由于某些原因，内存没有被操作系统或可用内存池回收。
简单来说就是不再用道德内存，没有及时释放。

要了解js内存泄漏，首先我们需要了解一下js的内存管理机制，即js的垃圾回收机制。

### Mark-and-sweep

大部分垃圾回收语言用的算法称之为Mark-and-sweep。算法由以下几步组成。

1.垃圾回收器创建一个“roots”列表。Roots通常是代码中全局变量的引用。js中，“window”对象就会一个全局变量，被当做root。window对象总是存在，因此垃圾回收器可以检查它和它的所有子对象是否存在，即检查该对象是否垃圾。

2.所有的roots被检查和标记为激活（既不是垃圾）。所有的子对象也被递归地检查，从root开始的所有对象如果是可大的，他就不被当作垃圾。

3.所有未被标记的内存都会被当作垃圾，收集器就释放内存，归还给操作系统。

现在的垃圾回收期改良了算法，但本质是相同的。可达内存被标记，其余的被当做垃圾回收。



### 垃圾回收机制

js具有自动垃圾回收机制，也就是说，执行环境负责管理代码执行过程中使用到的内存。所需内存的分配以及无用内存的释放完全实现了自动管理

js的垃圾回收机制就是找出不再使用的变量，将其清除，释放其占用的内存。当然这个回收不是时刻存在的，是有固定的时间间隔的。

js中最常用的垃圾回收方式有两种：标记清除和引用计数。

#### 1. 标记清除
这是javascript中最常用的垃圾回收方式。当变量进入环境时，例如在函数中声明一个变量，就将这个变量标记为“进入环境”。当变量离开环境时，再将其标记为“离开环境”。之后执行后，将其清除。

从js回收机制来说，永远不会释放进入执行环境的变量所占用的内存，因为只要执行流进入相应的环境，就有可能会用到它们。

```
function func() {
  var a = 1;//被标记进入环境
  var b = 2;//被标记进入幻境
}
func();//执行完毕之后，变量a,b被标记离开环境，被回收
```
如果我们代码写法不当，会使变量一直处于“进入环境”的状态，无法被回收

#### 2. 引用计数
另一种不太常见的垃圾回收策略是引用计数。引用计数的含义是跟踪记录每个值被引用的次数。当声明了一个变量并将一个引用类型赋值给该变量时，则这个值的引用次数就是1。相反，如果包含对这个值引用的变量又取得了另外一个值，则这个值的引用次数就减1。当这个引用次数变成0时，则说明没有办法再访问这个值了，因而就可以将其所占的内存空间给收回来。这样，垃圾收集器下次再运行时，它就会释放那些引用次数为0的值所占的内存。

```
function test(){
 var a = {} ; //a的引用次数为0 
 var b = a ; //a的引用次数加1，为1 
 var c = a; //a的引用次数再加1，为2
 var b ={}; //a的引用次数减1，为1
}
```

只要引用次数不为0，垃圾收集器就无法释放该变量占用的内存


### 导致内存泄漏的几种情况

#### 1.意外的全局变量

```
function leaks(){  
    leak = 'xxxxxx';//leak 成为一个全局变量，不会被回收
}
```
调用完函数以后，变量仍然存在,导致泄漏。
在js文件头部加上 ‘use strict’ 启用严格模式来避免这类问题, 严格模式会阻止你创建意外的全局变量。

#### 2.被遗忘的计时器或回调

```
var someResource = getData();
setInterval(function() {
    var node = document.getElementById('Node');
    if(node) {
        node.innerHTML = JSON.stringify(someResource));
    }
}, 1000);
```

如果后续id为Node的节点被移除了，定时器里的node变量仍然持有其引用，导致游离的DOM子树无法释放。同时, 因为计时器回调函数中包含对 someResource 的引用, 定时器外面的 someResource 也不会被释放.

当不需要setInterval或者setTimeout时，定时器没有被clear，定时器的回调函数以及内部依赖的变量都不能被回收，造成内存泄漏

#### 3.未清除 dom 元素的引用

在js中，我们对于经常需要访问的dom节点一般都是将其缓存下来。

```
var elements = { 
    button: document.getElementById('button'), 
    image: document.getElementById('image'), 
    text: document.getElementById('text') 
}; 
 
function doStuff() { 
    image.src = 'http://some.url/image'; 
    button.click(); 
    console.log(text.innerHTML); 
    // 更多逻辑 
} 
 
function removeButton() { 
    // 按钮是 body 的后代元素 
    document.body.removeChild(document.getElementById('button')); 
 
    // 此时，仍旧存在一个全局的 #button 的引用 
    // elements 字典。button 元素仍旧在内存中，不能被 GC 回收。 
} 
```

此外还要考虑 DOM 树内部或子节点的引用问题。假如你的 JavaScript 代码中保存了表格某一个 <td> 的引用。将来决定删除整个表格的时候，直觉认为 GC 会回收除了已保存的 <td> 以外的其它节点。实际情况并非如此：此<td> 是表格的子节点，子元素与父元素是引用关系。由于代码保留了 <td> 的引用，导致整个表格仍待在内存中。保存 DOM 元素引用的时候，要小心谨慎

#### 4.闭包
JavaScript开发的一个关键方面是闭包。闭包是一个内部函数，可以访问外部（封闭）函数的变量。由于JavaScript运行时的实现细节，可能存在以下形式泄漏内存：

```
var theThing = null;
var replaceThing = function(){
  var originalThing = theThing; 
  var unused = function(){ 
    //对'originalThing'的引用
    if(originalThing) {
      console.log("hi"); 
    }
  };

  theThing = { 
    longStr: new Array(1000000).join（'*'）,
    someMethod: function(){ 
      console.log（"message"）; 
    } 
  }; 
};
setInterval(replaceThing，1000);

```
一旦replaceThing被调用，theThing会获取由一个大数组和一个新的闭包（someMethod）组成的新对象。然而，originalThing会被unused变量所持有的闭包所引用（这是theThing从以前的调用变量replaceThing）。需要记住的是，一旦在同一父作用域中为闭包创建了闭包的作用域，作用域就被共享了。

在这种情况下，闭包创建的范围会将someMethod共享给unused。然而，unused有一个originalThing引用。即使unused从未使用过，someMethod 也可以通过theThing在整个范围之外使用replaceThing。而且someMethod通过unused共享了闭包范围，unused必须引用originalThing以便使其它保持活跃（两封闭之间的整个共享范围）。这就阻止了它被收集。

所有这些都可能导致相当大的内存泄漏。当上面的代码片段一遍又一遍地运行时，你会看到内存使用率的不断上升。当垃圾收集器运行时，其内存大小不会缩小。这种情况会创建一个闭包的链表，并且每个闭包范围都带有对大数组的间接引用。

#### 5.循环引用

循环引用 在引用计数策略下会导致内存泄漏，标记清除不会。 
```
function fn() {
 var a = {};
 var b = {};
 a.pro = b;
 b.pro = a;
} 
fn();
```

a和b的引用次数都是2，fn()执行完毕后，两个对象都已经离开环境。 
在标记清除方式下是没有问题的，但是在引用计数策略下，a和b的引用次数不为0，不会被垃圾回收器回收内存。如果fn函数被大量调用，就会造成内存泄漏。


参考资料：JavaScript垃圾回收机制 <https://www.cnblogs.com/zhwl/p/4664604.html>
          &#8195;&#8195;&#8195;&#8195;&#8195;JavaScript内存泄露的4种方式及如何避免 <https://blog.csdn.net/qappleh/article/details/80337630>






