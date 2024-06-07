---
title: 更安全的更新对象和数组
layout: default
parent: Patterns
nav_order: 1
---

# 更安全的更新对象和数组
{: .no_toc }

在 React 驱动的前端项目中，我们推荐采用在没有 mutation 的前提下更新对象和数组。
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## 模式：将 state 视为只读的，将对象视为不可变的

我们推荐像处理数字、布尔值和字符串一样来处理对象，即将对象也视为不可变的。因此应该替换掉它们的值，而不是对它们进行修改。

这里有为什么需要这么做的原因。

- [React Docs: state 如同一张快照](https://zh-hans.react.dev/learn/state-as-a-snapshot)
- [React Docs: 把一系列 state 更新加入队列](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates)
- [React Docs: 更新 state 中的对象](https://zh-hans.react.dev/learn/updating-objects-in-state)
- [React Docs: 更新 state 中的数组](https://zh-hans.react.dev/learn/updating-arrays-in-state)

具体来说，我们应该**把所有存放在 state 中的 JavaScript 对象都视为只读的**。

```js
const [position, setPosition] = useState({
  x: 0,
  y: 0,
});

// ❌ 错误的做法，直接修改对象
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}

// ✅ 正确的做法，创建一个新对象
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

## 模式：使用展开语法复制对象，但要记住它的复制只有一层

通常，我们会只想要更新表单中的一个字段，其他的字段仍然使用之前的值。这时候我们推荐使用 `...` [对象展开](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals)语法，以避免单独复制每个属性。

```js
const [person, setPerson] = useState({
  firstName: 'Docs',
  lastName: 'Apetdog',
  email: 'docs@apetdog.com'
});

// ⭕️ 不推荐的做法，单独复制每个字段
setPerson({
  firstName: e.target.value, // 要更新的字段
  lastName: 'Apetdog',
  email: 'docs@apetdog.com'
});

// ✅ 推荐的做法，使用展开语法
setPerson({
  ...person, // 复制之前的所有字段
  firstName: e.target.value,
});

// ✅ 推荐的做法，动态命名
setPerson({
  ...person,
  [e.target.name]: e.target.value, // 同时处理多个字段
});
```

`...` 展开语法本质是“浅拷贝”，因此我们需要多次使用展开语法来更新一个嵌套属性。

```js
const [person, setPerson] = useState({
  firstName: 'Docs',
  lastName: 'Apetdog',
  email: 'docs@apetdog.com',
  artwork: {
    title: 'Logo',
    type: 'symbol',
    image: 'https://apetdog.github.io/assets/images/logo.svg',
  }
});

// ✅ 推荐做法
setPerson({
  ...person,
  artwork: {
    ...person.artwork,
    type: 'art', // 更新 type 的值
  }
});
```

这虽然看起来有点冗长，但对于很多情况都能有效地解决问题。

## 模式：永远以返回一个新数组的方式来更新数组

下面是常见数组操作的参考表。当操作 React state 中的数组时，我们需要避免使用左列的方法，而首选右列的方法：

|  | 避免使用 (会改变原始数组) | 推荐使用 (会返回一个新数组） |
| --- | --- | --- |
| 添加元素 | `push`，`unshift` | `concat`，`[...arr]` 展开语法|
| 删除元素 | `pop`，`shift`，`splice` | `filter`，`slice`|
| 替换元素 | `splice`，`arr[i] = ...` 赋值 | `map` |
| 排序 | `reverse`，`sort` | 先将数组复制一份 |

### 使用 `...` 展开运算来向数组中添加元素

使用展开操作就可以完成 push() 和 unshift() 的工作，将新元素添加到数组的末尾和开头。

```js
const [artists, setArtists] = useState([]);

// ❌ 错误的做法，直接修改数组
onClick={() => {
  artists.push({
    id: nextId++,
    name: name,
  });
};

// ✅ 推荐的做法，使用展开语法
setArtists([
  ...artists, // 新数组包含原数组的所有元素
  { id: nextId++, name: name } // 并在末尾添加了一个新的元素
]);
// or
setArtists([
  { id: nextId++, name: name },
  ...artists // 将原数组中的元素放在末尾
]);
```

### 使用 `filter` 过滤元素来从数组中删除元素

创建一个新的数组，该数组由那些 ID 与 artists.id 不同的 artists 组成。

```js
// ✅ 推荐的做法，使用 filter 语法
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

### 使用 `map` 来替换数组中的元素

要替换一个元素，请使用 map 创建一个新数组。

```js
const initialCounters = [
  0, 0, 0
];

const [counters, setCounters] = useState(
  initialCounters
);

// ✅ 推荐的做法，使用 map 语法
const nextCounters = counters.map((c, i) => {
  if (i === index) {
    // 递增被点击的计数器数值
    return c + 1;
  } else {
    return c;
  }
});
setCounters(nextCounters);
```

### 使用 `...` 展开运算和 `slice()` 来向数组特定位置插入元素

```js
const [artists, setArtists] = useState(
  initialArtists
);

const nextArtists = [
  // 插入点之前的元素
  ...artists.slice(0, insertAt),
  // 新的元素
  { id: nextId++, name: name },
  // 插入点之后的元素：
  ...artists.slice(insertAt),
];
```

## 模式：创建拷贝值来更新数组内部的对象

当更新一个嵌套的 state 时，我们需要从想要更新的地方创建拷贝值，一直这样，直到顶层。

```js
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // 创建包含变更的*新*对象
    return { ...artwork, seen: nextSeen };
  } else {
    // 没有变更
    return artwork;
  }
}));
```

## 模式：使用 Immer 编写简洁的更新逻辑

[Immer（德语为：always）](https://immerjs.github.io/immer)是一个小型包，可让我们以更方便的方式使用不可变状态。在 React 中可以使用 [use-immer](https://github.com/immerjs/use-immer)。

这是因为我们并不是在直接修改原始的 state，而是在修改 Immer 提供的一个特殊的 `draft` 对象。同理，我们也可以为 `draft` 的内容使用 `push()` 和 `pop()` 这些会直接修改原值的方法。
