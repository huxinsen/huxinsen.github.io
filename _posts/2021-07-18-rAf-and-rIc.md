---
layout: post
title: 搞懂帧动画和闲时回调
description: '学习 requestAnimationFrame 和 requestIdleCallback 的原理和用法'
share: false
tags: [JS动画]
image:
  feature: abstract-5.jpg
---

之前经常看到 `requestAnimationFrame` 和 `requestIdleCallback`，但对其原理却是一知半解，这次深入学习做个笔记。

## 1. 动画的原理

`视觉暂留`（英文：Persistence of vision）也称为正片后像，是光对视网膜所产生的视觉，在光停止作用后，仍然保留一段时间的现象。是现代影视、动画等视觉媒体制作和传播的根据。

动画，顾名思义，就是能“动”的画。由于人眼的视觉暂留效应，当前位置的图像在大脑的印象还没消失，紧接着图像又被移到了下一个位置，因此看到图像在流畅地移动，即视觉效果上形成的动画。要达成最基本的视觉暂留效果至少需要 10 帧/秒 (frame per second, FPS)。现代电影的帧率通常在 24 帧/秒。

## 2. CSS 动画 vs JavaScript 动画

网页的动画分为 CSS 动画和 JavaScript 动画，它们有各自适用的场景。

CSS 动画：

- 当 UI 元素具有较小的自包含状态时，请使用 CSS。
- 使用 CSS 动画进行更简单的“一次性”转换，例如切换 UI 元素状态。

JavaScript 动画

- 如果需要对动画进行大量的控制时，请使用 JavaScript。
- 使用 JavaScript 动画实现弹跳、停止、暂停、倒带或减速等高级效果。
- 如果选择使用 JavaScript 制作动画，请使用 Web Animations API 或框架。
- 如果想手动控制整个动画时，直接使用 requestAnimationFrame。

## 3. 传统实现 JavaScript 动画的方法

传统上使用 `setTimeout` 或 `setInterval`，通过设置一个间隔时间来不断地改变图像的位置，从而达到动画效果。

看一个小例子：

```javascript
const adiv = document.getElementById('mydiv');
let xpos = 0;

window.setTimeout(function move() {
  xpos += 1; // 每次移动 div 1px
  adiv.style.transform = 'translateX(' + xpos + 'px)';
  window.setTimeout(move, 10); // 每 10ms 移动一次
}, 10);
```

通过不断调用 `setTimeout` 每次移动元素 1px，实现动画。代码看起来没有问题，但实际运行起来发现动画是有卡顿的，不够流畅。可以看实际效果：[setTimeout animation](https://codepen.io/huxinsen/pen/rNmmgEK)。

通常情况下，使用 `setTimeout` 制作的动画卡顿的原因主要有两个：

1. 每次的时间间隔不一致。由于 `setTimeout` 是异步的，需要等待同步代码执行完成再执行，所以实际执行时间可能晚于预期。
2. 频率和屏幕刷新频率不一致。常见的显示器的刷新频率为 60 FPS，高刷的显示器能达到 90 或 120 FPS。当 `setTimeout` 调用的频率和屏幕刷新频率不一致，可能会出现丢帧的现象，如下图所示。

   ![丢帧](/images/2021-07-18-rAf-and-rIc/dropped-frames.png)

<h2 name="event-loop">4. Event Loop 主要流程</h2>

`setTimeout` 导致丢帧，造成动画卡顿的根本原因是动画刷新的频率和屏幕的刷新频率不一致。那怎么解决呢？答案就是使用 `requestAnimationFrame`。

`window.requestAnimationFrame()` 方法告诉浏览器你想执行一个动画，并且请求浏览器在下一次重绘之前调用指定的函数来更新动画。为了更好地理解`下一次重绘`，我们结合一下 `event loop` 的主要流程来看。

1. 从任务队列中取出一个宏任务并执行。
2. 检查微任务队列，执行并清空微任务队列，如果在微任务的执行中又加入了新的微任务，也会在这一步一起执行。
3. 进入更新渲染阶段，判断是否需要渲染。这里有一个 `rendering opportunity` 的概念，也就是说不一定每轮 `event loop` 都会对应一次浏览器渲染，而是要根据屏幕刷新率、页面性能、页面是否在后台运行来共同决定，通常来说这个渲染间隔是固定的（对于 60 FPS 的显示器来说，每 16.66ms 渲染一次）。（所以多个任务很可能在一个渲染间隔内执行）
   - 浏览器会尽可能地保持帧率稳定，例如页面性能无法维持 60 FPS 的话，浏览器会选择 30 FPS 的更新速率，而不是偶尔丢帧。
   - 如果浏览器上下文不可见（比如后台运行），那么页面会降低到 4 FPS 甚至更低。
   - 如果满足以下条件，会跳过渲染：
     1. 浏览器判断更新渲染不会带来视觉上的改变。
     2. `map of animation frame callbacks` 为空，也就是 `requestAnimationFrame` 回调为空。
   - 如果跳过渲染，那么下面的几步也不会继续执行：
     1. 对于需要渲染的文档，如果是顶层浏览上下文，执行 `autofocus` 的计算。
     2. 对于需要渲染的文档，如果窗口的大小发生了变化，执行 `resize` 回调。
     3. 对于需要渲染的文档，如果页面发生了滚动，执行 `scroll` 回调。
     4. 对于需要渲染的文档，检查媒体查询及执行 `change` 回调。
     5. 对于需要渲染的文档，更新动画（Web Animations）及执行动画事件的回调。
     6. 对于需要渲染的文档，执行全屏相关事件的回调。
     7. 对于需要渲染的文档，执行 **`requestAnimationFrame`** 的回调。
     8. 对于需要渲染的文档，执行 `IntersectionObserver` 的回调。
     9. 报告 `first-paint` 和 `first-contentful-paint` 时间戳。
     10. 对于需要渲染的文档，**重新渲染**，绘制用户界面。
4. 如果任务队列和微任务队列都为空，且此轮 `event loop` 跳过渲染，执行空闲回调，也就是 **`requestIdleCallback`** 的回调。
5. 重复以上步骤

可以看到 `requestAnimationFrame` 不是任务，也不是微任务，而是在重新渲染屏幕之前调用我们指定的回调函数。

## 5. requestAnimationFrame 用法

语法：`window.requestAnimationFrame(callback)`

- callback：回调函数，被调用时会传入执行的时间戳。
- 返回值：大于 0 的 Request ID，可传入 `window.cancelAnimationFrame()` 取消对应的请求。

```javascript
const adiv = document.getElementById('mydiv');
let xpos = 0;

function movediv(timestamp) {
  xpos += 1;
  adiv.style.transform = 'translateX(' + xpos + 'px)';
  // 再次调用，准备下一帧
  window.requestAnimationFrame(movediv);
}

// 调用 requestAnimationFrame，传入操作动画的函数
window.requestAnimationFrame(movediv);
```

这次使用 `requestAnimationFrame` 实现前面的例子，使用方法上和 `setTimeout` 差不多，不过不需要传入时间，因为浏览器会自动计算下次渲染的时间。可以查看实际效果 [requestAnimationFrame animation](https://codepen.io/huxinsen/pen/GRmmbKp)，动画更加平滑顺畅。

前面的例子很简单，而实际使用场景通常比较复杂，往往需要实现在特定时间内完成一段动画。这就要使用回调函数的时间戳参数了，看下面的例子：

```javascript
const element = document.getElementById('some-element-you-want-to-animate');
let start;

function step(timestamp) {
  if (start === undefined) start = timestamp;
  const elapsed = timestamp - start;

  // Math.min() 确保元素最终停在 200px
  element.style.transform =
    'translateX(' + Math.min(0.1 * elapsed, 200) + 'px)';

  if (elapsed < 2000) {
    // 2s 后停止动画
    window.requestAnimationFrame(step);
  }
}

window.requestAnimationFrame(step);
```

通过时间戳可以计算出当前时间对应的动画位置，这样每一帧都确定下来了。

## 6. requestAnimationFrame polyfill

`requestAnimationFrame` 目前的[兼容性](https://caniuse.com/?search=requestAnimationFrame)很高了，如下是一个 [polyfill 实现](https://github.com/darius/requestAnimationFrame)，可以参考:

```javascript
if (!Date.now)
  Date.now = function () {
    return new Date().getTime();
  };

(function () {
  'use strict';

  var vendors = ['webkit', 'moz'];
  for (var i = 0; i < vendors.length && !window.requestAnimationFrame; ++i) {
    var vp = vendors[i];
    window.requestAnimationFrame = window[vp + 'RequestAnimationFrame'];
    window.cancelAnimationFrame =
      window[vp + 'CancelAnimationFrame'] ||
      window[vp + 'CancelRequestAnimationFrame'];
  }
  if (
    /iP(ad|hone|od).*OS 6/.test(window.navigator.userAgent) || // iOS6 is buggy
    !window.requestAnimationFrame ||
    !window.cancelAnimationFrame
  ) {
    var lastTime = 0;
    window.requestAnimationFrame = function (callback) {
      var now = Date.now();
      var nextTime = Math.max(lastTime + 16, now);
      return setTimeout(function () {
        callback((lastTime = nextTime));
      }, nextTime - now);
    };
    window.cancelAnimationFrame = clearTimeout;
  }
})();
```

## 7. requestIdleCallback 为何物？

`window.requestIdleCallback()` 方法将指定的函数加入队列，该函数将在浏览器空闲时期被调用。怎么算空闲呢？从 [`event loop` 一节](/#event-loop)可以看到:

> 4\. 如果任务队列和微任务队列都为空，且此轮 `event loop` 跳过渲染，执行空闲回调，也就是 **`requestIdleCallback`** 的回调。

看两种空闲时期的场景：

![帧间空闲时期](/images/2021-07-18-rAf-and-rIc/inter-frame-idle-period.png)

第一种是帧间空闲时期，即把确定的一帧提交给屏幕和开始处理下一帧之间的时间。这种空闲时期通常很短（比如 60 FPS 的设备上，小于 16ms）。

![没有待定的帧要渲染的空闲时期](/images/2021-07-18-rAf-and-rIc/idle-period-when-no-pending-frame-updates.png)

另一种是浏览器没有屏幕刷新而空闲。这种情况下，浏览器可能没有任务要执行，空闲时期就会持续下去。为了避免不可预测的任务（例如处理用户输入）引起用户可察觉的延迟（用户对耗时小于 100ms 的响应感觉是瞬时的），这些空闲时期的长度限制为最大值 50ms。

在 [RAIL](https://web.dev/rail/) (Response, Animation, Idle and Load) 模型中，提到`在 50ms 内处理事件`。为什么不是 100ms 呢？

![空闲任务影响输入响应时间的最大限制](/images/2021-07-18-rAf-and-rIc/idle-tasks-affect-input-response-budget.png)

研究表明，人们通常认为 100 毫秒内对用户输入的响应是瞬时的。空闲时期的长度限制为最大值 50ms，那么在空闲任务开始后立即发生用户输入，浏览器有剩余的 50ms 响应用户输入。考虑到这一点，安全的做法是假设只有剩余的 50ms 可用于实际输入处理。

## 8. requestIdleCallback 用法

语法：`window.requestIdleCallback(callback[, options])`

- callback：回调函数，被调用时会传入 `IdleDeadline` 对象，包含 `timeRemaining` 函数（返回剩余时间，上限为 50ms），`didTimeout` 属性。
- options：配置对象，支持 `timeout` 属性，浏览器必须在该时间内执行回调函数。
- 返回值：大于 0 的 Request ID，可传入 `window.cancelIdleCallback()` 取消对应的请求。

使用注意事项：

- 执行低优先级的任务：如发送统计数据（下面的例子）。
- 页面后台运行时，浏览器会对空闲时期的生成进行节流。
- 避免操作 DOM。
  - 当回调在帧末尾触发（前面场景一，当前的帧已经提交了，样式和布局已经计算完毕），如果操作 DOM 的话，之前的样式和布局计算就无效了。
  - 另外改变 DOM 带来的对时间的影响也不可预计，这样容易超出浏览器给的执行时间限制。
  - 由前文，`requestAnimationFrame` 才是操作 DOM 的好地方。
- 避免执行运行时间不可预计的任务。

```javascript
var eventsToSend = [];
var isRequestIdleCallbackScheduled = false;

function onNavOpenClick() {
  // Animate the menu.
  menu.classList.add('open');
  // Store the event for later.
  eventsToSend.push({
    category: 'button',
    action: 'click',
    label: 'nav',
    value: 'open'
  });

  schedulePendingEvents();
}

function schedulePendingEvents() {
  // Only schedule the rIC if one has not already been set.
  if (isRequestIdleCallbackScheduled) return;

  isRequestIdleCallbackScheduled = true;

  if ('requestIdleCallback' in window) {
    // Wait at most two seconds before processing events.
    requestIdleCallback(processPendingAnalyticsEvents, { timeout: 2000 });
  } else {
    processPendingAnalyticsEvents();
  }
}

function processPendingAnalyticsEvents(deadline) {
  // Reset the boolean so future rICs can be set.
  isRequestIdleCallbackScheduled = false;

  // If there is no deadline, just run as long as necessary.
  // This will be the case if requestIdleCallback doesn’t exist.
  if (typeof deadline === 'undefined')
    deadline = {
      timeRemaining: function () {
        return Number.MAX_VALUE;
      }
    };

  // Go for as long as there is time remaining and work to do.
  while (deadline.timeRemaining() > 0 && eventsToSend.length > 0) {
    var evt = eventsToSend.pop();
    ga('send', 'event', evt.category, evt.action, evt.label, evt.value);
  }

  // Check if there are more events still to send.
  if (eventsToSend.length > 0) schedulePendingEvents();
}
```

## 9. requestIdleCallback shim

`requestIdleCallback` 的特性无法真正模拟出来，所以只有兼容的写法：

```javascript
window.requestIdleCallback =
  window.requestIdleCallback ||
  function (cb) {
    var start = Date.now();
    return setTimeout(function () {
      cb({
        didTimeout: false,
        timeRemaining: function () {
          return Math.max(0, 50 - (Date.now() - start));
        }
      });
    }, 1);
  };

window.cancelIdleCallback =
  window.cancelIdleCallback ||
  function (id) {
    clearTimeout(id);
  };
```

## 10. 总结

- `requestAnimationFrame` 的回调在重新渲染屏幕之前执行，非常适合用来做动画。
- `requestIdleCallback` 的回调在空闲时期执行，并且是否有空执行要看浏览器的调度，如果要求回调必须在某个时间内执行，请使用 `timeout` 参数。

## 参考链接

> - [CSS Versus JavaScript Animations  \|  Web Fundamentals](https://developers.google.com/web/fundamentals/design-and-ux/animations/css-vs-javascript)
> - [Window\.requestAnimationFrame\(\) \- Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
> - [HTML Standard](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)
> - [darius/requestAnimationFrame: Polyfill for requestAnimationFrame/cancelAnimationFrame](https://github.com/darius/requestAnimationFrame)
> - [window\.requestIdleCallback\(\) \- Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback)
> - [Using requestIdleCallback  \|  Web  \|  Google Developers](https://developers.google.com/web/updates/2015/08/using-requestidlecallback)
> - [Cooperative Scheduling of Background Tasks](https://www.w3.org/TR/requestidlecallback/)
> - [Measure performance with the RAIL model](https://web.dev/rail/)
> - [Background Tasks API \- Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/Background_Tasks_API)
