---
layout: post
title: JS高程笔记-JavaScript in HTML
description: 'JS高程笔记-JavaScript in HTML'
share: false
tags: [JS高程笔记]
image:
  feature: abstract-4.jpg
---

将 JavaScript 插入 HTML 页面的主要方法是通过 `<script>` 元素。该元素由 Netscape 提出，最先在 Netscape Navigator 2 实现。随后被加入正式的 HTML 规范。

## 1. 使用

使用 `<script>` 有两种方式，内联和外联。内联直接将代码包裹在 `<script>` 和 `</script>` 中；外联通过 `src` 属性引入外部的脚本。

- 内联脚本中的 `</script>` 需要转义：

  ```html
  <script>
    function sayScript() {
      console.log('<\/script>')
    }
  </script>
  ```

- 外联脚本的后缀通常为 `.js`，但不是必须的，浏览器不做检查，只要服务器返回的是 JavaScript 代码都可以。

- 当脚本设置了 `src` 属性并添加了内联代码，内联代码会被忽略。

- 外联脚本的初始请求不受同源策略限制，但返回的 JavaScript 的执行受同源策略限制。
- 内联脚本和外联脚本（未设置 `async` 和 `defer`）按照在文档中的顺序加载并执行，阻塞页面的解析。
- XHTML 对 `<script>` 有更严格的要求，如须指定 `type` 属性为 `text/javascript`；对 `<` 等进行转义（或使用 `<![CDATA[` 和 `]]>` 包裹代码）。
- 推荐使用外联脚本：可维护性高；方便缓存；HTML 和 XHTML 语法一致。
- 对于支持 SPDY/HTTP2 的浏览器，将脚本分成多个独立的小的脚本组件进行请求的耗时和使用单一的大脚本的耗时相差不大，但这些小的脚本组件更能提高缓存的命中率。
- `<noscript>` 元素用来为不支持脚本（或关闭脚本功能）的浏览器展示脚本未执行的替代内容。

## 2. 属性

- async

  - 异步加载 async 脚本，不阻塞页面的解析。
  - 加载完成就执行，以“加载优先”的顺序执行。
  - 仅适用于外部脚本。

- defer

  - 异步加载 defer 脚本，不阻塞页面的解析。
  - defer 脚本等到 DOM 解析完毕，但在 `DOMContentLoaded` 事件之前执行。
  - 具有 defer 特性的脚本保持其执行的相对顺序（即在文档中的顺序），就像常规脚本一样。（IE9 及以下不能保证相应的执行顺序）
  - 仅适用于外部脚本。

- crossorigin

  服务器端如果没有设置 CORS，普通的跨域 `<script>` 标签，将只向 `window.onerror` 反馈尽量少的内容。给 `<script>` 标签添加 crossorigin 属性，并在服务器端设置 Access-Control-Allow-Origin 响应头，允许脚本被跨域访问，就可以在 `window.onerror` 中获取更详细的日志信息。crossorigin 有两个取值 `anonymous` 和 `use-credentials`。

  补充：

  - [HTML5 \<img\> \<link\> 标签里的 crossorigin 属性到底有什么用 \| Chrisyue's Blog](https://www.chrisyue.com/what-the-hell-is-crossorigin-attribute-in-html-img-or-link-tag.html)
  - [\<img\>: The Image Embed element \- HTML: HyperText Markup Language \| MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attributes)
  - [Allowing cross\-origin use of images and canvas \- HTML: HyperText Markup Language \| MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_enabled_image)

- integrity

  脚本的哈希值。浏览器校验脚本的哈希值是否与预期一致，只有一致才能执行，否则拒绝执行并报错。

  详情请看：[Subresource Integrity \- Web 安全 \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/%E5%AD%90%E8%B5%84%E6%BA%90%E5%AE%8C%E6%95%B4%E6%80%A7)

- type

  该属性定义 `<script>` 元素包含或 `src` 引用的脚本语言。

  1. 属性的值为 MIME 类型。支持的 MIME 类型包括 `text/javascript`、 `text/ecmascript`、`application/javascript` 和 `application/ecmascript`。如果没有定义这个属性，脚本会被视作 JavaScript。

  2. 如果 MIME 类型不是 JavaScript 类型（上述支持的类型），则该元素所包含的内容会被当作数据块而不会被浏览器执行。

  3. 如果 type 属性为 `module`，代码会被当作 JavaScript 模块。

     需要注意：

     - 默认具有 defer 特性
     - async 对内联脚本有效
     - 加载跨域外联脚本，需要 CORS 响应头
     - 重复外联脚本只执行一次

## 3. 动态脚本

- 当脚本被附加到文档时，脚本就会立即开始加载。
- 默认情况下，动态脚本的行为是“异步”的，且先加载完成的脚本先执行。（如果设置 `script.async=false`，则可以改变这个规则。脚本将按照脚本在文档中的顺序执行，就像 `defer` 那样。）
- 预加载内容：[Preloading content with rel="preload" \- HTML: HyperText Markup Language \| MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content)

## 参考链接

> - [脚本：async，defer](https://zh.javascript.info/script-async-defer)
> - [async vs defer attributes \- Growing with the Web](https://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html)
> - [[html] script 的 crossorigin 属性 \- 简书](https://www.jianshu.com/p/a45c9d089c93)
> - [\<script\> \- HTML（超文本标记语言） \| MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/script)
> - [Modules, introduction](https://javascript.info/modules-intro#browser-specific-features)
> - [javascript \- When is a CDATA section necessary within a script tag? \- Stack Overflow](https://stackoverflow.com/questions/66837/when-is-a-cdata-section-necessary-within-a-script-tag)
