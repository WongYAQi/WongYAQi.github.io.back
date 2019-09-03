---
title: 如何修改Hexo主题yilia
date: 2019-07-03 22:38:45
tags: hexo
---


如何定制化yilia的源码，并加上插件功能
<!-- more -->

对于HEXO主题yilia，个人主要是不喜欢它的左侧边栏，喜欢二次元定制化，所以修改了左侧的背景图片。
```javascript
.left-col::before{
	content: "";
	background:url(https://i.loli.net/2019/08/16/qJmF8NVDkgThIub.png) no-repeat 100%;
	background-size:cover;
    opacity:0.5;//透明度设置
	 z-index:-1;
	position: absolute; 
	width: inherit;
	height: inherit;
}
```
完成源码修改后，需要进行编译

```javascript
npm run dist
```
然而这并不是结束，编译结束后，还需要使用hexo进行更新静态文件，更新完成后直接上传到github

```javascript
hexo g -d
```
