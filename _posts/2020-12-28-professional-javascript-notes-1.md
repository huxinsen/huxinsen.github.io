---
layout: post
title: JS高程笔记-What Is JavaScript?
description: 'JS高程笔记-What Is JavaScript?'
share: false
tags: [JS高程笔记]
image:
  feature: abstract-2.jpg
---

尽管经常将 JavaScript 和 ECMAScript 用作同义词，但 JavaScript 不仅限于 ECMA-262 中定义的内容。实际上，一个完整的 JavaScript 实现由以下三个不同的部分组成：ECMAScript、DOM (Document Object Model)和 BOM (Browser Object Model)。

## 一、JavaScript 的历史

当 JavaScript 在 1995 年出现时，它的主要目的是处理以前留给服务器端语言（例如 Perl）的某些输入验证的任务（省去了服务端验证的往返时间）。之后 JavaScript 便成为市场上主流浏览器的重要功能。JavaScript 不再局限于简单的数据验证，现在可以与浏览器窗口及其内容的几乎所有方面进行交互。

- 诞生

  1995 年，Netscape 的程序员 Brendan Eich 为 Netscape Navigator 2 的发布开发一种名为 Mocha（后来更名为 LiveScript）的脚本语言。目的是在浏览器和服务器上都使用它，被称为 LiveWire。而当 Netscape Navigator 2 正式发布时，Netscape 为了利用 Java 的名声，将 LiveScript 更名为 JavaScript。

- 两家争霸

  由于 JavaScript 1.0 非常受欢迎，Netscape 在 Netscape Navigator 3 中发布了 1.1 版。不久之后微软发布了 Internet Explorer 3，其带有一个称为 JScript 的 JavaScript 实现。

- 标准化

  没有统一的标准势必影响 JavaScript 的发展。1997 年，JavaScript 1.1 作为提案提交给了欧洲计算机制造商协会（European Computer Manufacturers Association, ECMA）。分配了第 39 技术委员会（TC39）以“标准化通用、跨平台、与供应商无关的脚本语言的语法和语义”。TC39 花了几个月的时间来敲定 ECMA-262，该标准定义了一种新的脚本语言，名为 ECMAScript（通常发音为 “ ek-ma-script”）。

## 二、JavaScript 和 ECMAScript 的关系

尽管经常将 JavaScript 和 ECMAScript 用作同义词，但 JavaScript 不仅限于 ECMA-262 中定义的内容。实际上，一个完整的 JavaScript 实现由以下三个不同的部分组成：

- ECMAScript
- DOM (Document Object Model)
- BOM (Browser Object Model)

Web 浏览器只是一种其中可能存在 ECMAScript 实现的宿主环境。其他宿主环境包括 NodeJS——一个服务器端 JavaScript 平台以及日益淘汰的 Adobe Flash。

ECMAScript 只是对语言的描述，该语言实现了规范中描述的所有方面。JavaScript 实现了 ECMAScript，Adobe ActionScript 也实现了它。

## 三、ECMAScript 版本

<div class="table-wrapper" markdown="block">

| 版本 |   发表日期    | 与前版本的差异                                                                                                                                                                                                                                                                                                     |
| :--: | :-----------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|  1   | 1997 年 6 月  | 首版                                                                                                                                                                                                                                                                                                               |
|  2   | 1998 年 6 月  | 格式修正，以使其形式与 ISO/IEC16262 国际标准一致。                                                                                                                                                                                                                                                                 |
|  3   | 1999 年 12 月 | 强大的正则表达式、更好的词法作用域链处理、新的控制指令、异常处理、更加明确的错误定义、数据输出的格式化等。                                                                                                                                                                                                         |
|  4   |     放弃      | 由于关于语言的复杂性出现分歧，第 4 版本被放弃，其中的部分成为了第 5 版本及 Harmony 的基础。（到 2008 年 8 月，ECMAScript 第 4 版提案已缩减为一个代号为 ECMAScript Harmony 的项目）                                                                                                                                 |
|  5   | 2009 年 12 月 | 新增“严格模式（strict mode）”，提供更彻底的错误检查，以避免容易出错的结构。阐明了第三版规范中的许多歧义，并适应了与该规范不一致的现实世界实现的行为。添加了一些新功能，例如 getter 和 setter，对 JSON 的库支持以及对对象属性的更完整反射。                                                                         |
| 5.1  | 2011 年 6 月  | 使规范更符合 ISO/IEC 16262:2011 第三版。                                                                                                                                                                                                                                                                           |
|  6   | 2015 年 6 月  | ECMAScript 2015（ES2015），第 6 版，最早被称作是 ECMAScript 6（ES6），添加了类和模块的语法，其他特性包括迭代器、Python 风格的生成器和生成器表达式、箭头函数、二进制数据、静态类型数组、集合（maps，sets 和 weak maps）、promise、reflection 和 proxies。作为最早的 ECMAScript Harmony 版本，也被叫做 ES6 Harmony。 |
|  7   | 2016 年 6 月  | ECMAScript 2016（ES2016），第 7 版，多个新的概念和语言特性，例如 Array.prototype.includes、求幂运算符 （\*\*）和异步编程的 await 与 async 关键字。                                                                                                                                                                 |
|  8   | 2017 年 6 月  | ECMAScript 2017（ES2017），第 8 版，多个新的概念和语言特性，例如 Object.values()、Object.entries()、String padding: `padStart()`和`padEnd()`、函数参数列表结尾允许逗号、Object.getOwnPropertyDescriptors()和 async 函数。                                                                                          |
|  9   | 2018 年 6 月  | ECMAScript 2018 （ES2018），第 9 版，多个新的概念和语言特性，例如异步迭代，新的正则表达式特性和 rest/spread 语法和 Promise.prototype.finally。                                                                                                                                                                     |
|  10  | 2019 年 6 月  | ECMAScript 2019 （ES2019），第 10 版，多个新的概念和语言特性，例如 Array.prototype.flat、Array.prototype.flatMap、稳定的 Array.sort 和 Object.fromEntries。                                                                                                                                                        |
|  11  | 2020 年 6 月  | ECMAScript 2020 （ES2020），第 11 版，多个新的概念和语言特性，例如无效合并运算符（??）、BigInt 和 Promise.allSettled。                                                                                                                                                                                             |

</div>

到 2008 年，五种主要的网络浏览器（Internet Explorer，Firefox，Safari，Chrome 和 Opera）均符合第三版 ECMA-262。Internet Explorer 8 是第一个实施第五版 ECMA-262 规范的浏览器，并在 Internet Explorer 9 中提供了完整的支持。Firefox 4 紧随其后。

## 四、DOM

文档对象模型（DOM）是 HTML 和 XML 文档的编程接口。W3C（World Wide Web Consortium ）负责 DOM 标准的制定。

需要注意 DOM 不是特定于 JavaScript 的，实际上已被多种其他语言实现。但是，对于 Web 浏览器，已经使用 ECMAScript 实现了 DOM，现在 DOM 构成了 JavaScript 语言的很大一部分。

DOM 级别：

<div class="table-wrapper" markdown="block">

|   级别   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| :------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DOM 0 级 | 注意没有 DOM 0 级的标准，它只是 DOM 历史上的参考点。DOM 0 级被认为是 Internet Explorer 4.0 和 Netscape Navigator 4.0 最初支持的 DHTML。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| DOM 1 级 | DOM1 级（DOM Level 1）于 1998 年 10 月成为 W3C 的推荐标准。DOM1 级由两个模块组成：DOM 核心（DOM Core）和 DOM HTML。其中，DOM 核心规定的是如何映射基于 XML 的文档结构，以便简化对文档中任意部分的访问和操作。DOM HTML 模块则在 DOM 核心的基础上加以扩展，添加了针对 HTML 的对象和方法。                                                                                                                                                                                                                                                                                                                                                                         |
| DOM 2 级 | 如果说 DOM1 级的目标主要是映射文档的结构，那么 DOM2 级的目标就要宽泛多了。DOM2 级在原来 DOM 的基础上又扩充了（DHTML 一直都支持的）鼠标和用户界面事件、范围、遍历（迭代 DOM 文档的方法）等细分模块，而且通过对象接口增加了对 CSS（Cascading Style Sheets，层叠样式表）的支持。DOM1 级中的 DOM 核心模块也经过扩展，支持 XML 命名空间。DOM2 级引入了下列新模块，以处理新类型和新接口：DOM 视图（DOM Views）：定义了跟踪不同文档（例如，应用 CSS 之前和之后的文档）视图的接口；DOM 事件（DOM Events）：定义了事件和事件处理的接口；DOM 样式（DOM Style）：定义了处理元素 CSS 样式的接口；DOM 遍历和范围（DOM Traversal and Range）：定义了遍历和操作文档树的接口。 |
| DOM 3 级 | DOM3 级则进一步扩展了 DOM，引入了以统一方式加载和保存文档的方法（在 DOM Load and Save 模块中定义）；新增了验证文档的方法（在 DOM Validation 模块中定义）。DOM3 级也对 DOM 核心进行了扩展，支持 XML 1.0 规范，包括 XML Infoset、XPath 和 XML Base。                                                                                                                                                                                                                                                                                                                                                                                                             |

</div>

当前，W3C 不再将 DOM 维护为一系列级别，而是将其作为 DOM Living Standard，将其快照称为 DOM4。其中一个更新是用 Mutation Observers 替代 Mutation Events。

## 参考链接

> - [ECMAScript \- Wikipedia](https://en.wikipedia.org/wiki/ECMAScript)
> - [JavaScript 语言的历史 \- JavaScript 教程 \- 网道](https://wangdoc.com/javascript/basic/history.html)
> - [javascript \- What is the difference in DOM levels, and how do they interrelate? \- Stack Overflow](https://stackoverflow.com/questions/20334072/what-is-the-difference-in-dom-levels-and-how-do-they-interrelate)
> - [javascript \- What is the difference between DOM Level 0 events vs DOM Level 2 events? \- Stack Overflow](https://stackoverflow.com/questions/5642659/what-is-the-difference-between-dom-level-0-events-vs-dom-level-2-events)
