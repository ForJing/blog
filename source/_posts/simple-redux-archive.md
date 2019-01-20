---
title: redux最简实现
date: 2019-01-20 10:37:30
categories:
  - 前端
tags:
  - React
  - Redux
---

本文实现一个最简单的 redux。

<!--more-->

我们是如何使用 redux 的？

```js
//  counter reducer
function counter(state = 0, action) {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    default:
      return state;
  }
}

// main.js
import { createStore } from "redux";
const store = createStore(counter);
const { getState, dispatch, subscribe } = store;

const unsubscribe = store.subscribe(() => console.log(store.getState()));

store.dispatch({ type: "INCREMENT" });
store.dispatch({ type: "DECREMENT" });

unsubscribe();
```

下面是最简实现

```js
const createStore = reducer => {
  let state;
  let listeners = [];

  const getState = () => state;

  const subscribe = listener => {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(i => i != listener);
    };
  };

  const dispatch = action => {
    state = reducer(state, action);
    listeners.forEach(listener => {
      listener();
    });
  };

  // 初始化state, 如果没有type， 会走reducer的switch->default分支返回一个默认值
  dispatch({});

  return { getState, dispatch, subscribe };
};

function counter(state = 0, action) {
  switch (action.type) {
    case "INCREMENT":
      return state + 1;
    case "DECREMENT":
      return state - 1;
    default:
      return state;
  }
}

const store = createStore(counter);

const unsubscribe = store.subscribe(() => console.log(store.getState()));

store.dispatch({ type: "INCREMENT" });
store.dispatch({ type: "INCREMENT" });
store.dispatch({ type: "INCREMENT" });
store.dispatch({ type: "DECREMENT" });

unsubscribe();
store.dispatch({ type: "DECREMENT" });

console.log("----------");
console.log(store.getState());
```

运行结果如下

```
1
2
3
2
"-------------"
1
```
