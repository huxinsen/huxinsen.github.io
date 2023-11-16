---
layout: post
title: 批量上报
description: '前端批量上报代码片段'
share: false
tags: [上报]
image:
  feature: abstract-12.jpg
---

前端批量上报代码片段

```javascript
function report(data) {
  if (window.requestIdleCallback) {
    window.requestIdleCallback?.(() => {
      navigator.sendBeacon('/report/v1/tr', JSON.stringify(data));
    })
  } else {
    navigator.sendBeacon('/report/v1/tr', JSON.stringify(data));
  }
}

var pendingQueue = [];
var isPendingQueueScheduled = false;

function schedulePendingQueue() {
  if (isPendingQueueScheduled) {
    return;
  }

  isPendingQueueScheduled = true;

  setTimeout(processPendingQueue, 500);
}

function processPendingQueue() {
  isPendingQueueScheduled = false;

  if (pendingQueue.length > 0) {
    var data = pendingQueue;
    pendingQueue = [];

    var countData = [];
    var timeData = [];
    var hyperData = [];

    data.forEach(function (item) {
      switch (item.type) {
        case 'count':
          countData.push(item.value);
          break;
        case 'time':
          timeData.push(item.value);
          break;
        case 'hyper':
          hyperData.push(item.value);
          break;
        default:
          break;
      }
    });

    var params = {};

    if (countData.length > 0) {
      params.count = countData;
    }

    if (timeData.length > 0) {
      params.time_costs = timeData;
    }

    if (hyperData.length > 0) {
      params.hyper = hyperData;
    }

    if (Object.keys(params).length === 0) {
      return;
    }

    report(params);
  }
}

function putInReportQueue(reportItem) {
  pendingQueue.push(reportItem);

  schedulePendingQueue();
}

putInReportQueue({ type: 'count', value: { label: 'html_pv_total' } });
```
