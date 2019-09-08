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

