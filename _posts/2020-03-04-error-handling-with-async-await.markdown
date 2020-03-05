---
layout: post
title: async/await 错误处理
description: 'async/await 错误处理机制'
share: false
tags: [JavaScript]
image:
  feature: abstract-5.jpg
---

async/await 基本规则

- `async function` 定义一个异步函数，其返回值是一个 Promise 对象。
- `await` 等待 Promise 对象或其他任何值，返回 Promise 对象的 `fulfilled` 值或其他任何值。
- `await` 只能用在 `async` 函数体内，暂停 `async` 函数执行，等到结果后继续执行。如果等待的 Promise 处理异常（rejected），则抛出 `rejected` 值。

## 一、`try/catch`

使用 `try/catch` 处理异步错误

```javascript
async function run() {
  try {
    await Promise.reject(new Error('Oops'))
  } catch (error) {
    error.message // Oops
  }
}

run()
```

`try/catch` 当然也可以处理同步错误：

```javascript
async function run() {
  try {
    throw new Error('sync')
    await Promise.reject(new Error('async'))
  } catch (error) {
    error.message // sync
  }
}

run()
```

然而，`try/catch` 不是万能的。例如，你返回了一个 rejected Promise。

```javascript
async function run() {
  try {
    return Promise.reject(new Error('Oops'))
  } catch (error) {
    console.log('This will **not** print')
  }
}

// Unhandled promise rejection!
run()
```

可以使用 `return await` 解决这个问题：

```javascript
async function run() {
  try {
    return await Promise.reject(new Error('Oops'))
  } catch (error) {
    console.log(error.message) // Oops
  }
}

run()
```

## 二、`catch()`

另一个常用方法是使用 `catch()`

```javascript
function myAsyncFunction() {
  return Promise.reject(new Error('Oops'))
}

async function run() {
  // `err` will be `null` if the promise fulfilled, or an error if
  // the promise rejected
  const err = await myAsyncFunction()
    .then(() => null)
    .catch(err => err)
}

run()
```

既然 `async` 函数返回的是 Promise 对象，我们可以调用 `.catch()`。这是更推荐的做法，因为它可以处理所有错误，包括同步错误、异步错误以及返回 rejected promise 的错误。

```javascript
async function syncError() {
  throw new Error('sync')
}

async function asyncError() {
  await Promise.reject(new Error('async'))
}

async function returnRejected() {
  return Promise.reject(new Error('returnRejected'))
}

syncError().catch(err => console.log(err.message)) // 'sync'
asyncError().catch(err => console.log(err.message)) // 'async'
// 'returnRejected'
returnRejected().catch(err => console.log(err.message))
```

## 参考链接

> - [Async Functions in JavaScript](http://thecodebarbarian.com/async-functions-in-javascript.html)
> - [await - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
