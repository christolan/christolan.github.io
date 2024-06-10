---
title: 面向对象与函数式编程中的 event listener
date: "2024-03-08T22:24:00+00:00"
published: true
feature: ""
---

吐槽我们的 Android 项目中，我十分看不顺眼的添加监听写法。

<!-- more -->

在前端开发中，使用函数式的 JavaScript 语言，所以绝大部分的 API 设计中，都会把 event listener 设计成一个函数，在对应的时机直接调用这些函数。

```javascript
const listener = (data) => {
  console.log(data);
};

addEventListener(listener);
```

在 android 开发中，最早使用面向对象的 Java 语言，后续即便换成支持函数式编程的 Kotlin 语言，绝大部分的 event listener 还是被设计成了一个对象，在对应的时机调用这些对象上的成员函数。

```kotlin
var listener = object: Listener {
    override fun onChanged (data) {
        Log.d("data", data)
    }
}

addEventListener(listener)
```

两者之间其实并没有什么本质上的差异，无论是对象还是函数，其实都是一个引用而已，能够通过这个引用找到可以执行的回调函数就够了。

但是，从形式上来看，构造一个对象，是比构造一个函数要更加麻烦的，所以在 android 开发中，就经常会出现这样的情况：

```kotlin
class Xxxx: Listener {
    init {
        addEventListener(this)
    }

    override fun onChanged (data) {
        Log.d("data", data)
    }
}
```

在当前的对象上实现 Listener 的接口，然后直接把 this 传进去。

在比较简单的场景下，这样写确实会方便一些，但是代码写出来并不是终点，无数的后续迭代早就已经排上了日程。在 addEventListener 不多的情况下，很容易分辨哪些方法是给谁用的，但是如果 addEventListener 多了起来，并且很多 Listener 的接口你并不熟悉，那这样的代码看起来就极其难受。

甚至你无法知道，this 上的一个方法会不会被多个 Listener 同时使用，产生预料之外的效果。

直到 2024 年 3 月 9 日，我还无法理解这种操作为什么会在我们的项目里蔓延，得到几乎所有人的认可。

想看看哪一天，我会真正明白这个问题背后的原因。

又或者哪一天，我能斩钉截铁地说明：这，就是陋习！
