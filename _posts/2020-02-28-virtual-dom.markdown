---
layout: post
title: 虚拟 DOM
description: '虚拟 DOM 实现及其意义'
share: false
modified: 2020-02-29
tags: [DOM]
image:
  feature: 2020-02-28-virtual-dom/DOM-model.svg
---

文档对象模型（Document Object Model, DOM）是 HTML 和 XML 文档的编程接口。它提供了对文档的结构化的表述，并定义了一种方式可以从程序中对该结构进行访问，从而改变文档的结构、样式和内容。DOM 将文档解析为一个由节点和对象（包含属性和方法的对象）组成的结构集合。简言之，它将 Web 页面和程序语言连接起来。

一个 Web 页面是一个文档。这个文档可以在浏览器窗口显示或作为 HTML 源码打开。两种情况中它们都是同一份文档。文档对象模型（DOM）提供了对同一份文档的另一种表现、存储和操作的方式。DOM 是 Web 页面的面向对象形式的表述，它能够使用如 JavaScript 等脚本语言进行修改。

一个例子：

```html
<html>
  <head>
    <title>My title</title>
  </head>
  <body>
    <h1>A heading</h1>
    <a href="">Link text</a>
  </body>
</html>
```

![DOM model](/images/2020-02-28-virtual-dom/DOM-model.svg)

## 虚拟 DOM

![div properties](/images/2020-02-28-virtual-dom/div.png)

把一个简单的 div 元素的属性都打印出来，可以看到有这么多属性。由于标准的设计，真实的 DOM 元素丰富而复杂，但我们关心的只是少数元素的少数属性。所以，建立一个 JavaScript 简单对象，非常轻量，用它保存我们真正关心的与 DOM 相关的少数数据；对它进行操作，然后对比操作前后的差异，再根据映射关系去操作真正的 DOM。

这就是虚拟 DOM 的理念。

## 为什么虚拟 DOM 更好

如果没有 Virtual DOM，简单来想就是直接重置 innerHTML。很多人都没有意识到，在一个大型列表所有数据都变了的情况下，重置 innerHTML 其实是一个还算合理的操作。真正的问题是在 “全部重新渲染” 的思维模式下，即使只有一行数据变了，它也需要重置整个 innerHTML，这时候显然就有大量的浪费。

我们可以比较一下 innerHTML vs. Virtual DOM 的重绘性能消耗：

- innerHTML: render html string O(template size) + 重新创建所有 DOM 元素 O(DOM size)
- Virtual DOM: render Virtual DOM + diff O(template size) + 必要的 DOM 更新 O(DOM change)

Virtual DOM render + diff 显然比渲染 html 字符串要慢，但是！它依然是纯 js 层面的计算，比起后面的 DOM 操作来说，依然便宜了太多。可以看到，innerHTML 的总计算量不管是 js 计算还是 DOM 操作都是和整个界面的大小相关，但 Virtual DOM 的计算量里面，只有 js 计算和界面大小相关，DOM 操作是和数据的变动量相关的。前面说了，和 DOM 操作比起来，js 计算是极其便宜的。这才是为什么要有 Virtual DOM：它保证了：

1. 不管你的数据变化多少，每次重绘的性能都可以接受；
2. 你依然可以用类似 innerHTML 的思路去写你的应用。

## Vue 虚拟 DOM Diff 算法

### 原理

1. 先同级比较，再比较子节点
2. 先判断一方有子节点，一方没子节点的情况
3. 比较都有子节点的情况
4. 递归比较子节点

![Vue Diff](/images/2020-02-28-virtual-dom/vue-diff.jpg)

### 源码

> core/vdom/patch.js

```javascript
const oldCh = oldVnode.children // 旧的子节点
const ch = vnode.children // 新的子节点
if (isUndef(vnode.text)) {
  if (isDef(oldCh) && isDef(ch)) {
    // 比较子节点
    if (oldCh !== ch)
      updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly)
  } else if (isDef(ch)) {
    // 新的有子节点 旧的没有
    if (isDef(oldVnode.text)) nodeOps.setTextContent(elm, '')
    addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue)
  } else if (isDef(oldCh)) {
    // 如果旧的有 新的没有 就删除
    removeVnodes(oldCh, 0, oldCh.length - 1)
  } else if (isDef(oldVnode.text)) {
    // 旧的有文本 新的没文本 将旧的清空
    nodeOps.setTextContent(elm, '')
  }
} else if (oldVnode.text !== vnode.text) {
  // 文本不相同 进行替换
  nodeOps.setTextContent(elm, vnode.text)
}
function updateChildren(
  parentElm,
  oldCh,
  newCh,
  insertedVnodeQueue,
  removeOnly
) {
  let oldStartIdx = 0 // 旧开始索引
  let newStartIdx = 0 // 新开始索引
  let oldEndIdx = oldCh.length - 1 // 旧结束索引
  let oldStartVnode = oldCh[0] // 旧开始节点
  let oldEndVnode = oldCh[oldEndIdx] // 旧结束节点
  let newEndIdx = newCh.length - 1 // 新结束索引
  let newStartVnode = newCh[0] // 新开始节点
  let newEndVnode = newCh[newEndIdx] // 新结束节点
  let oldKeyToIdx, idxInOld, vnodeToMove, refElm

  const canMove = !removeOnly

  if (process.env.NODE_ENV !== 'production') {
    checkDuplicateKeys(newCh)
  }

  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    if (isUndef(oldStartVnode)) {
      // 判断为`无效标记`，跳过该节点
      oldStartVnode = oldCh[++oldStartIdx]
    } else if (isUndef(oldEndVnode)) {
      // 判断为`无效标记`，跳过该节点
      oldEndVnode = oldCh[--oldEndIdx]
    } else if (sameVnode(oldStartVnode, newStartVnode)) {
      // 旧开始节点和新开始节点比较
      patchVnode(
        oldStartVnode,
        newStartVnode,
        insertedVnodeQueue,
        newCh,
        newStartIdx
      )
      oldStartVnode = oldCh[++oldStartIdx]
      newStartVnode = newCh[++newStartIdx]
    } else if (sameVnode(oldEndVnode, newEndVnode)) {
      // 旧结束节点和新结束节点比较
      patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx)
      oldEndVnode = oldCh[--oldEndIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldStartVnode, newEndVnode)) {
      // 旧开始节点和新结束节点比较
      patchVnode(
        oldStartVnode,
        newEndVnode,
        insertedVnodeQueue,
        newCh,
        newEndIdx
      )
      // 进行 DOM 移动，把旧开始真实 DOM 节点移动到尾部
      canMove &&
        nodeOps.insertBefore(
          parentElm,
          oldStartVnode.elm,
          nodeOps.nextSibling(oldEndVnode.elm)
        )
      oldStartVnode = oldCh[++oldStartIdx]
      newEndVnode = newCh[--newEndIdx]
    } else if (sameVnode(oldEndVnode, newStartVnode)) {
      // 旧结束节点和新结束节点比较
      patchVnode(
        oldEndVnode,
        newStartVnode,
        insertedVnodeQueue,
        newCh,
        newStartIdx
      )
      // 进行 DOM 移动，把旧结束真实 DOM 节点移动到头部
      canMove &&
        nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm)
      oldEndVnode = oldCh[--oldEndIdx]
      newStartVnode = newCh[++newStartIdx]
    } else {
      // 在旧子节点中查找是否存在`新开始节点`
      if (isUndef(oldKeyToIdx))
        oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx)
      idxInOld = isDef(newStartVnode.key)
        ? oldKeyToIdx[newStartVnode.key]
        : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx)
      if (isUndef(idxInOld)) {
        // 不存在则插到头部
        createElm(
          newStartVnode,
          insertedVnodeQueue,
          parentElm,
          oldStartVnode.elm,
          false,
          newCh,
          newStartIdx
        )
      } else {
        // 存在则移动到头部
        vnodeToMove = oldCh[idxInOld]
        if (sameVnode(vnodeToMove, newStartVnode)) {
          // 节点类型一样，则更新属性后移动
          patchVnode(
            vnodeToMove,
            newStartVnode,
            insertedVnodeQueue,
            newCh,
            newStartIdx
          )
          // 设置`无效标记`，下次跳过此节点不处理
          oldCh[idxInOld] = undefined
          canMove &&
            nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm)
        } else {
          // 节点类型不一样，则新建真实 DOM 节点插入
          createElm(
            newStartVnode,
            insertedVnodeQueue,
            parentElm,
            oldStartVnode.elm,
            false,
            newCh,
            newStartIdx
          )
        }
      }
      newStartVnode = newCh[++newStartIdx]
    }
  }
  // 旧队列处理完毕，新队列未处理完
  if (oldStartIdx > oldEndIdx) {
    refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
    addVnodes(
      parentElm,
      refElm,
      newCh,
      newStartIdx,
      newEndIdx,
      insertedVnodeQueue
    )
  } else if (newStartIdx > newEndIdx) {
    // 新队列处理完毕，旧队列未处理完
    removeVnodes(oldCh, oldStartIdx, oldEndIdx)
  }
}
```

### 简易实现

[https://github.com/huxinsen/front-end-learning/tree/vue-virtual-dom](https://github.com/huxinsen/front-end-learning/tree/vue-virtual-dom)

## 参考链接

> - [什么是 DOM? - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction)
> - [如何理解虚拟 DOM? - 工业聚的回答 - 知乎](https://www.zhihu.com/question/29504639/answer/44662943)
> - [网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么？ - 尤雨溪的回答 - 知乎](https://www.zhihu.com/question/31809713/answer/53544875)
> - [Vue.js 源码](https://github.com/vuejs/vue)
