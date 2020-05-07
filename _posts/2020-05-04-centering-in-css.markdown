---
layout: post
title: CSS 居中
description: 'CSS 居中方法总结'
share: false
tags: [CSS]
image:
  feature: abstract-7.jpg
---

CSS 居中是 CSS 布局里最常见的问题，本文总结在不同情况下的实现方法及其优缺点。

## 一、水平居中

### 1. inline 或 inline-\* 元素

给块级父元素设置 `text-align: center;`，对 inline, inline-block, inline-table, inline-flex 等都奏效。

### 2. 一个 block level 元素

- 对该元素设置 `margin:0 auto;` 。
- 使用定位属性，设置父元素为相对定位，子元素为绝对定位 `left:50%`，再设置绝对子元素的 `margin-left: -元素宽度的一半;` 或 `transform: translateX(-50%);`。
- `flexbox` 布局，对父元素设置 `display: flex; justify-content: center;`。

### 3. 多个 block level 元素

多个元素水平放置

- 子元素设置 `display: inline-block;`，父元素设置 `text-align: center;`。
- `flexbox` 布局，对父元素设置 `display: flex; justify-content: center;`。

多个元素垂直放置

- 子元素设置 `margin:0 auto;`。

## 二、垂直居中

### 1. inline 或 inline-\* 元素

- 单行

  - 设置上下 `padding` 相等。
  - 设置 `line-height` 等于盒子的 `height`。

- 多行

  - 设置上下 `padding` 相等。
  - 将元素转换成 `table cell`：

    ```html
    <table>
      <tr>
        <td>
          I'm vertically centered multiple lines of text in a real table cell.
          I'm vertically centered multiple lines of text in a real table cell.
          I'm vertically centered multiple lines of text in a real table cell.
        </td>
      </tr>
    </table>
    ```

    或使用 CSS：

    ```css
    .center-table {
      display: table;
    }
    .center-table p {
      display: table-cell;
      vertical-align: middle;
    }
    ```

  - `flexbox` 布局：
    ```css
    .flex-center-vertically {
      display: flex;
      flex-direction: column;
      justify-content: center;
    }
    ```
  - 使用 `ghost element`：
    ```css
    .ghost-center {
      position: relative;
    }
    .ghost-center::before {
      content: '';
      display: inline-block;
      height: 100%;
      vertical-align: middle;
    }
    .ghost-center p {
      display: inline-block;
      vertical-align: middle;
    }
    ```

### 2. 一个 block level 元素

- 使用定位属性，设置父元素为相对定位，子元素为绝对定位 `top:50%`，再设置绝对子元素的 `margin-top: -元素高度的一半;` 或 `transform: translateY(-50%);`。
- 允许拉伸到容器高度：将元素转换为 `table` 布局。
- `flexbox` 布局：
  ```css
  .flex-center-vertically {
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
  ```

## 三、水平垂直居中

- table:
  ```html
  <table style="width: 100%;">
    <tr>
      <td style="text-align: center;">
        Unknown stuff to be centered.
      </td>
    </tr>
  </table>
  ```
  或者：
  ```html
  <div class="something-semantic">
    <div class="something-else-semantic">
      Unknown stuff to be centered.
    </div>
  </div>
  ```
  ```css
  .something-semantic {
    display: table;
    width: 100%;
  }
  .something-else-semantic {
    display: table-cell;
    text-align: center;
    vertical-align: middle;
  }
  ```
- 使用 `ghost element`。
- 组合使用前述定位属性。
- 已知宽高：设置父元素为相对定位，子元素为绝对定位，`top: 0; right: 0; bottom: 0; left: 0; margin: auto;`。
- `flexbox` 布局：

  ```html
  <div class="parent">
    <div class="child"></div>
  </div>
  ```

  ```css
  div.parent {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  ```

  或：

  ```css
  div.parent {
    display: flex;
  }
  div.child {
    margin: auto;
  }
  ```

- 使用 `grid`：

  ```html
  <div class="parent">
    <div class="child"></div>
  </div>
  ```

  ```css
  div.parent {
    display: grid;
  }
  div.child {
    justify-self: center;
    align-self: center;
  }
  ```

  或：

  ```css
  div.parent {
    display: grid;
  }
  div.child {
    margin: auto;
  }
  ```

## 四、优缺点总结

1. **负 margin**

   优点：

   - 跨浏览器兼容性好，包括 IE6-7
   - 代码量少

   缺点：

   - 不能自适应。不支持百分比属性值和 `min-/max-` 属性设置
   - 内容可能溢出容器
   - 需要补偿 `padding` 或使用 `box-sizing: border-box;`

2. **Transforms**

   优点：

   - 内容高度可变
   - 代码量少

   缺点：

   - IE8 不支持
   - 需要浏览器厂商前缀
   - 可能影响其他 `transform` 效果
   - 某些情况下导致边界或文本渲染模糊

3. **Absolute Center**

   ```css
   .Absolute-Center {
     margin: auto;
     position: absolute;
     top: 0;
     left: 0;
     bottom: 0;
     right: 0;
   }
   ```

   优点：

   - 跨浏览器兼容性好
   - 代码量少
   - 支持百分比属性值和 `min-/max-`属性
   - 不受 `padding` 影响

   缺点：

   - 内容定高
   - 设置 `overflow:auto` 防止内容溢出

4. **Table-Cell**

   优点：

   - 内容高度可变
   - 内容溢出会将父元素撑开
   - 跨浏览器兼容性好

   缺点：

   - 需要更多的 `html` 标记

5. **Inline-Block**

   优点：

   - 内容高度可变
   - 内容溢出会将父元素撑开
   - 跨浏览器兼容性好

   缺点：

   - 比较 hack，不好理解

6. **Flexbox**

   优点：

   - 内容可变宽高
   - 可实现更复杂的布局

   缺点：

   - 不兼容 IE9（包括 IE9) 之前的版本

## 参考链接

> - [Centering in CSS: A Complete Guide - CSS-Tricks](https://css-tricks.com/centering-css-complete-guide/)
> - [CSS 水平居中+垂直居中+水平/垂直居中的方法总结](https://blog.csdn.net/weixin_37580235/article/details/82317240)
> - [CSS Techniques – Absolute Horizontal And Vertical Centering In CSS](https://www.tuicool.com/articles/iQZZnq)
> - [盘点 8 种 CSS 实现垂直居中水平居中的绝对定位居中技术](https://blog.csdn.net/freshlover/article/details/11579669)
> - [Centering in the Unknown - CSS\-Tricks](https://css-tricks.com/centering-in-the-unknown/)
> - [position absolute 绝对定位 设置问题](https://www.cnblogs.com/SamWeb/p/5919529.html)
> - [The Anti-hero of CSS Layout - "display:table"](https://colintoh.com/blog/display-table-anti-hero)
