---
layout: post
title: JS高程笔记-Introduction
description: 'JS高程笔记-Introduction'
share: false
tags: [JS高程笔记]
image:
  feature: abstract-1.jpg
---

之前没有完整看过 JS 高程，最近出了第四版，于是下决心要从头看完（英文版）。为了加深理解，也方便以后查看相关要点，做此系列笔记。

## 浏览器内核

针对标准，不同的浏览器有各自的实现，如下表。

<div class="table-wrapper" markdown="block">

| 浏览器  | 引擎                  |
| ------- | --------------------- |
| Chrome  | Blink/V8              |
| Firefox | Gecko/SpiderMoney     |
| Safari  | WebKit/JavaScriptCore |

</div>

浏览器内核分为渲染引擎和 JS 引擎。最开始渲染引擎和 JS 引擎并没有区分得很明确，后来 JS 引擎越来越独立，内核就倾向于只指渲染引擎。

- 浏览器
  - IE：Trident
  - Edge：谷歌的开源 Chromium
  - Chrome：Webkit -> Blink，Blink 内核可以看成是 Webkit 的精简高效强化版。
  - Firefox：Gecko
  - Safari：Webkit
  - Opera：Presto -> Webkit -> Blink
- Chrome 和 Chromium 的区别
  - Chromium 是谷歌的开源项目，由开源社区维护，拥有诸多尖端优势。
  - Chrome 基于 Chromium，但是它是闭源的，跨平台多端支持。
  - 谷歌把核心技术都保留在了 Chrome 中。
  - Chromium 是基于 WebKit 的，Chrome 由 Chromium 开发而来。

目前浏览器会为模拟原生移动应用程序的 API 提供最优的支持。
