---
title: 深入理解useEffect
date: 2020-06-29 23:45:06
tags: JavaScript
---

# 深入理解 useEffect 的同步概念

在一开始学习 hooks 的时候，由于以前的思维定势，总是会把相关概念往生命周期上靠，这样学起来很是疑惑，自从经过某位高人指点突然豁然开朗，感觉自己貌似领悟了真谛，于是乎就简单记录一下一些关于 hooks 的想法，其中大部分是 useEffect 的，这个 hook 很重要，很多派生 hook 都是基于 useEffect 建立的，接下来也可以学习一下阿里出的一个 hooks 库叫 [ahooks](https://github.com/alibaba/hooks)

> 学习 hooks 时，最好不要去想象成跟生命周期一样的逻辑，这样很容易使自己掉坑里，hooks 跟生命周期是完全不同的两码事情，可以当作新的思想去学习看待。

为了方便理解，先约定好每次 render 称之为“一帧”

## Capture Value

我们可以把具有在某一帧中的数据完全独立的数据称之为"Capture Value"
以下是一个官方的计数器组件例子

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

在某一帧下 count 可能是 1、2、3，所以这个 count 没有任何的绑定关系，在函数中算是一个常量，每一次渲染都会重新获取最新的 count。推己及人，组件的 props、state 也是属于“某一帧的”，也就是说，在某一次渲染行为中，props、state 都不会发生变化

我们可以把这种特性叫做“Capture Value”特性，即每一次函数组件被渲染可以看成是一个全新的开始

同理 useEffect 也具有“Capture Value”特性，即每个 effect 获取的 count 都来自于它属于的那一帧。理解了这个概念我们就对 hooks 的工作机制有了更深一步的理解
比如以下例子：

```JSX
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```

由于 useEffect 具有“Capture Value”特性，所以 count 每次获取的都是 0，这样后续的 setCount 就完全不起作用了

## 避免不必要的 useEffect 调用

示例组件：

```jsx
const Extra = ({ text = "" }) => {
  return <div>{text}</div>;
};
```

在老版本的 react 中，我们可能给父组件的 state 中 text 设置成'',然后通过接口获取或者其他操作在 componentDidMount 中把 text 修改成了'hello',然后通过其他操作在 componentDidUpdate 把 text 变成了'hi',这样 从''=>'hello'=>'hi',react 会对改变的 props 进行检测，发现只是改变了 text，则只是进行了如下改变：

```JS
domNode.innerText = 'hi';
```

但是 effect 则不同，effect 在每次发生 state 变化使其重新渲染后都会被重新调用，这样在上述例子中就会出现三次 effect，有时候我们可能改变了 state a 却触发了其他 effect，所以 useEffect 设计了第二个参数：

```jsx
useEffect(() => {
  document.text = text;
}, [text]);
```

这样只有当 text 发生变化的时候才会执行 effect

初学过程中很容易忽略第二个参数导致程序死循环，因为触发了 effect 之后 callback 中继续触发的其他变化会导致 effect 再次重现

## 更好的处理边缘情况

在普通 class 组件中，我们对一个数据的操作会散落在各个生命周期函数中

1. componentDidMount 中获取数据
2. componentDidUpdate 中根据是否发生变化来更新数据
3. unmount 的时候销毁数据

而在 hooks 中，我们只需要关系数据是否发生变化，从而不在拘泥于各种生命周期的处理，这样就会避免同一个 action 在 componentDidMount、componentDidUpdate 中调用两次的尴尬场景

> 由于 useEffect 回调函数的调用具有滞后性，不会阻塞页面渲染，所以性能表现会更好一些

## 不推荐在 useEffect 使用 async/await 等异步操作

数据竞争在业务上也是一种很常见的问题，比如 componentDidMount 中进行了异步操作调用了一个接口，但是很慢，在 componentDidUpdate 也调用了一次，第二次比第一次先返回就出现问题了。

在 useEffect 中进行类似的操作时，react 会发出警告

对此我找到了一篇关于异步操作的文章，还没有看完先贴在[这](https://www.robinwieruch.de/react-hooks-fetch-data)

## 总结

hooks 作为 react 新的 api，打破了无状态组件这一概念，使得所有组件都可以拥有内部状态，但是 hooks 的加入使得 react 引入了一个新的概念：同步，所以我认为 react 的设计思路已经更换了方向。所以学起来吧，拥抱新的 react
