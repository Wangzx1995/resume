# React 面试题（含隐藏答案 · 详细版）

共 **24 题**，涵盖基础、Hooks、组件通信、渲染机制、Router、状态管理与进阶特性。点击「查看答案」即可展开，每题都包含**原理 / 对比 / 代码示例 / 坑点 / 面试加分点**。

---

## 零、React 基础入门

### 1. JSX 是什么？它和 JavaScript 有什么关系？

<details>
<summary>查看答案</summary>

**JSX 的本质**

JSX 是 JavaScript 的语法扩展（XML-like syntax extension），允许在 JS 中书写类似 HTML 的结构。它会被 Babel 等编译器转换为 `React.createElement(type, props, ...children)` 调用。

```jsx
// JSX
const element = <h1 className="title">Hello</h1>;

// 编译后
const element = React.createElement("h1", { className: "title" }, "Hello");
```

**JSX 规则**

- 必须有一个根元素（或用 `<></>` Fragment）
- 标签必须正确闭合
- 自定义组件必须大写开头
- `{}` 内可以嵌入任意 JavaScript 表达式
- `class` 要写成 `className`，`for` 要写成 `htmlFor`

**为什么需要 JSX？**

- 声明式描述 UI
- 与 JavaScript 无缝集成
- 编译时检查，减少运行时错误

**面试加分点**：能说出 JSX 只是语法糖，最终都会编译成 JS 对象；理解 JSX 表达式 `{}` 只能放表达式不能放语句。

</details>

---

### 2. props 和 state 有什么区别？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 维度             | props             | state                |
| ---------------- | ----------------- | -------------------- |
| 来源             | 父组件传入        | 组件内部管理         |
| 可修改性         | 只读，不可修改    | 可通过 setState 修改 |
| 作用             | 组件间通信        | 组件内部数据         |
| 变化是否触发渲染 | 是                | 是                   |
| 使用场景         | 接收外部数据/回调 | 维护组件自身状态     |

**props 示例：**

```jsx
function Welcome({ name, age }) {
  return <h1>Hello, {name}</h1>;
}

<Welcome name="Tom" age={20} />;
```

**state 示例：**

```js
const [count, setCount] = useState(0);
```

**典型坑点：**

```js
// ❌ 直接修改 props
function Welcome(props) {
  props.name = "Jerry"; // 错误！props 不可变
}

// ✅ 如果需要修改，应该提升到父组件的 state
```

**面试加分点**：能解释 props 是父组件的 state 或常量向下传递；强调 React 数据流是单向的，子组件通过回调通知父组件修改 state。

</details>

---

### 3. 什么是受控组件和非受控组件？

<details>
<summary>查看答案</summary>

**受控组件：**

表单元素的值由 React 的 state 控制，每个输入变化都通过 onChange 更新 state。

```jsx
function ControlledInput() {
  const [value, setValue] = useState("");
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}
```

**非受控组件：**

表单元素的值由 DOM 自身管理，通过 ref 获取值。

```jsx
function UncontrolledInput() {
  const inputRef = useRef(null);
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  return (
    <>
      <input ref={inputRef} defaultValue="hello" />
      <button onClick={handleSubmit}>提交</button>
    </>
  );
}
```

**核心区别表：**

| 维度       | 受控组件    | 非受控组件                       |
| ---------- | ----------- | -------------------------------- |
| 数据来源   | React state | DOM                              |
| 实时获取值 | 容易        | 需要 ref                         |
| 表单验证   | 实时验证    | 提交时验证                       |
| 适用场景   | 大多数表单  | 简单表单、文件上传、第三方库集成 |
| 代码量     | 较多        | 较少                             |

**选择建议：**

- 大多数情况优先使用受控组件
- 文件输入 `<input type="file">` 必须是非受控的
- 需要与第三方非 React 库集成时考虑非受控

**面试加分点**：能说出受控组件让 React 拥有数据的单一真相源；能解释为什么文件输入不能用受控组件（出于安全考虑，浏览器不允许 JS 设置 file input 的值）。

</details>

---

### 4. React 的事件机制是什么？什么是合成事件？

<details>
<summary>查看答案</summary>

**合成事件（SyntheticEvent）：**

React 没有直接使用浏览器原生事件，而是封装了一套跨浏览器兼容的事件系统。所有事件处理器接收的都是 React 的合成事件对象。

```jsx
function Button() {
  const handleClick = (e) => {
    e.preventDefault(); // 合成事件方法
    console.log(e.type); // "click"
  };
  return <button onClick={handleClick}>Click</button>;
}
```

**事件委托：**

React 17 之前，事件委托到 document；React 17 及以后，事件委托到 root 容器。

```
React 16: document.addEventListener("click", ...)
React 17+: rootContainer.addEventListener("click", ...)
```

**合成事件 vs 原生事件：**

| 特性     | 合成事件                                      | 原生事件         |
| -------- | --------------------------------------------- | ---------------- |
| 兼容性   | 跨浏览器统一                                  | 各浏览器有差异   |
| 性能     | 事件委托，减少监听器                          | 每个节点单独绑定 |
| 访问方式 | onClick/onChange 等                           | addEventListener |
| 事件池   | React 17 之前有事件池，异步访问需 e.persist() | 无               |

**典型坑点：**

```js
// ❌ React 17 之前，异步访问合成事件对象可能拿到 null
handleClick = (e) => {
  setTimeout(() => {
    console.log(e.target); // React 17 之前可能为 null
  }, 0);
};

// ✅ 使用 e.persist() 或升级到 React 17+
```

**面试加分点**：能解释事件委托的好处（减少内存占用、动态子元素也能响应）；能说出 React 17 事件委托位置的变更及其对事件冒泡的影响。

</details>

---

### 5. React 中条件渲染和列表渲染有哪些方式？

<details>
<summary>查看答案</summary>

**条件渲染方式：**

**1）if / else**

```jsx
function Greeting({ isLogin }) {
  if (isLogin) return <UserInfo />;
  return <LoginButton />;
}
```

**2）三元运算符**

```jsx
return <div>{isLogin ? <UserInfo /> : <LoginButton />}</div>;
```

**3）逻辑与运算符 &&**

```jsx
return <div>{hasMessage && <Message />}</div>;
```

**4）三元 + 括号分组**

```jsx
return <div>{isLogin ? <UserInfo /> : <LoginButton />}</div>;
```

**列表渲染：**

```jsx
function List({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

**条件渲染坑点：**

```jsx
// ❌ 用 && 时，如果 count 为 0 会渲染出 0
<div>{count && <span>有消息</span>}</div>

// ✅ 转成布尔值或明确判断
<div>{count > 0 && <span>有消息</span>}</div>
```

**列表渲染坑点：**

```jsx
// ❌ 用 index 做 key（列表会变化时）
{
  list.map((item, index) => <li key={index}>{item.name}</li>);
}

// ✅ 用稳定唯一 ID
{
  list.map((item) => <li key={item.id}>{item.name}</li>);
}
```

**面试加分点**：能解释条件渲染本质是返回不同的 JSX；理解 `&&` 短路求值在 React 中可能渲染出 0、"" 等 falsy 值；强调列表必须加 key。

</details>

---

## 一、基础与核心概念

### 6. React 18 相比之前版本有哪些主要变化？

<details>
<summary>查看答案</summary>

**1）并发渲染（Concurrent Rendering）**

- 引入可中断的渲染机制，React 可以根据优先级调度更新
- 核心 API：`startTransition`、`useDeferredValue`
- 允许高优先级更新（如输入）打断低优先级更新（如列表过滤）

**2）自动批处理（Automatic Batching）**

- React 18 之前只有事件处理函数中的 setState 会批处理
- React 18 后，Promise、setTimeout、原生事件中的 setState 也会自动合并为一次渲染

```js
// React 18 前：setTimeout 中两次 setState 触发两次渲染
// React 18 后：合并为一次渲染
setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
}, 1000);
```

**3）Suspense 增强**

- 服务端渲染支持 `<Suspense>`，实现选择性注水（Selective Hydration）
- 配合 `React.lazy` 做代码分割更完善

**4）新的 Hooks**

| Hook                   | 作用                        |
| ---------------------- | --------------------------- |
| `useId`                | 生成稳定唯一 ID（SSR 友好） |
| `useTransition`        | 标记非紧急更新              |
| `useDeferredValue`     | 延迟某些值的更新            |
| `useSyncExternalStore` | 安全订阅外部状态            |
| `useInsertionEffect`   | 在 DOM 插入前执行样式注入   |

**5）Strict Mode 行为变化**

- 开发环境下组件会**故意双重挂载**，帮助检测副作用
- 需要确保 `useEffect` 清理函数正确

**6）createRoot API**

```js
// React 17
ReactDOM.render(<App />, document.getElementById("root"));

// React 18
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);
```

**面试加分点**：能区分「并发渲染」和「并发模式」；能解释自动批处理减少了不必要的渲染次数；提到 `useId` 解决了 SSR 中 ID 不稳定的痛点。

</details>

---

### 7. 类组件和函数组件有什么区别？为什么现在推荐函数组件？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 维度        | 类组件                    | 函数组件                  |
| ----------- | ------------------------- | ------------------------- |
| 状态管理    | `this.state` / `setState` | `useState` / `useReducer` |
| 生命周期    | 完整生命周期钩子          | `useEffect` 等 Hooks      |
| `this` 指向 | 需要绑定或箭头函数        | 无 this，闭包捕获         |
| 复用逻辑    | HOC / render props        | Hooks / 自定义 Hook       |
| 代码量      | 相对冗余                  | 更简洁                    |
| 性能优化    | `shouldComponentUpdate`   | `React.memo` / `useMemo`  |
| 新特性支持  | 不支持新 Hooks            | 全面支持 React 18 特性    |

**为什么推荐函数组件？**

1. **Hooks 让状态逻辑复用更简单**，避免 HOC 嵌套地狱
2. **没有 this 指向问题**，代码更直观
3. **更符合函数式编程思想**，输入 props 输出 UI
4. **React 团队主要精力在 Hooks 方向**，新特性优先支持函数组件
5. **组件更小、更易测试**

**类组件仍有用武之地：**

- 老项目维护
- 需要 `componentDidCatch` 的错误边界（函数组件需借助第三方库或 React 19 新特性）
- 某些第三方库仍要求继承类组件

**典型坑点：**

```js
// 类组件中事件处理 this 问题
class Demo extends React.Component {
  handleClick() {
    console.log(this); // undefined，需要 bind 或箭头函数
  }
  render() {
    return <button onClick={this.handleClick}>click</button>;
  }
}
```

**面试加分点**：能说出函数组件在 React 16.8 之前是「无状态组件」，Hooks 的出现让它拥有完整能力；能对比 Hooks 复用和 mixin/HOC 复用的优劣。

</details>

---

### 8. `useState` 和 `useReducer` 有什么区别？分别适合什么场景？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 维度       | `useState`       | `useReducer`                |
| ---------- | ---------------- | --------------------------- |
| 适用状态   | 简单、独立的状态 | 复杂、相互关联的状态        |
| 更新逻辑   | 在组件内直接写   | 集中到 reducer 函数         |
| 可读性     | 少量状态更清晰   | 复杂逻辑更可维护            |
| 可预测性   | 一般             | 强（类似 Redux 单向数据流） |
| 初始化函数 | 支持惰性初始化   | 支持惰性初始化              |

**useState 示例：**

```js
const [count, setCount] = useState(0);
const [name, setName] = useState("");
```

**useReducer 示例：**

```js
const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + state.step };
    case "setStep":
      return { ...state, step: action.payload };
    default:
      return state;
  }
}

const [state, dispatch] = useReducer(reducer, initialState);
```

**选择建议：**

- 状态简单、互不影响 → `useState`
- 多个状态经常一起变化 → `useReducer`
- 状态转换逻辑复杂 → `useReducer`
- 需要类似 Redux 的开发体验 → `useReducer`

**典型坑点：**

```js
// ❌ useState 直接修改对象
setUser((prev) => {
  prev.name = "tom"; // 直接修改，React 可能检测不到
  return prev;
});

// ✅ 返回新对象
setUser((prev) => ({ ...prev, name: "tom" }));
```

**面试加分点**：能说出 `useState` 内部其实就是简化版的 `useReducer`；在复杂表单、多状态联动场景下 `useReducer` 更有优势。

</details>

---

### 9. `useEffect`、`useLayoutEffect` 和 `useInsertionEffect` 的区别？

<details>
<summary>查看答案</summary>

**执行时机对比：**

| Hook                 | 执行时机                               | 阻塞渲染？ |
| -------------------- | -------------------------------------- | ---------- |
| `useEffect`          | 渲染提交到屏幕之后（异步）             | 否         |
| `useLayoutEffect`    | 渲染提交到屏幕之前，DOM 变更后同步执行 | 是         |
| `useInsertionEffect` | DOM 变更前同步执行                     | 是         |

**1）useEffect —— 最常用**

```js
useEffect(() => {
  const handler = () => console.log("resize");
  window.addEventListener("resize", handler);
  return () => window.removeEventListener("resize", handler);
}, []);
```

**2）useLayoutEffect —— 需要同步测量 DOM**

```js
useLayoutEffect(() => {
  const rect = ref.current.getBoundingClientRect();
  setHeight(rect.height); // 在浏览器绘制前同步设置，避免闪烁
}, []);
```

**3）useInsertionEffect —— CSS-in-JS 样式注入**

```js
useInsertionEffect(() => {
  const style = document.createElement("style");
  style.textContent = generateCSS(rule);
  document.head.appendChild(style);
  return () => style.remove();
}, [rule]);
```

**选择建议：**

| 场景                     | 推荐                   |
| ------------------------ | ---------------------- |
| 数据获取、订阅、事件绑定 | `useEffect`            |
| 防止布局抖动、测量 DOM   | `useLayoutEffect`      |
| 注入 CSS-in-JS 样式      | `useInsertionEffect`   |
| SSR                      | 避免 `useLayoutEffect` |

**坑点：**

- `useLayoutEffect` 在 SSR 中会触发警告，因为服务端没有 DOM 绘制
- 大多数场景不需要 `useLayoutEffect`，优先用 `useEffect`
- `useInsertionEffect` 无法访问 ref（DOM 还没插入）

**面试加分点**：能解释三者的执行顺序：`useInsertionEffect` → `useLayoutEffect` → `useEffect`；知道 `useLayoutEffect` 和 `componentDidMount` 时机类似。

</details>

---

### 10. React 的生命周期如何在函数组件中对应？

<details>
<summary>查看答案</summary>

**类组件 vs 函数组件生命周期映射：**

| 类组件                   | 函数组件                                          |
| ------------------------ | ------------------------------------------------- |
| constructor              | `useState` 初始化                                 |
| render                   | 函数体本身                                        |
| componentDidMount        | `useEffect(() => {}, [])`                         |
| componentDidUpdate       | `useEffect(() => {}, [deps])`                     |
| componentWillUnmount     | `useEffect` 返回的清理函数                        |
| shouldComponentUpdate    | `React.memo` + `useMemo` / `useCallback`          |
| getDerivedStateFromProps | 用 `useEffect` 监听 props 变化更新 state          |
| componentDidCatch        | 函数组件需借助 `react-error-boundary` 或 React 19 |

**挂载流程：**

```js
useEffect(() => {
  console.log("mounted");
  return () => console.log("unmounted");
}, []);
```

**更新流程：**

```js
useEffect(() => {
  console.log("updated when count changes");
}, [count]);
```

**清理函数：**

```js
useEffect(() => {
  const id = setInterval(() => {
    setCount((c) => c + 1);
  }, 1000);
  return () => clearInterval(id); // 卸载或依赖变化前执行
}, []);
```

**注意点：**

1. `useEffect` 默认在**浏览器绘制后**执行，不阻塞渲染
2. 依赖数组必须诚实填写，否则容易出现闭包陷阱
3. 空依赖数组 `[]` 表示只在挂载和卸载时执行

**面试加分点**：能解释函数组件每次渲染都是独立闭包，所以 Effect 能捕获当前渲染的 props 和 state；能说明为什么 Hooks 不能放在 if/for 中（必须保证调用顺序稳定）。

</details>

---

## 二、Hooks 基础

### 11. 什么是 Hooks？它解决了什么问题？

<details>
<summary>查看答案</summary>

**Hooks 是什么？**

Hooks 是 React 16.8 引入的新特性，它让你在**函数组件中使用 state 和其他 React 特性**，而不需要写类组件。

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>点击了 {count} 次</button>;
}
```

**它解决了什么问题？**

1. **类组件的痛点**
   - `this` 指向难理解
   - 生命周期函数逻辑分散
   - 状态逻辑复用困难（HOC 嵌套地狱）

2. **函数组件无法使用状态**
   - React 16.8 之前，函数组件只能接收 props、返回 JSX
   - Hooks 让函数组件拥有完整能力

3. **逻辑复用更简单**
   - 通过自定义 Hook 复用状态逻辑
   - 避免 HOC / render props 的嵌套

**常用 Hooks 概览：**

| Hook          | 作用                  |
| ------------- | --------------------- |
| `useState`    | 定义状态              |
| `useEffect`   | 处理副作用            |
| `useContext`  | 读取 Context          |
| `useRef`      | 获取 DOM / 保存可变值 |
| `useMemo`     | 缓存计算结果          |
| `useCallback` | 缓存函数引用          |
| `useReducer`  | 复杂状态管理          |

**面试加分点**：能说出 Hooks 让函数组件从「无状态」变成「有状态」；能对比类组件和函数组件在代码简洁性上的差异。

</details>

---

### 12. React 常用 Hooks 有哪些？各自的作用是什么？

<details>
<summary>查看答案</summary>

**1）useState —— 让函数组件有状态**

```js
const [count, setCount] = useState(0);
```

用于定义组件内部状态，状态变化会触发重新渲染。

**2）useEffect —— 处理副作用**

```js
useEffect(() => {
  console.log("组件挂载或 count 变化");
  return () => console.log("清理函数");
}, [count]);
```

用于数据请求、订阅、DOM 操作等副作用。返回的清理函数在组件卸载或依赖变化前执行。

**3）useContext —— 跨组件共享数据**

```js
const theme = useContext(ThemeContext);
```

用于读取 React Context，避免层层传递 props。

**4）useRef —— 获取 DOM / 保存不变值**

```js
const inputRef = useRef(null);

useEffect(() => {
  inputRef.current.focus();
}, []);
```

返回一个可变对象，修改 `.current` 不会触发重新渲染。

**5）useMemo —— 缓存计算结果**

```js
const expensiveValue = useMemo(() => {
  return computeHeavyTask(data);
}, [data]);
```

用于缓存复杂计算，避免每次渲染重新计算。

**6）useCallback —— 缓存函数**

```js
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
```

用于缓存函数引用，通常配合 `React.memo` 优化子组件渲染。

**7）useReducer —— 复杂状态管理**

```js
const [state, dispatch] = useReducer(reducer, initialState);
```

适合多个状态相互关联、逻辑复杂的场景，类似 Redux。

**常用 Hooks 对比表：**

| Hook        | 用途         | 是否触发渲染         |
| ----------- | ------------ | -------------------- |
| useState    | 管理状态     | 是                   |
| useEffect   | 副作用       | 否（依赖变化时执行） |
| useContext  | 读取上下文   | 否                   |
| useRef      | DOM / 可变值 | 否                   |
| useMemo     | 缓存值       | 否                   |
| useCallback | 缓存函数     | 否                   |
| useReducer  | 复杂状态     | 是                   |

**面试加分点**：能按使用频率分类记忆；能说出 `useState` 最常用，`useEffect` 次之，`useMemo` / `useCallback` 用于性能优化。

</details>

---

### 13. 使用 Hooks 有哪些基本规则？

<details>
<summary>查看答案</summary>

**规则 1：只在最顶层调用 Hook**

不要在 `if`、`for`、嵌套函数中调用 Hook，确保每次渲染时 Hook 的调用顺序一致。

```jsx
// ❌ 错误
function Demo({ flag }) {
  if (flag) {
    const [state, setState] = useState(0);
  }
}

// ✅ 正确
function Demo({ flag }) {
  const [state, setState] = useState(flag ? 0 : 1);
}
```

**规则 2：只在 React 函数中调用 Hook**

- 在函数组件中调用
- 在自定义 Hook 中调用

不要在普通 JavaScript 函数中调用。

```jsx
// ✅ 正确
function useMyHook() {
  const [value, setValue] = useState(0);
  return value;
}

// ❌ 错误
function normalFunction() {
  const [value] = useState(0); // 普通函数不能调用 Hook
}
```

**规则 3：自定义 Hook 必须以 use 开头**

```jsx
// ✅ 正确
function useWindowWidth() { ... }

// ❌ 错误，ESLint 不会按 Hook 规则检查
function getWindowWidth() { ... }
```

**为什么需要这些规则？**

React 通过 Hook 的调用顺序来对应每个 Hook 的状态。如果顺序变化，React 无法正确匹配状态。

**面试加分点**：能解释 Hooks 依赖调用顺序；知道 ESLint 的 `react-hooks/rules-of-hooks` 和 `react-hooks/exhaustive-deps` 规则分别检查调用位置和依赖完整性。

</details>

---

## 三、组件通信与状态提升

### 14. React 中有哪些组件通信方式？

<details>
<summary>查看答案</summary>

**1）Props 父子通信**

```js
function Parent() {
  const [count, setCount] = useState(0);
  return <Child count={count} onIncrement={() => setCount((c) => c + 1)} />;
}
```

**2）状态提升（Lifting State Up）**

多个兄弟组件共享状态时，把状态放到最近的公共父组件。

**3）Context 跨层级通信**

```js
const ThemeContext = createContext("light");

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  const theme = useContext(ThemeContext);
}
```

**4）Refs 转发（useImperativeHandle）**

```js
const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
  }));
  return <input ref={inputRef} />;
});
```

**5）全局状态管理**

- Redux
- Zustand
- Jotai / Recoil
- MobX

**6）事件总线（不推荐）**

```js
// 用 mitt 等库
const emitter = mitt();
emitter.on("foo", (data) => {});
emitter.emit("foo", data);
```

**选择建议：**

| 场景               | 方式             |
| ------------------ | ---------------- |
| 父子               | props            |
| 兄弟/少量共享      | 状态提升         |
| 跨多层且变化不频繁 | Context          |
| 跨页面/高频变化    | 状态管理库       |
| 调用子组件方法     | ref + forwardRef |

**Context 注意事项：**

- Context 值变化会导致所有消费组件重新渲染
- 不要把所有状态放一个 Context
- 可用 `useMemo` 包装 value，拆分 Provider

**面试加分点**：能解释 Context 不是状态管理方案，只是依赖注入；能区分「状态提升」和「全局状态管理」的适用边界。

</details>

---

### 15. Context 有什么性能问题？如何优化？

<details>
<summary>查看答案</summary>

**性能问题：**

Context Provider 的 `value` 变化时，**所有消费该 Context 的组件都会重新渲染**，即使它们只用到其中一部分数据。

```js
function App() {
  const [user, setUser] = useState({ name: "tom", age: 20 });
  const [theme, setTheme] = useState("dark");

  return (
    <AppContext.Provider value={{ user, theme }}>
      <Toolbar />
      <Sidebar />
    </AppContext.Provider>
  );
}
```

- 每次 `setTheme`，`value` 对象都是新的
- 即使 `Toolbar` 只用 `user`，也会重新渲染

**优化方案：**

**1）拆分 Context**

```js
<UserContext.Provider value={user}>
  <ThemeContext.Provider value={theme}>
    <Toolbar />
  </ThemeContext.Provider>
</UserContext.Provider>
```

**2）useMemo 稳定 value**

```js
const value = useMemo(() => ({ user, theme }), [user, theme]);
return <AppContext.Provider value={value}>...</AppContext.Provider>;
```

**3）组件内按需选择**

```js
// ❌ 消费整个 context
const { user } = useContext(AppContext);

// ✅ 只消费需要的 context
const user = useContext(UserContext);
```

**4）使用状态管理库**

对于高频变化、复杂状态，用 Zustand/Redux/Jotai 等库，它们有更细粒度的订阅机制。

**面试加分点**：能解释 Context 的重新渲染是「全量通知」机制；能给出「按主题拆分 Context」的最佳实践。

</details>

---

### 16. `React.memo`、`useMemo` 和 `PureComponent` 有什么区别？

<details>
<summary>查看答案</summary>

**对比表：**

| API             | 层级     | 比较方式              | 使用场景         |
| --------------- | -------- | --------------------- | ---------------- |
| `PureComponent` | 类组件   | 浅比较 props 和 state | 类组件性能优化   |
| `React.memo`    | 函数组件 | 浅比较 props          | 函数组件性能优化 |
| `useMemo`       | Hook     | 缓存计算结果          | 缓存昂贵计算     |
| `useCallback`   | Hook     | 缓存函数引用          | 稳定子组件 props |

**React.memo 示例：**

```js
const Child = React.memo(({ data, onClick }) => {
  return <div onClick={onClick}>{data.name}</div>;
});
```

**自定义比较函数：**

```js
const Child = React.memo(
  (props) => {
    return <div>{props.name}</div>;
  },
  (prevProps, nextProps) => {
    return prevProps.id === nextProps.id;
  },
);
```

**PureComponent 示例：**

```js
class Child extends React.PureComponent {
  render() {
    return <div>{this.props.name}</div>;
  }
}
```

**浅比较的坑：**

```js
// ❌ 直接修改数组/对象
const list = [...items];
list.push(newItem); // 虽然展开，但 push 修改了原数组
setItems(list);

// ✅ 创建新数组
setItems([...items, newItem]);
```

**面试加分点**：能解释浅比较只比较引用，不变异数据才能生效；能说明 `React.memo` 默认不比较 state 和 context。

</details>

---

## 四、渲染机制与性能

### 17. 解释 React 的 Diff 算法和 Key 的作用。

<details>
<summary>查看答案</summary>

**Diff 算法核心假设：**

1. 不同类型的元素会产生不同的树（直接销毁重建）
2. 通过 key 可以判断哪些元素是同一份（复用而非重建）

**对比流程：**

**1）同层比较**

React 不会跨层级移动节点，只在同一层级内对比。

**2）不同类型**

```jsx
// 旧
<div><Counter /></div>
// 新
<span><Counter /></span>

// div → span 类型不同，整个子树销毁重建
```

**3）相同类型比较 props**

只更新变化的属性，保留 DOM 节点。

**4）子元素列表比较**

这是 key 发挥作用的地方。

**Key 的作用：**

```jsx
// ❌ 用 index 作为 key
{
  list.map((item, index) => <li key={index}>{item.name}</li>);
}

// ✅ 用稳定唯一 ID
{
  list.map((item) => <li key={item.id}>{item.name}</li>);
}
```

- 帮助 React 识别哪些项被修改、添加、删除
- 没有 key 或用 index 时，列表顺序变化会导致不必要的 DOM 操作
- key 只在兄弟节点之间需要唯一

**不建议用 index 的场景：**

- 列表会排序、过滤、增删
- 列表项包含受控组件（输入框等）

**可以用 index 的场景：**

- 静态列表，永不变化
- 列表项都是纯展示

**面试加分点**：能解释 key 不是给开发者看的，是给 React diff 算法用的；能说出 key 相同但位置变化时 React 会移动 DOM 而不是销毁重建。

</details>

---

### 18. 什么是虚拟 DOM？React 为什么要用它？

<details>
<summary>查看答案</summary>

**虚拟 DOM 是什么？**

虚拟 DOM 是用 JavaScript 对象描述真实 DOM 结构的轻量表示。

```js
// JSX
const element = <div className="app"><h1>Hello</h1></div>;

// 编译后类似
const element = React.createElement("div", { className: "app" },
  React.createElement("h1", null, "Hello")
);

// 虚拟 DOM 对象
{
  type: "div",
  props: {
    className: "app",
    children: {
      type: "h1",
      props: { children: "Hello" }
    }
  }
}
```

**为什么要用虚拟 DOM？**

1. **跨平台**：虚拟 DOM 是平台无关的描述，可以映射到 Web、Native（React Native）、Canvas 等
2. **批量更新**：将多次 DOM 操作合并为一次，减少重排重绘
3. **Diff 算法**：只更新真正变化的部分，避免全量替换
4. **声明式编程**：开发者描述 UI 应该长什么样，React 负责高效更新

**虚拟 DOM 一定快吗？**

不一定。直接操作 DOM 在某些简单场景下可能更快。虚拟 DOM 的优势在于**可维护性和可预测性**，以及复杂场景下的整体性能。

**React 18 的优化：**

- 编译时优化不如 Vue 3 激进
- 主要通过并发渲染、自动批处理、Suspense 等运行时机制优化

**面试加分点**：能区分「虚拟 DOM 是手段不是目的」；能解释 React 的更新流程：JSX → 虚拟 DOM → Diff → 真实 DOM 更新。

</details>

---

### 19. React 18 的并发特性 `startTransition` 和 `useDeferredValue` 怎么用？

<details>
<summary>查看答案</summary>

**问题场景：**

搜索框输入时，输入响应是紧急更新，搜索结果列表渲染是非紧急更新。如果列表渲染阻塞了输入，体验会变差。

**1）startTransition**

```js
import { startTransition } from "react";

function handleChange(e) {
  const value = e.target.value;
  setInput(value); // 紧急更新

  startTransition(() => {
    setSearchQuery(value); // 非紧急更新，可被打断
  });
}
```

**2）useDeferredValue**

```js
import { useDeferredValue } from "react";

function SearchResults({ query }) {
  const deferredQuery = useDeferredValue(query);

  // 用 deferredQuery 渲染重组件
  return <HeavyList query={deferredQuery} />;
}
```

**两者区别：**

| 特性     | `startTransition`    | `useDeferredValue` |
| -------- | -------------------- | ------------------ |
| 控制对象 | state 更新函数       | 某个值             |
| 使用位置 | 事件处理中           | 组件内部           |
| 典型场景 | 搜索时区分输入和结果 | 子组件接收延迟值   |

**注意事项：**

- 只有包裹在 `startTransition` 中的更新才具备「可中断」能力
- `useDeferredValue` 返回的值在首次渲染时和原值相同
- 需要配合 `isPending` 显示 loading：

```js
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setQuery(value);
});

return (
  <>
    {isPending && <Spinner />}
    <Results query={query} />
  </>
);
```

**面试加分点**：能解释并发渲染允许 React 暂停低优先级工作，优先响应用户输入；能区分 urgent update 和 transition update。

</details>

---

## 五、React Router

### 20. React Router v6 相比 v5 有哪些重要变化？

<details>
<summary>查看答案</summary>

**主要变化：**

**1）`<Switch>` 变为 `<Routes>`**

```jsx
// v5
<Switch>
  <Route path="/" exact component={Home} />
  <Route path="/about" component={About} />
</Switch>

// v6
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
</Routes>
```

**2）`component/render` 变为 `element`**

```jsx
// v5
<Route path="/user/:id" component={User} />

// v6
<Route path="/user/:id" element={<User />} />
```

**3）`useHistory` 移除，改用 `useNavigate`**

```js
// v5
const history = useHistory();
history.push("/about");

// v6
const navigate = useNavigate();
navigate("/about");
navigate(-1); // 返回
```

**4）嵌套路由声明式配置**

```jsx
<Routes>
  <Route path="/" element={<Layout />}>
    <Route index element={<Home />} />
    <Route path="about" element={<About />} />
    <Route path="users/*" element={<Users />} />
  </Route>
</Routes>
```

**5）`useParams`、`useSearchParams`、`useLocation` 等保留**

```js
const { id } = useParams();
const [searchParams, setSearchParams] = useSearchParams();
```

**6）路由匹配规则变化**

- v6 中所有 Route 默认是精确匹配
- 通配符用 `*`，子路由需要在父路由也写 `*`

**7）`Prompt` 组件移除**

需要拦截离开确认时，需用第三方方案或 React Router 提供的 `useBlocker`。

**面试加分点**：能说出 v6 的 API 设计更偏向 JSX element，路由配置更声明式；能解释嵌套路由中 `<Outlet />` 的作用。

</details>

---

### 21. 路由懒加载如何实现？

<details>
<summary>查看答案</summary>

**使用 React.lazy + Suspense：**

```jsx
import { lazy, Suspense } from "react";
import { BrowserRouter, Routes, Route } from "react-router-dom";

const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**与 Vue Router 懒加载的对比：**

| 特性     | Vue Router       | React Router                 |
| -------- | ---------------- | ---------------------------- |
| 语法     | `() => import()` | `React.lazy(() => import())` |
| 加载态   | 路由级配置       | `<Suspense fallback>`        |
| 错误处理 | 路由钩子         | `<ErrorBoundary>`            |

**进阶用法：**

```jsx
// 预加载
const About = lazy(() => import("./pages/About"));

<button onMouseEnter={() => import("./pages/About")}>悬停预加载</button>;
```

**面试加分点**：能解释 `React.lazy` 必须配合 `Suspense` 使用；能说出动态 import 会被打包工具自动 code-splitting。

</details>

---

## 六、状态管理

### 22. Redux 和 Zustand 有什么区别？各自适合什么场景？

<details>
<summary>查看答案</summary>

**对比表：**

| 维度     | Redux Toolkit              | Zustand            |
| -------- | -------------------------- | ------------------ |
| 学习曲线 | 较陡                       | 平缓               |
| 代码量   | 较多（store/slice/action） | 较少               |
| 中间件   | 丰富（redux-thunk等）      | 较少但够用         |
| TS 支持  | 好                         | 优秀               |
| 订阅粒度 | 连接整个 state             | 可按 selector 订阅 |
| 适用规模 | 大型、复杂应用             | 中小型、快速开发   |

**Redux Toolkit 示例：**

```js
import { createSlice, configureStore } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
  },
});

const store = configureStore({ reducer: counterSlice.reducer });
```

**Zustand 示例：**

```js
import { create } from "zustand";

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// 组件中使用
const count = useStore((state) => state.count);
```

**选择建议：**

| 场景                           | 推荐         |
| ------------------------------ | ------------ |
| 大型应用、多人协作、复杂调试   | Redux        |
| 快速开发、轻量状态、Hooks 友好 | Zustand      |
| 原子化状态、细粒度订阅         | Jotai/Recoil |
| 简单全局状态                   | Context      |

**面试加分点**：能说出 Redux 的三原则（单一数据源、state 只读、使用纯函数修改）；能解释 Zustand 没有 Provider 包裹的设计优势。

</details>

---

## 七、进阶与工程化

### 23. 什么是高阶组件（HOC）和 Render Props？现在还用吗？

<details>
<summary>查看答案</summary>

**1）高阶组件（HOC）**

接收一个组件，返回一个新组件，用于复用组件逻辑。

```js
function withLogger(WrappedComponent) {
  return function WithLogger(props) {
    useEffect(() => {
      console.log("mounted");
    }, []);
    return <WrappedComponent {...props} />;
  };
}

const Enhanced = withLogger(MyComponent);
```

**2）Render Props**

通过 props 传递一个函数，由父组件决定渲染什么。

```js
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  // ... 监听鼠标移动
  return render(position);
}

<MouseTracker
  render={({ x, y }) => (
    <p>
      {x}, {y}
    </p>
  )}
/>;
```

**现在还常用吗？**

- Hooks 出现后，大部分场景被自定义 Hook 替代
- HOC 仍用于第三方库（如 React Router 的 `withRouter` 历史版本）
- Render Props 在需要动态渲染内容的场景仍有价值

**与 Hooks 的对比：**

| 方案         | 优点                 | 缺点                 |
| ------------ | -------------------- | -------------------- |
| HOC          | 可复用、可组合       | 嵌套地狱、props 冲突 |
| Render Props | 灵活                 | 嵌套复杂             |
| Hooks        | 简洁、无嵌套、易测试 | 需要遵循规则         |

**面试加分点**：能解释 HOC 和 Render Props 解决的是「逻辑复用」问题，Hooks 是更现代的方案；能说出 HOC 常见的命名冲突（displayName、props 覆盖）。

</details>

---

### 24. React 项目有哪些常见性能优化手段？

<details>
<summary>查看答案</summary>

**1）避免不必要的渲染**

- `React.memo` 缓存函数组件
- `useMemo` / `useCallback` 缓存计算和函数
- `useReducer` 或拆分 state，避免无关更新

**2）代码分割**

```jsx
const Heavy = lazy(() => import("./Heavy"));
```

**3）虚拟列表**

```jsx
import { FixedSizeList } from "react-window";

<FixedSizeList height={400} itemCount={10000} itemSize={35}>
  {Row}
</FixedSizeList>;
```

**4）避免在 render 中创建新对象/函数**

```jsx
// ❌
return <Child style={{ color: "red" }} onClick={() => {}} />;

// ✅
const style = useMemo(() => ({ color: "red" }), []);
const handleClick = useCallback(() => {}, []);
```

**5）合理使用 Context**

- 拆分 Context
- 使用 `useMemo` 稳定 value

**6）图片懒加载**

```jsx
<img loading="lazy" src="..." />
```

**7）利用 React 18 并发特性**

- `startTransition` / `useDeferredValue`
- `<Suspense>` 配合代码分割

**8）避免滥用 useEffect**

- 不是所有副作用都需要 useEffect
- 数据获取考虑用 React Query / SWR

**9）生产构建优化**

- 启用 gzip / brotli
- 使用 CDN
- 合理配置 code splitting

**10）使用性能分析工具**

- React DevTools Profiler
- Chrome Performance
- Lighthouse

**面试加分点**：能强调「先测量再优化」；能区分 React 18 的自动优化（并发、批处理）和开发者手动优化（memo、虚拟列表等）。

</details>

---

## 使用说明

- 在 Cursor / VSCode / Qoder 等 Markdown 预览中，点击 **「查看答案」** 即可展开。
- 每题答案均包含：**核心要点 → 代码示例 → 对比/原理 → 坑点 → 面试加分点**
- 若预览环境不支持 `<details>`，请使用支持 HTML 的 Markdown 阅读器查看。
