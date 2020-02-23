---
title: TypeScript————类型相关总结
date: 2020-01-20 14:04:59
tags: JavaScript
---

### 1. interface 和 type

interface 的特点

1. interface 和同名 interface、class 自动聚合
2. 自身只能表示 object/class/function 的类型

type 特点

1. 不局限于 object/class/function
2. 不可以重名
3. 更复杂的类型操作

所以一般是自用一般是 type 提供 api 相关用 interface

### 2. & | 操作符

只是写法上跟位运算符相同，实际意义为：

```typescript
interface IA {
  a: number;
  b: number[];
}
interface IB {
  b: number[];
  c: string[];
}

type TA = IA | IB; // 包含ab或者bc即可
type TB = IA & IB; // 必须包含abc
```

### 3. 泛型

在定义一个函数、type、interface、class 时，在名称后面加上<>表示即接受类型参数

```tsx
// 定义
class React.Component<P = {}, S = {}, SS = any> {}
// 调用
class Login extends React.Component<ILoginProps, ILoginState> {}
```

### 4. 复杂的条件类型

条件也支持运算 与三目运算类似

```typescript
type IsEqualType<A, B> = A extends B ? (B extends A ? true : false) : false;
type NumberEqualsToString = IsEqualType<number, string>; // false
type NumberEqualsToNumber = IsEqualType<number, number>; // true
```

### 5. 其他环境的变量

在实际应用开发时有一种场景，当前作用域下可以访问某个变量，但这个变量并不由开发者控制。例如通过 Script 标签直接引入的第三方库 CDN、一些宿主环境的 api 等。这个时候可以利用 ts 的环境声明功能，来告诉 TS 当前作用域可以访问这些变量，以获得类型提醒。

1. declare

```typescript
declare const IS_MOBILE = true; // 编译后此行消失
const wording = IS_MOBILE ? '移动端' : 'PC端';
```

2. /// 指令

```typescript
/// <reference path="../typings/monaco.d.ts" />
const range = new monaco.Range(2, 3, 6, 7);
```

### 6. 类型转换

以下是常见的类型转换方法

```typescript
// set
type TSize = 'small' | 'default' | 'big' | 'large';
// map
interface IA {
  a: number;
  b: string;
}
// map => set
type IAKeys = keyof IA; // "a" | "b"
type IAValues = IA[keyof IA]; // string | number
// set => map
type TMap = {
  [k in TSize]: number;
};
// 根据索引取值
type SubA = IA['a']; // string
```

### 7. 工具类型

这些类型一般当作工具函数来生成一些派生类型

```typescript
type Size = 'small' | 'default' | 'big';
type Result = 1 | 2 | 3 | 'error' | 'success';
type Record<K extends keyof any, T> = { [P in K]: T }; // set => map
type Extract<T, U> = T extends U ? T : never; // 保留set的一部分
type Exclude<T, U> = T extends U ? never : T; // 删除set的一部分
type Pick<T, K extends keyof T> = { [P in K]: T[P] }; // 保留map的某些属性
type Omit<T, K> = Pick<T, Exclude<keyof T, K>>; // 删除map的某些属性
type SizeMap = Record<Size, number>;
type StringResult = Extract<Result, string>; // 'error' | 'success
type NumericResult = Exclude<Result, string>; // 1 | 2 | 3
type BiggerSizeMap = Pick<SizeMap, 'default' | 'big'>; // 保留了 default和 big
type DefaultSizeMap = Omit<BiggerSizeMap, 'big'>; // 删除big
```

> 注意：尽量不要滥用这些工具类型，使用过多的工具类型会使可读性变差，而且容易发生一些错误，契约高于实现，大部分情况还是使用直观的 interface、type。以上这些工具类型都已经包含在了 TS 标准库中，在应用中直接输入名字进行使用即可(老版本 typescript 需要手动写)。

### 8. never infer typeof 三个比较重要的参数

先说三个概念：幺元、零元、逆元 用数字表示可以近似表示为 1 0 -1

never 是 | 运算的幺元，即 x | never = x

```typescript
type NumericResult = Exclude<Result, string>; // 1 | 2 | 3
// 过程是 1 | 2 | 3 | 'error' | 'success' => 1 | 2 | 3 | never | never => 1 | 2 | 3
```

infer 的作用是让 TypeScript 自己推断，并将推断的结果存储到一个临时名字中，并且只能用于 extends 语句中。它与泛型的区别在于，泛型是声明一个“参数”，而 infer 是声明一个“中间变量”。官方示例。

```typescript
type Unpacked<T> = T extends (infer U)[]
  ? U
  : T extends (...args: any[]) => infer U
  ? U
  : T extends Promise<infer U>
  ? U
  : T;

type T0 = Unpacked<string>; // string
type T1 = Unpacked<string[]>; // string
type T2 = Unpacked<() => string>; // string
type T3 = Unpacked<Promise<string>>; // string
type T4 = Unpacked<Promise<string>[]>; // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>>; // string
```

typeof 用于获取一个“常量”的类型，这里的“常量”是指任何可以在编译期确定的东西，例如 const、function、class 等。它是从 实际运行代码 通向 类型系统 的单行道。理论上，任何运行时的符号名想要为类型系统所用，都要加上 typeof。但是 class 比较特殊不需要加，因为 ts 的 class 出现得比 js 早，现有的为兼容性解决方案。

```typescript
const config = { width: 2, height: 2 };
function getLength(str: string) {
  return str.length;
}

type TConfig = typeof config; // { width: number, height: number }
type TGetLength = typeof getLength; // (str: string) => number
```

### 9. 其他杂项

类型转换的语法为 <类型名> xxx 或 xxx as 类型名。推荐始终用 as 语法，因为第一种语法无法在 tsx 文件使用，而且容易和泛型混淆。一般只有这几种场景需要使用类型转换：自动推断不准；TS 报错，想不出更好的类型编写方法，手动抄近路；临时“放飞自我”。
在使用类型转换时，应该遵守几个原则：

1. 若要放松限制，只可放松到能运行的最严格类型上
2. 如果不知道一个变量的精确类型，只标注到大概类型（例如 any[]）也比 any 好
3. 任何一段“放飞自我”（完全没有类型覆盖）区代码不应超过 2 行，应在出现第一个可以确定类型的变量时就补上标注

尽量不使用 enum，它引入了 JavaScript 没有的数据结构（编译成一个双向 map），入侵了运行时，与 TypeScript 宗旨不符。用 string literal union（'small' | 'big' | 'large'）可以做到相同的事，且在 debug 时可读性更好。如果很在意条件比较的性能，应该用二进制 flag 加位运算。

### 参考：

- [TypeScript 文档 —— 高级类型](https://www.tslang.cn/docs/handbook/advanced-types.html)
- [腾讯 AlloyTeam](https://juejin.im/user/5d0254a35188254c9f4feaf1)
