---
title: Javascript中的事件类型(1)
date: 2019-09-04 20:06:42
tags: [javascript,每日一问]
---

写这边博文的契机，是在阅读Sortable.js源码的时候，发现了他们对dragable、pointenter事件的使用，还有事件的创建等等，这些都是以往没有接触过的，只使用了click，并没有深入理解事件的类型等。事件远不止click
<!-- more -->
说到Javascript的事件，我相信很多人的第一反应是onclick，onfocus，oncontextmenu。但是事实上，Javascript的事件种类远远超乎想象。

## 事件对象
事件对象的原始接口是`Event interface`，他们代表着会执行在DOM元素上的“事件”。  
Event对象的属性有：  

### 对象属性
**bubbles**  
bubbles是Boolean类型，指示事件是否可以冒泡，具体表现就是如果事件不可以冒泡，那我在执行ul-> li 的事件的时候，执行li的对应事件，不会触发ul的对应事件。

**cancelable**
cancelable是Boolean类型，指示事件是否可以取消，如果事件可以取消，那么就可以通过event.preventDefault()方法避免触发原事件，典型就是a标签。  

**currentTarget**  
**target**  
上面两个属性非常的容易混淆，让人迷惑。区分的话，主要从`Current`关键字入手，currentTarget代表当前触发事件方法的元素，而target代表我们人为、手动触发事件的元素。简单的说，target永远指向操作的源头,起源。    
```javascript
<ul id='0' >
  <li id='1'/>
  <li id='2'/>
</ul>
当我们触发li（2）的事件的时候，按照DOM事件流，最先触发的是li(2)的事件，然后冒泡，触发ul的相关事件。当触发li(2)的时候，currentTarget就是2，而target也是2。而冒泡触发0的时候，CurrentTarget是0，target是2.
```

**eventPhase**
eventPhase是一个无符号整型，简单的理解就是正数。在这里用来指示当前事件所处的事件流的代号。  

|值|事件流|
|-|-|
|1|捕获流|
|2|处理流|
|3|冒泡流|

相关的流，可以参考附图。这个图非常的详细。  

![eventflow _1_.png](https://i.loli.net/2019/09/04/2dLz4rGygmDTYWM.png)

**timeStamp**
DOMTimeStamp类型，一个时间戳，精确到毫秒级别，指示这个事件是什么时候生成的。有的系统可能不会有这个值，那么就返回0.    

**type**
type属性是一个DOMString类型，用来表示当前事件的类型是什么，比如click,contextmenu,dbclick。

### 对象方法
**initEvent**
event.initEvent(eventName,eventCanBubble,eventCanCel)方法，用来为Event实例初始化。虽然现在有很多方法被废弃了。
```javascript
var a = document.createEvent('Event');
a.initEvent('build',true,true);
```
通过这种方式来创建事件的相关属性，后续再进行事件的注册，触发。比如说click,dbclick这种事件都是浏览器已经注册完成了。

**preventDefault**
preventDefault()方法用来阻止事件的默认行为。这种方法只能作用在`cancelable`为true的事件上，为一个false的事件触发方法，不会有任何的结果。    
但是，什么叫做事件的默认行为呢？哪些行为是默认行为呢？  
举一个例子，checkbox的默认行为就是：点击后勾选和取消勾选。如果我们阻止了默认行为，那么就不会有勾选发生。  
再比如a标签，点击的默认行为就是跳转到href指定的路径。如果阻止了，就不会产生跳转的行为。  
但是要注意，这种阻止`不会影响冒泡行为`，该冒的泡那还是要冒的。
```javascript
<body>
    <div id='1'>
        1
        <input type='checkbox' id='2'></input>
        <a href='http://www.baidu.com' id='3'>百度</a>
    </div>
</body>
<script>
    var a = document.getElementById('3');
    a.addEventListener('click',event=>{
        event.preventDefault(); //不会触发a标签的导航
        //id(2)，则不会触发checkbox的勾选
    })
</script>
```

**stopPropagation**
stopPropagation()方法，~~~用来阻止冒泡~~~，用来阻止事件的继续执行。说冒泡是不准确的，我们甚至可以在捕获阶段就进行拦截。  


说完了事件对象的属性与方法，接下来说一说事件的创建。
## 事件创建
最古老的方式来创建一个事件
```javascript
var a = document.createEvent('Event');
a.initEvent('build',true,true);
var element = document.getElementById('1');
element.addEventListener('build',function(event){
    ...code
});
element.dispatchEvent(a);
```
通过这种方式我们实现了一个Event类型的事件，Event类型是最原始的事件接口，还有其他的事件接口，比如MouseEvent，UIEvent，这些都是Event接口的子类。

而现在最推荐的，主流的方式，是使用CustomEvent构造函数，这种方式创建了一个自定义的事件，仍然是以dispatchEvent的方式触发。
```javascript
var a = new CustomEvent('cat',{
    detail:{
        //detail属性代表什么？
        //代表事件触发时传入的数据信息
    },
    cancelable:true, //or false
    bubbles:true // or false
})
element.addEventListener('cat',function(event){
    console.log(event.detail);
});
element.dispatchEvent(a);
```

今天的内容到此为止，明天继续介绍事件的类型，主要进行Event,UIEvent,MouseEvent这种事件接口的介绍。