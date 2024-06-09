---
title: HashMap 的使用：数组取交集
date: "2024-03-03T21:17:00+00:00"
published: true
feature: ""
---

请问如何计算出两个数组的交集？

<!-- more -->

这是我在面试中非常喜欢问的问题，这个问题几乎所有人都能给出自己的回答，因为最不济你也知道，两层 for 循环就可以：

```javascript
const intersection = (a, b) => {
  const res = new Set();

  for (itemA of a) {
    for (itemB of b) {
      if (itemA === itemB) {
        res.add(itemA);
        break;
      }
    }
  }

  return res;
};
```

但是两层 for 循环意味着复杂度为 n 方，这样的计算方式显然是效率极低的。如何将复杂度降低到 2n 呢？答案就是将其中的一个数组转成 Map：

```javascript
const intersection2 = (a, b) => {
  const mapOfA = a.reduce((acc, item) => {
    acc[item] = true;
    return acc;
  }, {});

  const res = new Set();

  for (item of b) {
    if (mapOfA[item]) {
      res.add(item);
    }
  }

  return res;
};
```

这样就只需要两个数组各遍历一遍，就完成了取交集的功能。其中蕴含的，就是算法优化中的一个很基础的思想：空间换时间。
