---
title: 每日一问·Javascript中的事件类型(2)
date: 2019-09-08 10:48:32
tags: [每日一问,javascript]
---

写这边博文的契机，是在阅读Sortable.js源码的时候，发现了他们对dragable、pointenter事件的使用，还有事件的创建等等，这些都是以往没有接触过的，只使用了click，并没有深入理解事件的类型等。事件远不止click。本文主要介绍了各种事件类型。
【未完】
<!-- more -->

# 开始
一提到javascript中的事件类型，大家第一时间想到的肯定是click，而click属性MouseEvent类，其中除了click还具有哪些事件呢？除了MouseEvent鼠标事件，还有哪些事件大类呢？

## Event类
Event类是一个抽象的类。代表的是所有发生在DOM中的事件。下面派生了很多的子类。可以从下图中查阅。接下来我们讲一下最常用的几个事件类型。
![微信图片_20190908112541.png](https://i.loli.net/2019/09/08/te97bkqNoEX2AFh.png)


### 相关属性


## MouseEvent 
鼠标事件，此事件都是与鼠标有关的。常见一点的click，contextmenu，dbclick都是属于这个事件类型。

### 创建事件
可以通过手动创建事件，进行触发。当然，更多的我们是直接操作click,dbclick这些已经定义好的事件。
```javascript
document.getElementById('').addEventListener('click',event=>{
    // 操作
})
let evt = new MouseEvent('eventname',{
    // 可选项
    screenX: ..
    screenY: ..
})
doc.addEventListener('eventname',event=>{
    // ..操作  
})
doc.dispatchEvent(evt)

```

> Vue的框架中事件相关的内容是不是使用了自定义事件？

### 相关属性
那么MouseEvent有哪些属性呢？  
**screenX,screenY**  
这是经常使用的属性，代表相对于整个电脑屏幕而言的距离，screen就是屏幕的意思  

**clientX,clientY**  
这个属性也是经常用到的，client表示的是客户端，他们是用来表示这个事件触发的时候，相对于浏览器这个客户端工具的定位距离。  
实战中，这两个属性经常被用来做右键菜单的定位。  

**altKey,ctrlKey,metaKey,shiftKey**  
这个属性表示，当鼠标被按下的同时，`alt`，`ctrl`，`meta`，`shiftKey`键是否也被按下了。  

**button,buttons,which**  
button表示当前触发鼠标事件的案件，是哪一个。比如我们通过mouseup事件，每一个鼠标按键都会触发mouseup事件。  
|值|含义|
|-|-|
|0|主要按键（右手鼠标的左键）|
|1|辅助按键（钟健）|
|2|次要按键（右手鼠标的右键）|

而which属性也是能够表示当前触发事件的鼠标按键是哪一个，但是他和button所代表的含义有一点点差别，关于0的含义。  

要注意，有的鼠标还有浏览器的“前进”，“后退”按键哦~~  

> PS:click表示鼠标主要按键（一般是左键）被按下  

而buttons表示当前哪些按键被按下了？通过给定的数字的和来表示，就像Linux的755一样。  

**movementX,movementY**  
感觉这两个属性没啥用啊，代表mousemove事件中，相对于上一次事件的鼠标的偏移量。那么就有一个问题了，这个上一次事件是怎么定义的？浏览器是如何判断两次事件的间隔的？

> 浏览器如何判断mousemove事件的间隔？

**offsetX,offsetY**  
offset翻译为偏移量，表示的是当前事件触发的指针位置，相对于触发元素的偏移量。如果这其中有事件委托，那么相对的元素位置也会发生变化（大部分情况）  

**pageX,pageY**  
page是文档的意思，代表当前鼠标事件相对于文档的定位。  
为什么有这个属性呢，因为出现滚动的时候，screenX和clientX可能是不变的。他们都是相对于浏览器，屏幕而言的。这两个属性的参考系并没有因为我们拖动滚动条而发生改变。但是`offsetX`和`pageX`却改变了。所以要注意这个区别。  

### 标准事件
OK，现在属性的内容已经讲完了，接下来讲解下MouseEvent事件中，已经被定义好的鼠标事件。这里暂时不会贴代码，只是作为一个了解，如果哪天需要开发什么功能，你至少知道一点事件上的原理    
这一部分列举一下就好，因为很多都是经常用的，主要从名称上进行一个区分。  
**click,dbclick**  
最常用的点击事件，单击，双击  

**contextmenu**  
右键事件，经常被用来做右键菜单  

**mouse系列**  
mouseenter: enter，进入某个元素时触发  
mouseover: over，越过，鼠标进入元素时触发（包括进入子元素），但是在内部移动的时候时不会触发的。  
mousedown: down，按下，鼠标按键被按下时触发  
mouseup: up，升起，鼠标按键升起时触发  
mousemove: move，移动，鼠标在元素内部移动时触发  
mouseout: out，离开，鼠标离开元素，`或者`进入它的子元素时触发  
mouseleave：也是离开，但是是离开当前元素，与out要做区分  
其中move和over经常被弄混。。。惨。。。mouse系列的事件可以做很多事情，比如用mousemove来做拖动的效果，用mousemove+mousedown来做绘画  

**select**  
选择事件，这里指的是用鼠标对一段文本进行选择，选中的内容会出现反色现象。当选择完成后，就会触发select事件，想象一下富文本框中，对一段选中的文字进行字体变更，就至少要用这个事件进行处理。  

**wheel**  
本来我想分类一个滚轮事件，但是发现scroll其实被分为了视图事件（MDN）。wheel用来表示在元素内进行滚轮操作，常见的用来做方法缩小。他的接口是WheelEvent，是MouseEvent的子类。

## WheelEvent
鼠标滚轮事件，上下滑动滚轮。他是MouseEvent的子类。  
Event -> UIEvent -> MouseEvent -> WheelEvent  

### 相关属性
**deltaX,deltaY,deltaZ**  
只能说贫穷限制了我的想象力，deltaY好理解，就是在垂直方向上，滚动反映的一个数值。自己的鼠标测试是+-125。但是deltaX和deltaZ到底是什么鼠标啊啊啊啊！！！  
**deltaMode**  
指示当前的delta数据是什么单位格式的。  

## DragEvent
终于来到我的重头戏了，这篇博文最开始就是被这个事件影响了，Sortable.js插件就是以这一部分为基础的。以往没有关注过的东西，也在开始慢慢了解。拖动事件，用来表示拖动元素/文本的事件。所以除了`mousedown+mousemove`，我们还可以直接使用DragEvent事件来直接操作。  

> 首先注意一点，元素本身是不允许拖动的，必须为元素增加`draggable`属性，并设置为true字符串，才可以拖动元素！！！

TODO 接下来介绍相关的事件，以及属性

## ClipboardEvent
剪贴板事件，用来实现复制粘贴。还是说回富文本框，

## CompositionEvent
文本输入事件。继续富文本框

## 媒体事件
media,audio