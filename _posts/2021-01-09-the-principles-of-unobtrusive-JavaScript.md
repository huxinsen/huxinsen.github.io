---
layout: post
title: unobtrusive JavaScript的原则
description: 'The principles of unobtrusive JavaScript笔记'
share: false
tags: [JavaScript]
image:
  feature: abstract-3.jpg
---

看文章时碰到 `unobtrusive JavaScript`，于是搜到了*The principles of unobtrusive JavaScript* 这篇文章，做个简要的笔记。

## unobtrusive JavaScript 是什么

unobtrusive JavaScript 算不上一项技术，可以理解为一种思想或者编程哲学。unobtrusive 有 `不显著的；不容易被注意到的；不引人注目的` 的意思，很难用一个中文词准确概括它背后的意思，暂且译作 `非侵入的`。简而言之，非侵入 JavaScript 不会因为 JavaScript 无法正常工作将网站访问者拒之门外，他们仍然应该能够使用网站，尽管以更基本的水平使用。

## unobtrusive JavaScript 的原则

- 结构与行为分离

  结构与行为分开，代码更清晰，且易维护。

- 增加可用性层

  JavaScript 的目的是为网站增加一层可用性。请注意是“添加”：如果脚本是整个可用性层（换句话说，如果没有 JavaScript，该站点将无法使用），则脚本不是非侵入的。

  例子：表单验证。使用 JavaScript 验证表单的同时，也需要在服务端进行验证。前者提供了更平滑的界面，但只有后者才能提供适当的安全性（关闭 JavaScript 可以轻易绕过 JavaScript 验证），并在用户浏览器不支持 Javascript 时作为后备选项。

  几条重要原则：

  - 没有 JavaScript，网站应该也能工作。
  - 如果启用了 JavaScript，能为用户提供额外的可用性层，更快地执行相关任务。
  - JavaScript 是不安全的。对于关键任务，不能只使用 JavaScript。

  阻止 HTML 元素的默认行为（如文中弹出窗口和 Ajax 的例子）是非侵入 JavaScript 的典型例子。 如果用户的浏览器支持脚本，则执行它们；否则，退回到 HTML 元素的默认行为。 只要遵守此原则，脚本就是非侵入的。

- 简洁的、语义化的 HTML

  简洁的、语义化的 HTML 使得脚本更易开发和维护。

- 浏览器兼容性

  使用一些库可以解决浏览器兼容问题，但最好理解其底层原理，不要一上来就用。

## 原文链接

> - [The principles of unobtrusive JavaScript \- W3C Wiki](https://www.w3.org/wiki/The_principles_of_unobtrusive_JavaScript)
