---
title: 使用 HashMap 优化数组交集计算
date: "2024-03-03T21:17:00+00:00"
published: true
feature: ""
permalink: /hashmap-usage-array-intersection/
---

在面试中，计算两个数组的交集是一个常见问题。初学者可能会想到使用两层 for 循环来解决：

```javascript
const intersection = (a, b) => {
  const res = new Set();
  for (const itemA of a) {
    for (const itemB of b) {
      if (itemA === itemB) {
        res.add(itemA);
        break;
      }
    }
  }
  return res;
};
```

然而，这种方法的时间复杂度为 O(n²)，效率较低。为提升效率，我们可以利用 HashMap 将复杂度降低到 O(n)：

```javascript
const intersection2 = (a, b) => {
  const mapOfA = a.reduce((acc, item) => {
    acc[item] = true;
    return acc;
  }, {});

  const res = new Set();
  for (const item of b) {
    if (mapOfA[item]) {
      res.add(item);
    }
  }
  return res;
};
```

通过将一个数组转换为 Map，我们只需遍历两个数组各一次，从而显著提升了计算效率。这种方法体现了算法优化中的“空间换时间”原则。
