---
title: 浏览器粘贴事件以及File类型
date: 2019-09-24 22:33:59
tags: [每日一问,javascript]
---

浏览器粘贴事件以及File类型，实例mavon-editor【未完】
<!--more-->

浏览器粘贴事件，当使用鼠标右键、Ctrl+V时，在浏览器内，会触发`paste`事件，paste事件会携带粘贴对应的数据信息

复制图片就是通过这种方式

通过event.clipboardData.items 能够获得`DataTransferItem`类型数据
初始化FileReader对象reader，进行item.getAsFile()，能够获得一个File对象。
然后对reader.readAsDataUrl(file) ，能够得到一个包含result属性的对象。这个属性就是图片的Base64字符串

目前微信的截图功能，能够很好的通过上述方式预览图片
但是QQ，TIM的截图功能，传递的并不是一个MIME类型为image/png类型的DataTransferItem对象，而是三个对象，他们分别是
text/plain的`[图片]`，text/html的IMG的图片HTML字符串，还有一个image/png类型的对象。但是第三个对象无法识别