---
title: 每日一问·虚拟DOM是如何实现的
date: 2019-09-14 19:45:24
tags: [每日一问,Javascript]
---

虚拟DOM ，Virtual DOM是用于减少DOM操作过程中的性能消耗。那么VDOM是如何进行渲染与DIFF的呢？
<!-- more -->

在学习Vue框架的时候，必不可少的接触到了虚拟DOM的内容，对于虚拟DOM，相信每个人都能说出一点出来

- Vue响应式框架中用到了  
- 为了减少DOM消耗  
- 用JS对象替换DOM元素操作  

但是，他是怎么实现的呢？为什么可以实现从对象到html的渲染？是如何识别其中的内容的？  
接下来，用snabbdom.js进行分析

# Snabbdom.js
Snabbdom是Vue开发虚拟DOM的基础，整体是以**TypeScript**为基础开发，Snabbdom是偏底层的框架，它并不是只能被使用在Vue中，还有其他很多的项目用到了他，而Vue的话，还要考虑如何将响应式对应，指令加入到虚拟DOM中进行渲染。  

## 项目结构
```
--helpers  
----attachto.ts   
--modules  
----attributes.ts  
----class.ts  
----dataset.ts  
----eventlisteners.ts  
----hero.ts  
----module.ts // 钩子函数模块  
----props.ts  
----style.ts  
--h.ts  
--hooks.ts  
--htmldomapi.ts  
--is.ts  
--snabbdom.bundle.ts  
--snabbdom.ts  
--thunk.ts  
--tovnode.ts  
--vnode.ts  // 定义vnode,vnodedata结构
```

## 核心功能
### init()
Snabbdom要求先运行init函数进行初始化，提供list数组，最终返回一个patch方法用来进行渲染。  
那么Init方法主要做了什么呢？  

1. 声明了一堆函数  
2. 声明了一个hooks钩子数组对象，其中hooks与传入的Modules数组中的模块钩子函数进行比较。
3. 返回patch方法

那么我就有一个问题了  
> Vue的钩子函数是基于Snabbdom开发的吗？  

### h.ts
继续j看h.ts，因为Snabbdom.js就是以h()方法来生成VNODE的。  
```javascript
var vnode = h('div', {style: {color: '#000'}}, [
  h('h1', 'Headline'),
  h('p', 'A paragraph'),
]);
```
vnode可以用多种方式来生成  
```
h('div')  
h('div',{属性对象})
h('div',{},[后代数组])
function h(sel,b?,c?){...}
```
所以为h函数设置了重载，第二第三个参数可以为undefined  
首先对第三个参数进行判断是否存在，存在则也要判断具体的类型，再判断第二个，SVG会另外运行，其他的情况下，就会直接得到一个vnode对象
h方法就结束了

#### vnode.ts
vnode进行了Vnode和VnodeData对象的封装，同时返回了生成vnode的函数
```javascript
export interface VNode {
  sel: string | undefined; //目标元素
  data: VNodeData | undefined; // 属性信息
  children: Array<VNode | string> | undefined; // 后代信息
  elm: Node | undefined; // 节点信息，应该是真实DOM
  text: string | undefined;
  key: Key | undefined;
}

export function vnode(sel: string | undefined,
                      data: any | undefined,
                      children: Array<VNode | string> | undefined,
                      text: string | undefined,
                      elm: Element | Text | undefined): VNode {
  let key = data === undefined ? undefined : data.key;
  return {sel, data, children, text, elm, key};
}
```

### patch
在官网实例中，接下来进行patch，就可以将目标元素与虚拟DOM对象进行替换了。`patch(dom,vnode)`，所以直接进入snabbdom.ts寻找patch方法  
```javascript
patch(document.getElementById(''),vnode)
```
1. 判断**cbs**钩子函数中是否存在preHook钩子，如果有的话，会预先执行  
2. 判断传入的第一个参数是否是vnode，如果不是（Element）则生成一个vnode替换  
3. 比较新旧vnode是否相同，如果相同则直接渲染，如果不同则处理并把新节点（Element）放到旧节点的位置  
4. 执行新增节点队列的InsertHook钩子函数
5. 处理**cbs**中的PostHook钩子函数（如果有）

#### 处理ProHook钩子函数
在官网的教程中，这里传入了Style和Css的模块，可以理解为进行样式方面的初始化，预处理。  
```javascript
export const styleModule = {
  pre: forceReflow,
  create: updateStyle,
  update: updateStyle,
  destroy: applyDestroyStyle,
  remove: applyRemoveStyle
} as Module;

export const classModule = {create: updateClass, update: updateClass} as Module;
export default classModule;
```
可以看到，只有Style模块具有Pre钩子函数。其中钩子的作用就是`设置reflowForced变量为false`

#### 替换vnode
这一步主要是判断要替换的旧节点，是否是vnode虚拟节点。
> vue也有判断、替换的过程，它相比Snabbdom，携带了哪些更多的内容？  

对于Element节点，根据标签Tag，Id，ClassName来生成vnode，key值是undefined
比较的依据是key与sel的全等
> key === new.key && sel === new.sel

这种情况下，与vnode比较是肯定不同的

#### 比较新旧vnode
在不同vnode的情况下：  
主要内容就是`生成新Element节点`和`移除旧Element节点`  
在生成新Element节点的过程中，感觉也没有什么特别的地方，就是根据vnode.sel来决定元素的标签，id，class。然后递归转换children，其中对各种节点都进行判断，Comment，Text，Element，Ns。最后对`create钩子函数`做处理，如果vnode有对应的钩子函数，就添加vnode到队列中。等到所有的元素都生成完毕后，进行统一的CreateHook调用。

而对于相同vnode，就是一个Patch的过程：
> PS : 感觉相同的情况下还复杂点耶  

首先进行PrePatch钩子函数的处理（如果有）  
<span id="Branch1">Branch 1</span> 然后判断新vnode是否存在text
> 话说这个text是什么阿？看vnode定义，很少有第四个参数，不对。在h()中，text是第2，第3参数为Primate类型的情况下才会存在。如果没有text，说明当时h()方法中，不存在Primate类型的参数。  

然后进入一个重要的判断，当新旧vnode都有Children节点，并且CHildren节点还不相同的时候，进行更新节点操作，这个地方就是重中之重  
> 旧Vnode的Children是怎么来的？就是说简单的例子，用div+getElementById是触发不到这里的。只能是两个vnode进行Patch。

**如何进行后代元素的比较更新？**  
![Diff算法.png](https://i.loli.net/2019/09/15/N7XLjRSAatc5W93.png)  

![Diff算法实例.png](https://i.loli.net/2019/09/15/XMNq9OywRzmU7H1.png)  


如果只存在当前vnode的Children对象，则在旧vnode.text存在的情况下，设置当前节点的text为空，然后填充新vnode的后代到当前元素
```
伪代码
if(isDef(oldVnode.text))  set ele.text = '';
ele.insertBefore(..) // 后代通过insertBefore填充到ele中
```

反之，如果只定义了oldVnode的Children对象，则需要将ele中所有之前的子元素都清空，反正就是考虑如何删除element的后代和调用Destroy钩子    
```
伪代码 removeVnodes
invokeDestroy() : children + children.children(!= text)  

cbs.remove() , data.hooks.remove(),in last => ele.removeChild(children.element)
or
children.sel == null => children is Node text => ele.removeCHild(children.element)
```

如果都没有后代，oldVnode.text不为空，那么旧把当前ele的text设置为空
> 等到什么时候再写入text？

在 [Branch1](#Branch1)，如果vnode有text属性，那么说明现在只需要渲染成一个TextNode，所以在之前存在CHildren的情况下清空
```
伪代码
if(vnode.text !== oldVnode.text){
   if(oldValue.children exists) removeChild(oldVnode.Children) 
   set element.text = vnode.text
}
```

最后执行`PostPatch钩子函数`

# 收获
1. 在粗略看完Snabbdom的源码后，了解了Diff算法的规则。  
2. 了解到一种新的赋值并判断的方式
> if((i = data.hook) != undefined && (i = i.pre) != undefined) {...}