---
layout: post
title: Service Worker 原理与实践
description: '学习 Service Worker 的原理，并简单分析 offline-plugin 插件，最后介绍一个通知的例子'
share: false
tags: [Service Worker, 缓存]
image:
  feature: abstract-10.jpg
---

Service Workers 本质上充当 Web 应用程序、浏览器与网络之间的代理服务器。该 API 旨在创建有效的离线体验，它会拦截网络请求并采取适当的动作，更新缓存资源和自定义处理响应。它还提供推送通知和后台同步的 API。

## 1. Service Worker 简介

在这之前，[Application Cache](https://developer.mozilla.org/docs/Web/API/Window/applicationCache) 曾被推出：

- 可以方便地缓存指定的资源
- 但是做了很多预设，一旦不严格遵守就会导致一些不可恢复的错误
- 已经被一些浏览器移除

而 service worker 设计的原则之一就是错误应该都是可恢复的，并且给开发者尽可能多的自由去实现自己的想法。

Service worker 是一个注册在指定源和路径下的事件驱动的 worker。它使用 JavaScript 控制关联的页面或者网站，拦截和修改导航和资源请求，以非常精细的方式缓存资源。可以完全控制应用在特定情形（最常见的情形是网络不可用）下的表现。

Service worker 运行在 worker 上下文，因此不能访问 DOM。相对于驱动应用的主 JavaScript 线程，它运行在另外的线程中，所以不会造成阻塞。需要注意的是，它被设计成完全异步的，所以不能在 service worker 中使用同步 API（如同步 XHR 和 Web Storage）。

出于安全考量，service workers 只能运行在 HTTPS 协议上，本地允许 HTTP。

## 2. Service Worker 生命周期

### 2.1 目的

设计 service worker 生命周期基于以下目的：

1. 可以实现离线优先。
2. 允许新的 service worker 在不干扰现有的 service worker 的情况下完成准备工作。
3. 确保 scope 内的页面始终由同一 service worker（或没有 service worker）控制。
4. 确保一次只有一个版本的页面（网站）在运行。（在没有 service worker 的情况下，多个 tab 共享的 storage 处理不当，可能导致错误甚至数据丢失）

### 2.2 状态介绍

Service worker 的生命周期总共有以下 6 个状态：

- parsed
- installing
- installed
- activating
- activated
- redundant

![生命周期](/images/2023-07-08-service-worker/service-worker-lifecycle.png)

#### 2.2.1 parsed

这是初始状态，代表下载并解析成功。调用 register() 时下载 service worker。 如果下载/解析失败或者运行抛出异常，则注册失败，丢弃 service worker。

```javascript
if ('serviceWorker' in navigator) {
  // Register a service worker hosted at the root of the
  // site using the default scope.
  navigator.serviceWorker.register('/sw.js', {
    scope: "/",
    updateViaCache: "none",
  }).then((registration) => {
    console.log('Service worker registration succeeded:', registration);
  }, (error) => {
    console.error(`Service worker registration failed: ${error}`);
  });
} else {
  console.error('Service workers are not supported.');
}
```

register 第一个参数是 service worker 脚本的 URL，接下来看第二个参数的两个配置项：

1. scope

   - 用来确定 service worker 可以控制的范围
   - 默认为 ./，相对于 service worker 脚本的路径
   - 默认 scope 不能大于 service worker 脚本的路径，用 [Service-Worker-Allowed Header](https://www.w3.org/TR/service-workers/#service-worker-allowed) 可解除该限制
   - 通过 navigator.serviceWorker.controller 可以查看页面是否被 service worker 控制

2. updateViaCache

    默认情况下：浏览器在检查 service worker 脚本是否有更新时会忽略 HTTP 缓存，但对于脚本中的 importScripts 会判断 HTTP 缓存。`none` 表示主脚本和 importScripts 的 HTTP 缓存都忽略。

#### 2.2.2 installing

Service worker 执行时触发 install 事件，只调用一次。

```javascript
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches
      .open("v1")
      .then((cache) =>
        cache.addAll([
          "/",
          "/index.html",
          "/style.css",
          "/app.js",
          "/image-list.js",
          "/star-wars-logo.jpg",
          "/gallery/",
          "/gallery/bountyHunters.jpg",
          "/gallery/myLittleVader.jpg",
          "/gallery/snowTroopers.jpg",
        ])
      )
  );
});
```

- install 事件通常用来获取依赖的所有核心缓存数据
  - 通过 event.waitUntil() 传入 promise，可多次调用。
  - 所有 promise 都成功 resolve，install 才算完成，否则 install 失败，丢弃 service worker。
- 在 install 完成之前，service worker 都是 installing 状态。
- Cache API 虽然是在 Service Worker 规范里定义，但是 window 和 worker 都可以使用。

> Service workers 有两个生命周期的事件：`install` 和 `activate`。对于这两个事件，Service workers 使用了 ExtendableEvent 接口，该接口有个重要的方法 waitUntil(promise)。每个 ExtendableEvent 的对象有一个关联 extend lifetime promises（promise 的数组），waitUntil 方法将 promise 添加到这个数组。
>
> 一个 installing service worker 直到 install 事件关联的 extend lifetime promises 的 promise 都成功 resolve，才算安装成功（installed，已安装）。有任何 promise 失败则安装失败。这主要用来确保在获取了所依赖的所有核心缓存之前，不会将 service worker 视为“已安装”。
>
> 同样，一个 active service worker 直到 activate 事件关联的 extend lifetime promises 的 promise 都成功 resolve，才算激活成功（activated，已激活）。这主要用来确保在更新数据库结构并删除过时的缓存条目之前，不会将任何功能事件（如 `fetch` 和 `push`）分派给 service worker。
>
> 所以，标准的用法是在 `install` 和 `activate` 事件回调里，使用 waitUntil 方法传入要做的缓存和更新工作（promise）。

#### 2.2.3 installed

install 完成，service worker 为 installed 状态。如果是第一次启用 service worker，接着会触发 activate 事件进行激活。

#### 2.2.4 activating

```javascript
self.addEventListener('activate', event => {
  clients.claim();
  console.log('v1 now ready to handle fetches!');
});
```

1. activate 事件通常用来更新数据库结构并删除过时的缓存数据。([见下文](#activate))
2. activate 事件完成后并不意味着调用 register() 的页面就会被控制
     - 默认是为了保持一致性，如果页面在没有 service worker 的情况下加载，则其子资源也是如此。[例子1](https://huxinsen.github.io/service-worker-demo-main/)（首先显示小轮自行车，再次刷新显示为山地车）
     - 可以使用 `clients.claim()` 实现 activate 完成后立即控制页面。这会在由此 service worker 控制的任何 clients 中触发 navigator.serviceWorker 上的 `controllerchange` 事件。
  
        [例子2](https://huxinsen.github.io/service-worker-demo-claimed/)（不出意外，首次应该会直接显示山地车。前提是 service worker active 事件和 clients.claim() 在加载图片之前完成，页面才会首次就显示山地车）

        > 需要注意：
        >
        > 1. 使用 service worker 和使用网络加载的一致性。
        > 2. 仅首次加载 service worker 有用。

#### 2.2.5 activated

activate 完成后，功能事件（如 fetch 和 push）才会分配给 service worker。

```javascript
self.addEventListener("fetch", event => {
  // 阻止默认行为，自己处理请求
  event.respondWith(
    caches.match(event.request).then(response => {
      // 如果发现缓存有响应则返回，否则走网络请求
      return response || fetch(event.request);
    }).catch(() => {
      return caches.match("/fallback.html");
    })
  );
});
```

fetch 是 service worker 的一个很重要的功能性事件，使用扩展了 ExtendableEvent 接口的 FetchEvent 接口。

- 通过使用 FetchEvent.respondWith 方法，可以任意修改对这些请求的响应。

#### 2.2.6 redundant

不再需要的状态，有以下几种情况：

1. install 失败。
2. 出现新的 [waiting](#waiting) service worker，旧的 waiting service worker 被置为 redundant。
3. 出现新的 active service worker，旧的 active service worker 被置为 redundant。
4. 注销 service worker，则 installing service worker、waiting service worker 和 active service worker 全被置为 redundant。

#### 2.2.7 首次使用 service worker 过程

![首次使用](/images/2023-07-08-service-worker/first-install.gif)

### 2.3 更新机制

#### 2.3.1. Service Worker 如何更新

分析完首次使用 service worker 的过程，接下来看看 service worker 检查更新时机和更新判定标准。

检查更新时机：

1. scope 内的页面的导航。
2. 推送（push）和同步（sync）等功能性事件，除非在过去 24 小时内进行了更新检查。
3. 手动调用 registration.update()。如果预期用户可能长时间使用而不刷新页面的话，可以定时调用 update()。

更新判定标准：通过与现有 service worker 对比（字节对比），如果下载的文件是新的，就会尝试进行 install。

```javascript
if ("serviceWorker" in navigator) {
  navigator.serviceWorker
    .register("/sw.js", { scope: "/" })
    .then((registration) => {
      console.log("Registration succeeded.");
      button.onclick = () => {
        registration.update();
      };
    })
    .catch((error) => {
      console.error(`Registration failed with ${error}`);
    });
}
```

#### 2.3.2 再谈 installed

install 完成，service worker 为 installed 状态。

1. 如果是第一次启用 service worker，会触发 activate 事件进行激活。
2. 但在更新的场景下， 新的 service worker 等到所有已加载的页面不再使用旧的 service worker 才会激活，这个状态称为 <a name="waiting">waiting</a>。这样设计是确保浏览器一次只有一个版本的 service worker 在运行。

> 注意：刷新时，当前页面在接收到新的响应后才销毁，那么当前的（旧的）service worker 一直控制着页面。[例子3](https://huxinsen.github.io/service-worker-demo-main/v2)（首次仍会看到山地车，因为新的 v2 缓存还未生效）

![new service worker is waiting to activate](/images/2023-07-08-service-worker/waiting-to-activate.png)

使用 `self.skipWaiting()` 可以更快地进行激活，其作用是，把当前旧的service worker 给“踢掉”，并且

- 如果新 service worker 处于 waiting 状态，则立刻被激活
- 否则会在变成 waiting 状态时被激活
- [例子4](https://huxinsen.github.io/service-worker-demo-main/v3)（不出意外，首次应该会直接显示摩托车。类似前面 clients.claim() 的例子，如果 service worker 的 active 事件在加载图片之前完成才会首次就显示摩托车）

通常在 install 事件回调里调用。将此方法与 clients.claim() 一起使用，以确保 service worker 的更新对当前 client 和所有其他活跃的 clients 立即生效。

```javascript
self.addEventListener('install', event => {
  // skipWaiting 返回立刻 resolve 为 undefined 的 promise
  // 可以安全地忽略
  self.skipWaiting();

  event.waitUntil(
    // caching etc
  );
});
```

> 注意：如果新旧 service worker 先后接管页面可能导致问题，则不要使用 skipWaiting。

#### 2.3.3 <a name="activate">再谈 activate 事件</a>

activate 事件通常用来更新数据库结构并删除过时的缓存数据。

1. 通过 event.waitUntil() 传入 promise，可多次调用。
2. 所有 promise 都成功 resolve，activate 才算完成，否则 activate 失败，丢弃 service worker。
3. cache storage API 是跟域名对应的，最好给 cache 添加前缀，避免误删其他页面（应用）的 cache。

```javascript
self.addEventListener('activate', (event) => {
  const cacheAllowlist = ['v2'];

  event.waitUntil(
    caches.forEach((cache, cacheName) => {
      if (!cacheAllowlist.includes(cacheName)) {
        return caches.delete(cacheName);
      }
    })
  );
});
```

#### 2.3.4 service worker 更新过程

![更新过程](/images/2023-07-08-service-worker/update.gif)

#### 2.3.5 注意事项

##### 2.3.5.1 避免变更 Service Worker 脚本的 URL

前面的例子 3 和 4 用了不同的 URL 是为了方便切换不同的 service worker 版本，生产环境不建议变更。

可能会导致类似下面的问题：

1. index.html 注册 sw-v1.js 为 service worker。
2. sw-v1.js 缓存并使用了 index.html，实现离线优先。
3. 更新 index.html 使用 sw-v2.js。

这样下来用户永远不会获取到 sw-v2.js，因为 sw-v1.js 返回的是它缓存的旧版本的 index.html。就会陷入一个需要更新 service worker 来更新 service worker 的死循环了。

##### 2.3.5.2 几种操作的含义

![update on load and skip waiting dev tools screenshot](/images/2023-07-08-service-worker/update-on-load-and-skip-waiting.png)

1. Update on reload：刷新页面时强制 service worker 更新并激活
2. skipWaiting：立即激活
3. 强制刷新页面（shift-reload）：绕过 service worker 控制

## 3. Service Worker 缓存和 HTTP 缓存

### 3.1 浏览器请求资源的缓存顺序

Service Worker 缓存

- 需要手动定义 fetch 事件的 handler，添加检查和返回资源的逻辑。
- 支持实现精细的缓存控制。

HTTP 缓存（浏览器缓存）

- 如果在 HTTP 缓存中发现资源，并且没有过期，浏览器会自动使用该资源。

服务器端

- 如果在 service worker 缓存和 HTTP 缓存里都没有发现资源，浏览器则通过网络请求。
- 如果资源没有在 CDN 上缓存，那么会请求到源服务器。

![缓存顺序](/images/2023-07-08-service-worker/caching-order.avif)

### 3.2 Memory Cache Layer

Memory Cache Layer 设置在 service worker 前一层，Chrome 已经支持，但目前还没有明确的规范。

![memory cache layer test screenshot](/images/2023-07-08-service-worker/memory-cache-layer.png)

## 4. [offline-plugin](https://github.com/NekR/offline-plugin) 浅析

### 4.1 install 事件

```javascript
self.addEventListener('install', function (event) {
  console.log('[SW]:', 'Install event');

  var installing = undefined;

  if (strategy === 'changed') {
    installing = cacheChanged('main');
  } else {
    installing = cacheAssets('main');
  }

  event.waitUntil(installing);
});
```

updateStrategy 有两种，all 和 changed：

- all 会删除旧版本的缓存和添加新版本的缓存。
- changed 会根据文件哈希来更新缓存
  - 判断之前是否有缓存，没有则全量添加；
  - 有的话，对比变化进行更新。

### 4.2 activate 事件

```javascript
self.addEventListener('activate', function (event) {
  console.log('[SW]:', 'Activate event');

  var activation = cacheAdditional();

  // Delete all assets which name starts with CACHE_PREFIX and
  // is not current cache (CACHE_NAME)
  activation = activation.then(storeCacheData);
  activation = activation.then(deleteObsolete);
  activation = activation.then(function () {
    if (self.clients && self.clients.claim) {
      return self.clients.claim();
    }
  });

  if (navigationPreload && self.registration.navigationPreload) {
    activation = Promise.all([activation, self.registration.navigationPreload.enable()]);
  }

  event.waitUntil(activation);
});
```

1. 缓存一条 cache 数据，记录 hashmap 和 version，方便下次对比。
2. 删除旧版本的缓存
3. self.clients.claim()
4. navigation preload

   - service worker 的启动可能阻塞网络请求
   - 允许 navigation 请求和 service worker 的启动并行

### 4.3 fetch 事件

```javascript
self.addEventListener('fetch', function (event) {
  if (event.request.method !== 'GET') {
    return;
  }

  var url = new URL(event.request.url);
  url.hash = '';

  var urlString = url.toString();

  // 处理非 external URL
  if (externals.indexOf(urlString) === -1) {
    url.search = '';
    urlString = url.toString();
  }

  var assetMatches = allAssets.indexOf(urlString) !== -1;
  var cacheUrl = urlString;

  // 处理单页面重定向
  if (!assetMatches) {
    var cacheRewrite = matchCacheMap(event.request);

    if (cacheRewrite) {
      cacheUrl = cacheRewrite;
      assetMatches = true;
    }
  }

  if (!assetMatches) {
    // 处理 navigationPreload
    // ...
    // 请求如果不在资源列表则使用浏览器默认的处理方式
    return;
  }

  // Cache handling/storing/fetching starts here
  var resource = undefined;

  if (responseStrategy === 'network-first') {
    resource = networkFirstResponse(event, urlString, cacheUrl);
  } else { // 'cache-first' otherwise
    resource = cacheFirstResponse(event, urlString, cacheUrl);
  }

  event.respondWith(resource);
});
```

1. GET 方法校验
2. 处理单页面重定向
3. navigation preload 处理
4. 请求如果不在资源列表则使用浏览器默认的处理方式，否则
   - networkFirst 先请求网络，请求失败才读取缓存。
   - cacheFirst 先读取缓存，缓存为空才请求网络。

### 4.4 message 事件

```javascript
self.addEventListener('message', function (e) {
  var data = e.data;
  if (!data) return;

  switch (data.action) {
    case 'skipWaiting':
      {
        if (self.skipWaiting) self.skipWaiting();
      }break;
  }
});
```

1. skipWaiting 消息处理
2. offline-plugin 对外提供 install 的一些 hooks，以及发送 skipWaiting 消息的方法来支持 skipWaiting。

## 5. Service Worker 实现通知

### 5.1 应用逻辑

```javascript
// 0. 请求通知权限
export function requestNotificationPermission() {
  if (!("Notification" in window)) return;

  if (Notification.permission === "granted") {
    initListener();
    return;
  }

  if (Notification.permission === "default") {
    Notification.requestPermission().then((permission) => {
      if (permission === "granted") {
        initListener();
      }
    });
  }
}

export const TAB_STATUS = {
  noTab: "noTab",
  noVisibleTab: "noVisibleTab",
  hasVisibleTab: "hasVisibleTab",
};

// 2. 注册 sw.js
export function registerServiceWorker() {
  if ("serviceWorker" in navigator) {
    window?.requestIdleCallback?.(() => {
      navigator.serviceWorker.register("/sw.js").then(
        function (registration) {
          console.log("注册成功，scope: ", registration.scope);
        },
        function (err) {
          console.log("注册失败: ", err);
        }
      );
    });
    
    // 3. 监听 sw 传递的消息
    navigator.serviceWorker.addEventListener("message", (event) => {
      const lastReadMessageId = getLastReadMessageId();

      switch (event.data.msg) {
        case TAB_STATUS.noTab:
          break;
        case TAB_STATUS.noVisibleTab:
        case TAB_STATUS.hasVisibleTab:
          // 自定义一些操作，比如展示未读消息
          ee.emit("displayUnreadMessages", lastReadMessageId);
          break;
        default:
          break;
      }
    });
  }
}

// 4. 发送通知
export function pushNotification(text: string) {
  if (window?.Notification?.permission !== "granted") return;

  const visibleTabCount = getVisibleTabCount();
  if (visibleTabCount > 0) return;

  // 由 master tab 发送通知
  if (!sharedAgentChat.isMaster) return;

  navigator?.serviceWorker.getRegistration().then(function (reg) {
    reg?.showNotification("PN Title", {
      body: text, // 消息主体
      data: { // 任意类型的数据
        url: window.location.origin + '#pn', // 比如传了 url，用 #pn 来标记从通知进入
      },
    });
  });
}
```

### 5.2 Service Worker 逻辑

```javascript
self.addEventListener("activate", (event) => {
  event.waitUntil(self.clients.claim());
});

// 1. 准备 sw.js 文件 监听通知点击事件
self.addEventListener("notificationclick", (e) => {
  e.notification.close(); // 关闭通知
  e.waitUntil( // 获取所有 Window clients
    self.clients.matchAll({ type: "window" }).then((clientsArr) => {
      if (clientsArr.length) {
        const hasVisibleTab = clientsArr.some((windowClient) => {
          // 已有 visible tab
          if (windowClient.visibilityState === "visible") {
            windowClient.postMessage({ msg: "hasVisibleTab" });
            return true;
          }

          return false;
        });

        if (!hasVisibleTab) {
          clientsArr[0].postMessage({ msg: "noVisibleTab" });
          clientsArr[0].focus();
        }
      } else {
        self.clients
          .openWindow(e.notification.data.url) // 通过 data.url 打开新tab
          .then((windowClient) => {
            if (windowClient) {
              windowClient.postMessage({ msg: "noTab" });
              windowClient.focus();
            }
          });
      }
    })
  );
});
```

## 参考链接

> - [Service Workers](https://www.w3.org/TR/service-workers/)
> - [Service Worker API \- Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
> - [The service worker lifecycle](https://web.dev/service-worker-lifecycle/)
> - [Speed up service worker with navigation preloads](https://web.dev/navigation-preload/)
