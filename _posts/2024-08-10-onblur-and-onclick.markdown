---
layout: post
title: onblur 和 onclick 执行顺序
description: 'onblur 和 onclick 执行顺序问题和解决方法'
share: false
tags: [事件]
image:
  feature: abstract-2.jpg
---

在实现 input 失焦隐藏下拉框，并且点击下拉框选项隐藏下拉框的功能，或者实现 input 的 clear 功能会遇到这种问题：input 的 onblur 执行早于下拉框或 clear 按钮的 onclick。解决方案如下：

## 1. 使用 onmousedown

直接将 onclick 替换成 onmousedown。这种方案最简单，适用于大多数情况。但该方案 Click 的响应从 mouse up 移到了 mouse down，需要注意这种方案下误点击时不能通过移出点击元素实现取消点击。如果想做到点击可取消，需要结合使用 mousedown 和 mouseup。

## 2. 使用 onmousedown 和 onmouseup

```html
<div class="menu" onmousedown="setFlag()" onmouseup="doProcessing()">...</div>
<input id="input" onblur="removeMenu()" ... />
```

```javascript
var mouseFlag;

function setFlag() {
  mouseFlag = true;
}

function removeMenu() {
  if (!mouseFlag) {
    document.getElementById('menu').innerHTML = '';
  }
}

function doProcessing() {
    mouseFlag = false;
    ...
}
```

## 3. onmousedown 和 event.preventDefault()

onmousedown 默认会触发 blur 事件，使用 event.preventDefault() 可以阻止该事件，那么 onclick 就能执行。onblur 也会执行但是是在 onclick 之后了。

```html
<div class="menu" onmousedown="event.preventDefault()" onclick="onClick()">...</div>
<input id="input" onblur="onBlur()" ... />
```
