---
title: Vue组件-右键菜单
date: 2019-08-21 22:32:56
tags: vue
---

Vue组件-右键菜单vue-contextmenu-json
<!-- more -->
## Vue-Contextmenu-json
基于Vue的右键菜单组件，最开始开发的主要目的是想通过json数据来创建菜单，而不是再在对应的页面中写右键菜单的template模板，所有的工作只需要一个json对象和绑定到元素的指令。相关的Github主页是 https://github.com/WongYAQi/vue-contextmenu-json

## Usage
npm安装

```javascript
npm install vue-contextmenu-json
```

in main.js
```javascript
import VueContextMenu from 'vue-contextmenu-json';
Vue.use(VueContextmenu);
```

in *.vue
```javascript
<template>
    <div>
        <anyElement v-contextmenu='{array:theArrayData,id:theIdYouWant}'></anyElement>
    </div>
</template>

<script>
    export default {
        data () {
            return {
                theArrayData:[
                    {name:'Operation 1',function:'funca',child:null},
                    {name:'Operation 2',function:'funca',child:[
                        {name:'Operation 2-1',function:'funca'}
                    ]}
                ]
            }
        },
        methods:{
            funca () {
                console.log(1);
            }
        }
    }
</script>
```

参数Parameter

|name|type|content|
|-|-|-|
|array|Array|json对象，用来生成菜单信息|
|id|String|菜单id，区分同一个页面中的多个菜单组件|

And the array parameter is

|name|type|content|
|-|-|-|
|name|String|右键菜单的显示名称|
|function|String|右键菜单操作，只需要传入一个methods中的函数名称，methods的函数具有this(指向当前vue实例)，nodeNow(指向右键点击时的DOM元素)，event（事件对象）|
|child|Array|多级菜单，参数和现在的一致|

## Example
![wEIACnURtSVmuP9.gif](https://i.loli.net/2019/09/03/zbHwB7fgqG2IuQP.gif)

```javascript
<template>
    <div>
        <ul v-contextmenu='{array:ctmArray,id:"c"}'>
            <li v-for='item in array' :key='item.name'>
                    {{item.name}}
            </li>
        </ul>
        <!--
        <button @click='comName = null'>button</button>
        <i-ctm  v-if='comName === "i-ctm"' :array='ctmArray'></i-ctm>
        -->
    </div>
</template>

<script>
import iCtm from './components/contextmenu/index.js'
export default {
    name:'app',
    components:{
        'i-ctm':iCtm
    },
    data () {
        return {
            comName:'i-ctm',
            array:[
                {name:'Li 1'},
                {name:'Li 2'}
            ],
            ctmArray:[
                {name:'button 1',function:'funca',child:[
                    {name:'1-1111111',function:'funca',child:[
                        {name:'1-1111-11111',function:'funca'}
                    ]}
                ]},
                {name:'button 2',function:'funca'},
                {name:'button 3',function:'funca'},
                {name:'button 4',function:'funca'}
            ]
        }
    },
    methods:{
        funca (node) {
            this.array.push({name:'Li 3'});
            console.log(node); //  the element you open contextmenu, like 'Li 3' li
        }
    }
}
</script>
```

## End
If you have any problem or thoughts you think will make this component better,please let me know .