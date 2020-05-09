---
layout: post
title: 理解 vertical-align
description: '掌握 vertical-align 的原理'
share: false
tags: [CSS]
image:
  feature: abstract-8.jpg
---

vertical-align 用于行内元素和表格单元的垂直对齐，开发中经常会遇到，有必要了解其背后的原理。

## 一、应用范围

对于 `vertical-align`，新手通常的困惑是对块元素进行设置，想垂直居中其子元素，然而并不会起作用（关于居中的方法可查看[CSS 居中](https://huxinsen.github.io/centering-in-css/)）。因为，`vertical-align` 只对以下元素有效：

- `inline-level` 元素，包括 `inline`、`inline-table` 和 `inline-block`
- `table-cell` 元素

## 二、IFC

想了解 `vertical-align` 如何起作用，需要结合行内格式化上下文（Inline formatting context，`IFC`）理解。

`IFC` 如同 `BFC`，也是一种布局规则。其中的 `box` 水平放置。这些 `box` 水平方向的 `margin`，`border`，`padding` 有效。垂直方向有不同对齐方式，例如顶部、底部和基线对齐。容纳一行 `box` 的容器叫 `line box`。

- `line box` 的宽度由包含它的块（`containing box`）和块中是否有浮动来决定。一般 `line box` 的左右边界分别紧挨包含块的左右边界，宽度与包含块的一样。有浮动时，浮动的 `box` 会介于 `line box` 和包含块的边界之间，`line box` 的宽度因此会减小。

- `line box` 的高度由以下计算规则决定：

  1. 计算 `inline-level box` 的高度。

     对于 `inline box`，为其 `line-height`；对于替换元素, `inline-block` 和 `inline-table`，为 `margin box` 的高度。空行内元素的 `margin`、`border`、`padding` 和 `line-height` 也会影响 `line box` 高度的计算。

  2. `inline-level box` 依据 `vertical-align` 对齐。

     CSS 2 未定义 `line box` 的 `baseline`（基线）。每个 `line box` 以一个想象的零宽度的 `inline box` （W3C 称 `strut`）开始，`strut` 拥有包含块的 `font` 和 `line-height` 属性。`baseline` 在 `line box` 的位置由行内所有的 `inline-level box` 共同决定。

  3. `line box` 的高度为行内最高 `box` 的顶部和最低 `box` 的底部之间的距离。

- 当多个 `inline-level box` 一行容不下时，会被分成两个或多个 `line box`。同一个 `IFC` 下，不同 `line box` 的高度可能不一样，例如：某一行包含一个大图片，另一行只有文字。

- 如果一行所有 `inline-level box` 的宽度之和小于 `line box` 的宽度时，`line box` 内的水平布局由 `text-align` 属性决定。

- 当一个 `inline box` 的宽度超过 `line box` 时，会被分割成多个 `box` 分布于两个或多个 `line box`内。如果不能分割，则溢出 `line box`，例如只有一个字、`word-break` 不允许换行或 `white-space` 不允许换行。另外，分割处 `margin`、`border` 和 `padding` 无视觉效果。

还有一点要补充，当 `IFC` 中有块级元素插入时，会产生两个 `IFC`。

## 三、vertical-align 的取值

可以分成三类：

### 1. 相对于 line box 的 baseline

![Relative to baseline](/images/2020-05-08-vertical-align/baseline.png)

- baseline

  初始值，元素的基线与 `line box` 的基线对齐。

- sub

  元素的基线与 `line box` 的下标基线对齐。

- super

  元素的基线与 `line box` 的上标基线对齐。

- percentage

  元素的基线由 `line box` 的基线移动相对 `line-height` 给定百分比的距离。

- length

  元素的基线由 `line box` 的基线移动给定的距离。

- middle

  ![Middle](/images/2020-05-08-vertical-align/middle.png)

  元素的中线与 `line box` 的基线加上小写字母 x 高度（x-height）的一半的位置对齐。

### 2. 相对于 line box 的 strut box

![Relative to strut box](/images/2020-05-08-vertical-align/strut-box.png)

- text-top

  元素的顶部与 `strut box` 的顶部对齐。

- text-bottom

  元素的底部与 `strut box` 的底部对齐。

### 3. 相对于 line box 的边界

![Relative to line box](/images/2020-05-08-vertical-align/line-box.png)

- top

  元素的顶部与 `line box` 的顶部对齐。

- bottom

  元素的底部与 `line box` 的底部对齐。

## 四、表格单元

对于表格单元（`table-cell`）对来说，`vertical-align` 的默认值为 `middle`。
使用时，建议取值：`top`、`middle` 和 `bottom`，其他取值跨浏览器可能有不一致表现。

## 参考链接

> - [Inline formatting context](https://drafts.csswg.org/css2/visuren.html#inline-formatting)
> - [Vertical-Align: All You Need To Know](https://christopheraue.net/design/vertical-align)
> - [Deep dive CSS: font metrics, line-height and vertical-align](https://iamvdo.me/en/blog/css-font-metrics-line-height-and-vertical-align)
> - [What is Vertical Align? - CSS-Tricks](https://css-tricks.com/what-is-vertical-align/)
