---
title: Javascript中的文件类型
date: 2019-10-14 22:40:44
tags: [每日一问,javascript]
---

Javascript中的文件类型，File
<!-- more -->

目前通过Html5操作文件，有两种方式，分别是input元素上传，或者通过拖动行为拖动
1. Javascript是可以进行文件上传行为的，通过*type类型的input元素*，在完成了对应交互后，可以在元素的`files`属性中，得到FileList集合，集合元素就是我们上传的文件信息。  
2. 拖动行为，必须在为一个区域添加`drop事件`，在drop事件中，通过调用`dataTransfer`的相关属性，可以得到文件列表，继续完成文件操作。  

> ps: 在《Javascript中的事件类型2》中有讲到拖动事件

> ps: 看到这里，应该可以看到几种用途：拖动上传文件区域

```javascript
class File{
    lastModified: Number, //最后修改时间，毫秒数
    lastModifiedDate: Date, // '最后修改时间的Date类型对象'
    name: String, //文件名
    size: Number, //文件大小
    type: MIME //类型
}
```

# 那么，我们得到了File对象，又该如何使用这些对象呢？

这里又要再介绍一个类型`window.URL`，它是一个DOM接口，实现了两个操作  
1. createObjectURL(file)  
2. revokeObjectURL(file)

他们分别用来创建和清除一个URL对象。*URL对象的作用，就是让你能够像一个URL字符串一样使用它。*

> "blob:http://localhost:8081/d6a6e952-5ddd-47fc-9b31-14e0d68051e2" 这就是一个URL对象

> 通过得到File对象后，生成URL对象，然后在Img元素的src属性上使用

看到这里，是不是又发现了一个东西*blob*。其实blob才是javascript中最基础的文件类型。File是派生于blob的，暂且不讲。

第二种比较常用的使用方式，通过`FileReader`对象，读取对应File对象的数据，最终以某种格式显示出来。比如把image类型的File，读取为base64字符串。
```javascript
  var file = files[0]
  var f = new FileReader()
  f.readAsDataURL(file)
  f.onload = event => {
    vm.src = event.target.result
  }
```
> question: 为什么image可以显示base64字符串呢？

比较常见的使用就到这里结束了，工作见到的无非就是图片的处理（最多），各种有关图片的效果
接下来，要讲一些原理、基础方面的东西。

# Blob
