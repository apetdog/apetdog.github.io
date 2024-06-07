---
title: 更安全的更新对象和数组
layout: default
parent: Patterns
nav_order: 1
---

在 React 驱动的前端项目中，我们推荐采用如下的模式来确保更安全的更新对象和数组。

## 模式：将 state 视为只读的，将对象视为不可变的

我们推荐像处理数字、布尔值和字符串一样来处理对象，即将对象也视为不可变的。因此应该替换掉它们的值，而不是对它们进行修改。

这里有为什么需要这么做的原因。

- [React Docs: state 如同一张快照](https://zh-hans.react.dev/learn/state-as-a-snapshot)
- [React Docs: 把一系列 state 更新加入队列](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates)
- [React Docs: 更新 state 中的对象](https://zh-hans.react.dev/learn/updating-objects-in-state)
- [React Docs: 更新 state 中的数组](https://zh-hans.react.dev/learn/updating-arrays-in-state)

具体来说，你应该**把所有存放在 state 中的 JavaScript 对象都视为只读的**。

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
