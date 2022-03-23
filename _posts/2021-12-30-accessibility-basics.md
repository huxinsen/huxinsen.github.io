---
layout: post
title: 可访问性基础
description: '了解可访问性的含义及意义，学习其相关的最佳实践'
share: false
tags: [可访问性]
image:
  feature: abstract-6.jpg
---

前段时间听过两次分享，分别是提高用户体验和组件化，都提到了可访问性，于是想深入系统地学习一下可访问性，整理出此文。

## 1. 初识可访问性

### 1.1. 什么是可访问性

可访问性是一种让尽可能多的用户可以使用你的网站的做法。传统上我们认为这只与残疾人士有关，但提升网站的可访问性也可以让其他用户群体受益，比如使用移动设备的人群，以及那些使用低速网络的人群。

你可以把可访问性看成是同等地对待每一个人，给他们平等的机会，无论他们的能力或所处的环境如何。就像不让坐轮椅的人进入大楼是不对的（现代公共建筑通常有轮椅坡道或电梯），不让视觉有障碍的人士浏览我们的网站同样不应该。我们都是不同的，但我们都是人，因此享有同等的人权。

### 1.2. 构建易访问网站的重要性

- 使用语义化 HTML，不仅提升了可访问性，也增强了搜索引擎优化（SEO），使你的网站更容易被找到。
- 关心可访问性表露出良好的道德品质，可以提升公众形象。
- 其他一些改善可访问性的做法也会让你的网站更容易被其他群体使用，比如手机用户，低速网络环境的用户。事实上，每个人都可以从这些改善中受益。（比如[颜色及其对比](#color-and-contrast)）
- 这也是一些地方的法律规定。

### 1.3. 我们关注哪些 disability？

这里的 disability 不能只理解为残疾，它还有无能力的意思，比如用户没有鼠标，所以这里保留了英文没有翻译。disability 有很多种形式，难以枚举，但关键的处理方式是：在你使用的设备和上网方式之外，思考其他的用户怎样使用你的网站——你不是你的用户。四种主要的 disability 类型如下：

1. 视力障碍，包括失明、低视力和色盲

   - 世界卫生组织估计“全世界有 2.85 亿人视力受损：3900 万人失明，2.46 亿人视力低下。”
   - 多数有视力障碍的用户使用放大镜，要么物理放大镜，要么软件缩放功能。
   - 一些用户使用[屏幕阅读器](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing/Accessibility#screenreaders)，包括免费产品、付费产品和操作系统内置的软件。

2. 听力障碍

   - 世界卫生组织估计全世界有 4.66 亿人患有听力损失。
   - 为了提供访问，必须提供替代文本。视频应添加说明文字；音频内容应提供文字记录。
   - 由于在 DHH (Deaf or Hard of Hearing) 人群中，存在高度的[语言剥夺（language deprivation）](https://therapytravelers.com/language-deprivation/)的情况，应当考虑文字简化。

3. 行动不便

   - 身体问题或神经/遗传疾病导致的不同程度的行动不便（从使用鼠标困难到需要使用头戴指针与计算机交互）。
   - 行动不便也可能由上了年纪导致，还可能由硬件限制导致，比如用户没有鼠标。
   - 由于以上问题，web 开发工作通常要求页面内的组件可通过键盘访问。

4. 认知障碍

   - 涵盖范围很广，从智力障碍人士到因年龄增长思考和记忆困难的所有人。
   - 包括有精神疾病的人，如抑郁和精神分裂。
   - 包括有学习障碍得人，如阅读障碍和注意力缺陷多动障碍（attention deficit hyperactivity disorder, ADHD）。
   - 重要的是，尽管认知障碍的临床定义存在很多差异，但患有这些障碍的人会遇到一组常见的功能问题。这些问题包括：

     - 难以理解内容
     - 难以记住如何完成任务
     - 以及由不一致的网页布局导致不知所措

   - 认知障碍可能是片刻的、暂时的或者永久的。

     - 永久的如痴呆、阿尔兹海默症、阅读障碍和注意力缺陷多动障碍
     - 暂时的如被酒精或药物影响、暂时的情绪低落、以及睡眠不足

   - 为有认知障碍的人提供可访问性的良好基础包括：

     - 以多种方式交付内容，例如通过文本转语音或通过视频。
     - 易于理解的内容。
     - 重点关注重要内容。
     - 尽量减少干扰，例如不必要的内容或广告。
     - 一致的网页布局和导航。
     - 熟悉的元素，例如带下划线的链接未访问时为蓝色，访问过为紫色。
     - 使用进度指示器将流程划分为符合逻辑的基本步骤。
     - 在不影响安全性的情况下，网站身份验证尽可能简单。
     - 使表单易于完成，例如具有清晰的错误消息和简单的错误恢复。

## 2. WCAG

[WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/) (Web Content Accessibility Guidelines)，由 W3C 的 WAI (Web Accessibility Initiative，网络无障碍倡议) 小组发布，定义了 17 条具体指南，其中 6 条与认知可访问性特别相关。**具有认知可访问性的设计将催生良好的设计实践，使所有人受益。**

### 2.1. 适应性

- 创建可以以不同方式呈现的内容，而不丢失信息或结构，比如响应式布局。
- 让内容可以被软件理解是确保内容能以其他方式呈现的良好方式。

### 2.2. 时间

- 为用户提供足够的时间来阅读和使用内容，或者取消时间限制（如 30 分钟后自动退出，15 分钟内完成之后支付）。
- 如果移动、闪烁、滚动或自动更新的信息自动启动，持续时间超过 5 秒，并且与其他内容并行呈现，则用户必须能够暂停、停止、隐藏或控制它，除非它是必不可少的功能。
- 其余时间标准：

  - 尽量避免有时限的内容；
  - 允许用户延迟或关闭不重要的功能；
  - 会话过期后重新登录，活动数据不丢失；
  - 对不活跃可能导致数据丢失予以提示。

### 2.3. 导航

提供帮助用户导航、查找内容和确定他们所在位置的方法。

- 使用 `<title>`，可以帮助用户知道内容的目标。
- 使用清晰、描述性的标题和标签，方便导航和促进理解。
- 提供多种查找内容的方式

  比如有人喜欢按顺序浏览页面，有人喜欢使用目录、站点地图或搜索。

- 支持跳过内容，比如 [Skip Links](http://web-accessibility.carnegiemuseums.org/code/skip-link/)。

- 聚焦的顺序要合理

  为此，DOM 顺序应与视觉顺序相匹配，而视觉顺序又应与 [Tab 键顺序](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex)相匹配。

- 聚焦的元素应该在视觉上聚焦

  不要修改或删除 `:focus` 的样式，除非能使聚焦效果更醒目。

- 链接文字要有具体的含义

  一些辅助技术会把链接文字脱离语境，形成链接的列表供用户导航使用，所以可理解的链接文字则变得很重要。

- 可以查看当前的位置

  使用面包屑、站点地图，以及把当前页面标记为“当前”可以帮助定位当前位置。

### 2.4. 可读性

- 声明页面的语言以及非主语言内容的语言

  为 `html` 元素以及非主语言的文本设置 `lang` 属性。正确使用 `lang` 可以让一些屏幕阅读器在将文本转换为合成语音时正确朗读文本，它还可以帮助使用文本转语音软件的人。

- 定义不寻常的词和词的用法

  一些残疾导致难以理解非字面词的用法，例如习语、口语和专业术语。非母语用户也可能难以理解这些术语。

- 定义缩写

  首次使用时提供缩写的完整拼写，然后将缩写放在 `<abbr>` 元素中。如果缩写没有完整拼写，或者是不在文档主要语言（如拉丁语）中的单词的缩写，请解释其含义。另外可以考虑为首字母缩略词添加[注音](https://www.w3.org/WAI/WCAG21/Techniques/html/H62)。

- 提高可读性

  尽量简洁清晰地表达内容，最好看一遍就懂。原则包括：

  - 使用简短的单词和短句子；
  - 使用[现在时的主动语态](https://writing.wisc.edu/handbook/style/ccs_activevoice/)；
  - 使用正确的语法和拼写；
  - 可以为有认知障碍的用户提供低阅读水平的文本摘要（有时称为 TL;DR;，或“太长，没读”）；
  - 另外可借助一些视觉效果帮助更好地解释想法、事件和过程；
  - 可以借助一些工具检测内容的阅读水平。

- 提供发音的指导

  这可以帮助许多不同类型的人，包括喜欢大声朗读的人、非母语人士以及可能不熟悉上下文中术语含义的人。方式有：

  - 单词后提供发音；
  - 链接到发音列表；
  - 提供带有发音的词汇表；
  - 使用 `<ruby>` 元素来说明单词的发音方式。

### 2.5. 可预测性

可预测性包括内容呈现顺序可预测、功能和交互组件的行为可预测。

- 元素获得焦点时不要改变上下文，而是应该由用户主动触发功能引发上下文变化。

  - 不正确的例子：当组件获得焦点时自动提交表单；组件获得焦点时启动新窗口。
  - 正确的例子：下拉菜单链接被点击才进行跳转。

- 基于主动的请求更改设置。

  表单控制操作和数据输入应导致可预测的行为，更改状态需要有意识的用户操作，如选中复选框、输入数据或更改选择选项。在上下文更改之前还要确保提供一个提交按钮来触发上下文的更改，并描述会发生什么。

- 在整个网站上保持导航一致性。

  例如，如果在多个页面上有一个导航栏，请在同一位置使用相同链接使整个网站的导航统一。这不仅适用于导航：在所有重复组件每次出现时都以相同的相对顺序显示它们。

- 提供一致的标签。

  相同的功能每次使用时都应该有类似的标签。按钮标签、图标的替代文本和相似交互的图标等保持一致，可以帮助到所有用户。

- 保持一致和可预测，并使用规范。

  保持图标（以及标签文本，如果标记的话）的使用的一致性有助于人们理解图标代表什么。同样，不要更改如浏览器的后退按钮之类的默认值（或行为）。如果需要重定向用户，请提前告知用户。

### 2.6. 输入帮助

[输入协助指南](https://www.w3.org/WAI/WCAG21/Understanding/input-assistance)旨在降低用户（尤其是残障人士）犯错的可能性，如果他们确实犯了错误，则提高他们看到和理解错误消息并成功修复任何错误的可能性。

- 自动的错误检测

  如果有客户端错误检测，请遵循以下准则，使错误信息在传达给用户时尽可能有效：

  - 错误必须使用文本描述。有些人难以理解图标和其他视觉提示的含义。
  - 其他人可能难以理解错误消息的文本版本。对于这些人，还要提供图标和颜色之类的东西。
  - 确保错误消息尽可能具体。
  - 如果输入的值无效，请提供文本以标识不完整的必填字段和文本说明。
  - 如果错误阻止了表单提交，请聚焦到该错误。如果存在多个错误，请提供摘要，每个错误都链接到相关输入。
  - 此外，提供有关表单何时提交成功的反馈。

- 为用户输入提供说明

  - 在表单的开始，给出如何操作的文字说明。当用户需要输入信息时，请附上标签或说明，使用 `<fieldset>`、`<legend>` 和 `<label>`（作用：视觉和程序上联系在一起；扩大点击区域）元素进行输入。
  - 标签应该是描述性的，并放置在对应的输入附近。当需要特定格式进行输入时，请提供正确格式化的示例。此外，考虑在服务器端验证，帮助格式化输入数据，方便用户输入。
  - 如果表单项是必须的，请通过视觉和代码（[aria-required](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-required)="true" 和 required）的方式标记它。如果表单项会更改上下文，请提前告知用户会发生什么。

- 错误建议

  如果自动检测到输入错误并知道改正的建议，请向用户提供建议输入（除非这样做会危及内容的安全性或目的）。

- 预防灾难

  对于可能导致法律、财务或其他重大后果的提交材料，用户应该能够在最终提交之前查看、确认和更正信息。此外，除了提交按钮外，请务必提供确认复选框。如果提交导致法律或金融交易发生，请提供用户可以修改或取消请求的时间节点。

- 提供帮助

  应提供上下文相关的帮助。如果表单需要文本输入，请对目的和必要输入进行说明，包括对长格式文本输入的拼写检查和建议，以及帮助和支持材料的链接。如果需要输入特定的数据格式，请提供示例。

## 3. HTML: 可访问性的良好基础

只要确保始终将 HTML 元素用于正确的目的，就可以使大量 Web 内容具备可访问性。相反，如果不能适当地使用 HTML，会极大地破坏可访问性。

目标不是“要么全有要么全无”；我们做的每一项改进都将有助于可访问性事业。

语义化 HTML

- 提供良好的可访问性。
- 方便开发：可以默认获得一些功能的支持；更好理解。
- 对移动设备更好：代码更轻量。
- 方便搜索引擎能识别页面结构，有利于 SEO。

### 3.1. 好的语义

1.  好的文本结构（使用 h1、h2、p、ol、ul 等元素）。
2.  简洁的语言。避免 `-`（短线），展开缩略形式，展开首字母缩略词一两次。
3.  使用现代页面布局。表格布局的问题：屏幕阅读器容易出错，不够灵活，不好维护，占用更多的带宽。
4.  语义和布局之外，内容的源码顺序要合理，这样阅读器读的顺序才能合理。
5.  UI 控制。主要指按钮，链接和表单控制。它们默认可以通过键盘访问，聚焦的元素会被高亮，按回车键可以打开聚焦的链接，点击聚焦的按钮或开始在输入框输入内容。使用合适的元素即可获得这些支持。反例：div 作 button。
6.  实现键盘可访问性 (tabindex)。建议一开始就选用合适的元素。
7.  使用有意义的文本标签

    - good:

      ```html
      <p>
        Whales are really awesome creatures.
        <a href="whales.html">Find out more about whales</a>.
      </p>
      ```

    - bad:

      ```html
      <p>
        Whales are really awesome creatures. To find more out about whales,
        <a href="whales.html">click here</a>.
      </p>
      ```

### 3.2. 可访问的数据表格：为表格添加标题和表头。

### 3.3. 替代文本

1. 为图片添加文本说明（`alt`，`title`）

   - 设置好的文件名同样重要，当没有替代文本的时候，屏幕阅读器只会读出文件名。
   - 除了使用 `alt`，也可用 `aria-labelledby`，实现文本复用

2. 装饰图

   - 设置空的替代文本，屏幕阅读器识别这是图片，但不会试图描述它。
   - 设置 aria role 为 `presentation` 或 `none`，表示元素只有装饰作用，没有可访问性的语义。
   - 尽可能做成背景图。

3. 音视频配上文本

   - 除了对残疾人有益，对所有人都可能有益，比如低网速用户和处在嘈杂环境的用户。

### 3.4. 链接

1. 样式区别，除了颜色，还有明显的外形的差别
   - 颜色对比，与背景对比至少 4.5:1
   - 不同类型或状态的文本之间颜色对比 3:1
2. 点击事件
   - 不要使用伪按钮：设置 href 为 `#` 或 `javascript:void(0)` 有诸多问题：复制拖拽、在新窗口或 tab 打开、添加书签等。这种场景应该使用 `button`。
3. 外链和指向非 HTML 资源的链接
   - 对点击后的行为进行说明。如果使用了图标来标识这类行为，请确保图标设置了替代文本。
4. Skip links。跳过多个页面之间重复的内容，直达主要内容。
5. 距离
   - 放置在视觉上相近位置的大量交互式内容（如锚点），彼此要有一定间距。这样对患有精细运动控制问题的人更友好，另外可以防止在导航时误触。

## 4. CSS

### 4.1. 正确的语义与用户的期望

使用正确的语义与用户期望有很大关系——元素根据其功能具有特定的外观和行为，这些常见约定是用户所期望的。

1.  标准的结构。保证标题和列表突出，文本清晰易读。
2.  强调的文本，不要大改样式，否则令用户困惑。
3.  缩写的下划点线是传统的样式，不建议修改。
4.  链接
    - 可以自定义链接的样式，但要确保用户交互时给出对应的反馈。
    - 不要清除光标和高亮的样式，这对键盘可访问性很重要。
5.  表单元素的样式修改原则与链接一样。
6.  表格
    - 符合设计、达到美化效果。
    - 表头突出，使用斑马条纹。

### 4.2. <a name="color-and-contrast">颜色及其对比</a>

- 确保有足够的对比。
- 高对比度能够让智能手机或平板电脑的用户在阳光等明亮环境中更好地阅读页面。
- 不能仅依靠颜色来区分信息，因为这对看不出颜色区别的人没有用。例如，不要只用红色标记必要的表单字段，而是用星号和红色标记它们。

### 4.3. 隐藏

对于多个 tab 切换这种内容不是一次全部展示的场景，最好的做法是使用绝对定位，因为屏幕阅读器能全部读取出来。而使用 `visibility: hidden` 或 `display: none`，对屏幕阅读器来说，内容会被隐藏。除非有正当的理由，否则不建议使用。

### 4.4. 用户可能覆盖样式

用户可能出于各种原因覆盖样式，我们要确保设计足够灵活，比如主内容区域支持更大的字体出现滚动，而不是隐藏。

## 5. JavaScript

没有规定说对所有用户必须百分百可访问，只需要尽可能做到易访问。复杂的场景，比如 3D 游戏，对有视觉障碍的用户来说很难做到可访问。当然 3D 游戏并没有把这些用户作为主要目标用户，这也可以理解，但是满足键盘可访问性以及足够的颜色对比还是有必要的。

### 5.1. 不要过度使用 JavaScript

- 使用 JavaScript 生成 HTML 和 CSS 会产生可访问性和其他的问题，不建议这样做。
- 除了使用正确的元素，还要考虑用正确的技术达到目的。比如需要考虑是使用复杂的不标准的表单组件，还是使用一个文本输入框。

### 5.2. 非入侵使用 JavaScript

- JavaScript 用来增强能力，基础功能尽可能不依靠 JavaScript 来实现。
- 好的例子
  - 客户端校验
  - 为 HTML5 `<video>` 提供自定义控件，方便键盘用户访问，以及在 JavaScript 不可用时提供可直接访问视频的链接。（大多数浏览器无法键盘访问默认的 `<video>`）

### 5.3. 其他问题

- 添加鼠标相关的事件处理时，尽量提供对等的通过键盘访问的事件处理。
  - 放大镜图片例子
    ```javascript
    imgThumb.onmouseover = showImg;
    imgThumb.onmouseout = hideImg;
    imgThumb.onfocus = showImg;
    imgThumb.onblur = hideImg;
    ```
- Click 事件
  - 回车或移动设备 tap 也能触发。
  - 为非默认可聚焦的元素添加可聚焦处理（tabindex），还需要判断按下的是哪个键。

## 6. WAI-ARIA

[WAI-ARIA](https://www.w3.org/WAI/standards-guidelines/aria/) (Web Accessibility Initiative - Accessible Rich Internet Applications)，是由 W3C 编写的规范，定义了一组额外的 HTML 属性，这些属性补充缺失的语义以及提供额外的语义以改进可访问性。

### 6.1 一些新问题

- HTML5 新标签出现，带来旧标签的语义化问题。
- 一些新组件 (如 date 和 range) 兼容性不好，自定义 JavaScript 复杂组件无语义。

### 6.2 几方面的属性

需要注意，它们不会影响页面的结构和内容。

1. 角色，navigation、article、search 等。
2. 属性，aria-required="true"，aria-labelledby="label"等。
3. 状态，aria-disabled="true"等。

   状态与属性的不同之处在于，属性不会在应用程序的整个生命周期内发生变化，而状态可以更改，通常通过 JavaScript 进行更改。

### 6.3 使用场景

#### 6.3.1 提供 HTML5 乃至之外的语义

- 添加 role

  1. 一些旧浏览器识别不了 HTML5 元素的语义
  2. 只用了 div 的场景，需要添加语义

  ```html
  <header>
    <h1>...</h1>
    <nav role="navigation">
      <ul>
        ...
      </ul>
      <form role="search">
        <!-- search form  -->
      </form>
    </nav>
  </header>

  <main>
    <article role="article">...</article>
    <aside role="complementary">...</aside>
  </main>

  <footer>...</footer>
  ```

- aria-label，为屏幕阅读器提供 label：

  ```html
  <input
    type="search"
    name="q"
    placeholder="Search query"
    aria-label="Search through site content"
  />
  ```

#### 6.3.2 动态内容提示

```html
<div id="clock" role="timer" aria-live="polite" aria-atomic="true">
  <span id="clock-hours"></span>
  <span id="clock-mins"></span>
</div>
```

- [aria-live](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-live)，指定动态区域，以及处理动态区域内容更新的优先级。
  - `off:` 默认值，更新不被通知。
  - `polite`: 用户空闲时才通知更新。
  - `assertive`: 尽快通知更新。
- [aria-atomic](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-atomic)，是否将动态区域作为一个整体对待。
- [aria-relevant](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-relevant)，什么类型的变化被读出来，增加的和 (或) 删除的。

  默认值 `additions text` 表示增加文本和节点才会通知用户。
  注意 `removals` 和 `all` 应当谨慎使用。当移除内容有重要的含义时才需要通知，比如有人离开了聊天室。

#### 6.3.3 增强键盘可访问性

WAI-ARIA 为 tabindex 扩展了新值：

- tabindex="0" — 使非语义化的元素可被 `tab` 操作。
- tabindex="-1" — 使元素可通过 JavaScript 或视觉上被鼠标点击实现聚焦。

#### 6.3.4 为非语义化的组件提供语义

1.  表单验证和错误提示

    ```html
    <div class="errors" role="alert" aria-relevant="all">
      <ul></ul>
    </div>
    ```

    - role="alert" 指定了动态区域；添加了语义。
    - aria-relevant="all" 在错误新增或移除的时候都会告诉用户。
    - 必填字段除了使用星号标识还要加上 `aria-required="true"`。
    - 其他属性：aria-required、placeholder、aria-label、aria-labelledby、aria-describedby。

    - aria-valuemin 和 aria-valuemax 用来限定最小和最大值，但是支持不是很好，可以使用 placeholder:

      ```html
      <input
        type="number"
        name="age"
        id="age"
        placeholder="Enter 1 to 150"
        aria-required="true"
      />
      ```

2.  描述非语义按钮为按钮

    - 添加了 tabindex 以及 JavaScript 的事件处理，但屏幕阅读器仍不知道它是按钮，需要添加 role="button"。
    - 再次注意一开始要选对元素！

3.  指导用户使用复杂的组件

    ```html
    <ul role="tablist">
      <li
        class="active"
        role="tab"
        aria-selected="true"
        aria-setsize="3"
        aria-posinset="1"
        tabindex="0"
      >
        Tab 1
      </li>
      <li
        role="tab"
        aria-selected="false"
        aria-setsize="3"
        aria-posinset="2"
        tabindex="0"
      >
        Tab 2
      </li>
      <li
        role="tab"
        aria-selected="false"
        aria-setsize="3"
        aria-posinset="3"
        tabindex="0"
      >
        Tab 3
      </li>
    </ul>
    <div class="panels">
      <article class="active-panel" role="tabpanel" aria-hidden="false">
        ...
      </article>
      <!-- aria-hidden="true" ：对屏幕阅读器隐藏 -->
      <article role="tabpanel" aria-hidden="true">...</article>
      <article role="tabpanel" aria-hidden="true">...</article>
    </div>
    ```

## 参考链接

> - [What is accessibility?](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/What_is_accessibility)
> - [HTML: A good basis for accessibility](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML)
> - [CSS and JavaScript accessibility best practices](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/CSS_and_JavaScript)
> - [WAI-ARIA basics](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/WAI-ARIA_basics)
