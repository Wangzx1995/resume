# React 面试对话版（一问一答 · 点击展开答案）

共 **24 题**，模拟真实面试场景。点击「查看回答」展开答案。

---

## 零、React 基础入门

**面试官：JSX 是什么？它和 JavaScript 有什么关系？**

<details>
<summary>查看回答</summary>

> JSX 是 JavaScript 的语法扩展，允许在 JS 中书写类似 HTML 的结构。它会被 Babel 编译为 `React.createElement(type, props, ...children)` 调用。
>
> **JSX 规则**：
>
> - 必须有一个根元素（或用 `<></>`Fragment）
> - 标签必须正确闭合，自定义组件大写开头
> - `{}` 内嵌入表达式（不能放语句）
> - `class` → `className`，`for` → `htmlFor`
>
> **加分点**：JSX 只是语法糖，最终编译成 JS 对象；`{}` 只能放表达式不能放 if/for 语句。

</details>

---

**面试官：props 和 state 有什么区别？**

<details>
<summary>查看回答</summary>

> | 维度         | props          | state                |
> | ------------ | -------------- | -------------------- |
> | 来源         | 父组件传入     | 组件内部管理         |
> | 可修改性     | 只读，不可修改 | 可通过 setState 修改 |
> | 作用         | 组件间通信     | 组件内部数据         |
> | 变化触发渲染 | 是             | 是                   |
>
> **核心**：props 是父组件的 state 或常量向下传递；React 数据流是单向的，子组件通过回调通知父组件修改。
>
> **加分点**：props 不可变是 React 单向数据流的基础。

</details>

---

**面试官：什么是受控组件和非受控组件？**

<details>
<summary>查看回答</summary>

> **受控组件**：表单值由 React state 控制，每个变化通过 onChange 更新 state。
>
> **非受控组件**：表单值由 DOM 自身管理，通过 ref 获取值。
>
> | 维度       | 受控        | 非受控                 |
> | ---------- | ----------- | ---------------------- |
> | 数据来源   | React state | DOM                    |
> | 实时获取值 | 容易        | 需要 ref               |
> | 表单验证   | 实时验证    | 提交时验证             |
> | 适用场景   | 大多数表单  | 文件上传、第三方库集成 |
>
> **加分点**：受控组件让 React 拥有数据单一真相源；文件输入不能受控（浏览器安全限制不允许 JS 设置 file input 的值）。

</details>

---

**面试官：React 的事件机制是什么？什么是合成事件？**

<details>
<summary>查看回答</summary>

> React 封装了一套跨浏览器兼容的合成事件系统（SyntheticEvent），所有事件处理器接收的都是合成事件对象。
>
> **事件委托**：React 17+ 事件委托到 root 容器（之前是 document），减少监听器数量。
>
> | 特性   | 合成事件        | 原生事件         |
> | ------ | --------------- | ---------------- |
> | 兼容性 | 跨浏览器统一    | 有差异           |
> | 性能   | 事件委托        | 每个节点单独绑定 |
> | 事件池 | React 17 之前有 | 无               |
>
> **加分点**：事件委托好处是减少内存占用、动态子元素也能响应；React 17 变更委托位置解决了微前端场景的事件冒泡问题。

</details>

---

**面试官：React 中条件渲染和列表渲染有哪些方式？**

<details>
<summary>查看回答</summary>

> **条件渲染**：if/else、三元运算符、`&&` 逻辑与。
>
> **列表渲染**：`items.map(item => <li key={item.id}>...</li>)`
>
> **坑点**：
>
> - `&&` 时如果左值为 0 会渲染出 "0"（需转布尔值或用 `count > 0 &&`）
> - 列表 key 不要用 index（列表变化时会导致状态混乱）
> - key 用稳定唯一 ID
>
> **加分点**：条件渲染本质是返回不同 JSX；key 是给 React diff 算法用的，相同 key 会复用 DOM 而非销毁重建。

</details>

---

## 一、基础与核心概念

**面试官：React 18 相比之前版本有哪些主要变化？**

<details>
<summary>查看回答</summary>

> 1. **并发渲染**：可中断的渲染机制，高优先级更新打断低优先级。核心 API：`startTransition`、`useDeferredValue`
> 2. **自动批处理**：Promise、setTimeout 中的多个 setState 也会合并为一次渲染（之前只有事件处理函数内会批处理）
> 3. **Suspense 增强**：服务端支持选择性注水
> 4. **新 Hooks**：`useId`、`useTransition`、`useDeferredValue`、`useSyncExternalStore`、`useInsertionEffect`
> 5. **Strict Mode 双重挂载**：开发环境故意挂载两次检测副作用
> 6. **createRoot API**：`ReactDOM.createRoot(el).render(<App/>)` 替代 `ReactDOM.render`
>
> **加分点**：自动批处理减少不必要的渲染次数；`useId` 解决 SSR 中 ID 不稳定的痛点。

</details>

---

**面试官：类组件和函数组件有什么区别？为什么推荐函数组件？**

<details>
<summary>查看回答</summary>

> | 维度     | 类组件                  | 函数组件                |
> | -------- | ----------------------- | ----------------------- |
> | 状态     | `this.state`/`setState` | `useState`/`useReducer` |
> | 生命周期 | 完整钩子                | `useEffect` 等          |
> | this     | 需绑定                  | 无 this，闭包捕获       |
> | 逻辑复用 | HOC/render props        | 自定义 Hook             |
> | 新特性   | 不支持新 Hooks          | 全面支持 React 18       |
>
> **推荐理由**：Hooks 让状态逻辑复用更简单、没有 this 问题、更符合函数式编程、代码更简洁易测试。
>
> **加分点**：函数组件在 16.8 前是「无状态组件」，Hooks 出现让它拥有完整能力。

</details>

---

**面试官：`useState` 和 `useReducer` 有什么区别？分别适合什么场景？**

<details>
<summary>查看回答</summary>

> | 维度     | useState     | useReducer                |
> | -------- | ------------ | ------------------------- |
> | 适用     | 简单独立状态 | 复杂关联状态              |
> | 更新逻辑 | 组件内直接写 | 集中到 reducer 函数       |
> | 可预测性 | 一般         | 强（类 Redux 单向数据流） |
>
> **选择**：状态简单互不影响→useState；多状态经常一起变化/转换逻辑复杂→useReducer。
>
> **坑点**：修改对象/数组必须返回新引用（不可变更新），否则 React 可能检测不到变化。
>
> **加分点**：`useState` 内部其实就是简化版的 `useReducer`。

</details>

---

**面试官：`useEffect`、`useLayoutEffect` 和 `useInsertionEffect` 的区别？**

<details>
<summary>查看回答</summary>

> | Hook                 | 执行时机                   | 阻塞渲染 |
> | -------------------- | -------------------------- | -------- |
> | `useEffect`          | 渲染到屏幕后（异步）       | 否       |
> | `useLayoutEffect`    | DOM 变更后、绘制前（同步） | 是       |
> | `useInsertionEffect` | DOM 变更前（同步）         | 是       |
>
> **执行顺序**：useInsertionEffect → useLayoutEffect → useEffect
>
> **选择**：
>
> - 数据请求、订阅、事件绑定 → `useEffect`
> - 防止布局抖动、测量 DOM → `useLayoutEffect`
> - CSS-in-JS 样式注入 → `useInsertionEffect`
>
> **加分点**：`useLayoutEffect` 和 `componentDidMount` 时机类似；SSR 中避免用 useLayoutEffect。

</details>

---

**面试官：React 的生命周期如何在函数组件中对应？**

<details>
<summary>查看回答</summary>

> | 类组件                | 函数组件                                  |
> | --------------------- | ----------------------------------------- |
> | constructor           | `useState` 初始化                         |
> | render                | 函数体本身                                |
> | componentDidMount     | `useEffect(() => {}, [])`                 |
> | componentDidUpdate    | `useEffect(() => {}, [deps])`             |
> | componentWillUnmount  | useEffect 返回的清理函数                  |
> | shouldComponentUpdate | `React.memo` + useMemo/useCallback        |
> | componentDidCatch     | 需借助 `react-error-boundary` 或 React 19 |
>
> **要点**：
>
> - `useEffect` 默认浏览器绘制后执行，不阻塞渲染
> - 依赖数组必须诚实填写，否则闭包陷阱
> - 函数组件每次渲染都是独立闭包
>
> **加分点**：Hooks 不能放在 if/for 中，必须保证调用顺序稳定（React 通过调用顺序对应状态）。

</details>

---

## 二、Hooks 基础

**面试官：什么是 Hooks？它解决了什么问题？**

<details>
<summary>查看回答</summary>

> Hooks 是 React 16.8 引入的特性，让函数组件能使用 state 和其他 React 特性。
>
> **解决的问题**：
>
> 1. 类组件 this 指向难理解、生命周期逻辑分散
> 2. 状态逻辑复用困难（HOC 嵌套地狱）
> 3. 函数组件无法使用状态（16.8 前）
>
> **常用 Hooks**：useState（状态）、useEffect（副作用）、useContext（上下文）、useRef（DOM/可变值）、useMemo（缓存值）、useCallback（缓存函数）、useReducer（复杂状态）
>
> **加分点**：Hooks 让函数组件从「无状态」变成「有状态」；自定义 Hook 是最现代的逻辑复用方案。

</details>

---

**面试官：React 常用 Hooks 有哪些？各自的作用是什么？**

<details>
<summary>查看回答</summary>

> | Hook        | 用途                    | 是否触发渲染 |
> | ----------- | ----------------------- | ------------ |
> | useState    | 管理状态                | 是           |
> | useEffect   | 副作用（请求/订阅/DOM） | 否           |
> | useContext  | 读取 Context            | 否           |
> | useRef      | DOM 引用 / 可变值       | 否           |
> | useMemo     | 缓存计算结果            | 否           |
> | useCallback | 缓存函数引用            | 否           |
> | useReducer  | 复杂状态管理            | 是           |
>
> **使用频率**：useState 最高，useEffect 次之，useMemo/useCallback 用于性能优化。
>
> **加分点**：useRef 修改 `.current` 不触发渲染，适合存定时器 ID、前一次值等需要跨渲染周期保持的可变数据。

</details>

---

**面试官：使用 Hooks 有哪些基本规则？**

<details>
<summary>查看回答</summary>

> **规则 1**：只在最顶层调用 Hook，不能放在 if/for/嵌套函数中（保证调用顺序一致）。
>
> **规则 2**：只在 React 函数组件或自定义 Hook 中调用，不能在普通函数中调用。
>
> **规则 3**：自定义 Hook 必须以 `use` 开头（ESLint 才会按 Hook 规则检查）。
>
> **为什么**：React 通过 Hook 的调用顺序来对应每个 Hook 的状态。顺序变化会导致状态错乱。
>
> **加分点**：ESLint 的 `react-hooks/rules-of-hooks` 检查调用位置，`react-hooks/exhaustive-deps` 检查依赖完整性。

</details>

---

## 三、组件通信与状态提升

**面试官：React 中有哪些组件通信方式？**

<details>
<summary>查看回答</summary>

> | 方式                          | 适用场景                      |
> | ----------------------------- | ----------------------------- |
> | Props/回调                    | 父子通信                      |
> | 状态提升                      | 兄弟共享（提到公共父组件）    |
> | Context                       | 跨多层且变化不频繁            |
> | Refs + forwardRef             | 父调子方法                    |
> | 全局状态管理（Redux/Zustand） | 跨页面/高频变化               |
> | mitt（事件总线）              | 兄弟/跨层级事件通知（不推荐） |
>
> **选择原则**：父子→props；少量共享→状态提升；跨层级→Context；跨页面/复杂→状态管理库。
>
> **加分点**：Context 不是状态管理方案，只是依赖注入；能区分「状态提升」和「全局状态管理」的适用边界。

</details>

---

**面试官：Context 有什么性能问题？如何优化？**

<details>
<summary>查看回答</summary>

> **问题**：Provider 的 value 变化时，所有消费该 Context 的组件都会重新渲染，即使只用到部分数据。
>
> **优化方案**：
>
> 1. **拆分 Context**：按主题拆分（UserContext/ThemeContext），变化只影响对应消费者
> 2. **useMemo 稳定 value**：避免每次渲染创建新对象
> 3. **组件内按需选择**：只消费需要的 Context
> 4. **使用状态管理库**：高频变化用 Zustand/Redux（更细粒度的订阅机制）
>
> **加分点**：Context 的重新渲染是「全量通知」机制；「按主题拆分 Context」是最佳实践。

</details>

---

**面试官：`React.memo`、`useMemo` 和 `PureComponent` 有什么区别？**

<details>
<summary>查看回答</summary>

> | API           | 层级     | 比较方式           | 用途             |
> | ------------- | -------- | ------------------ | ---------------- |
> | PureComponent | 类组件   | 浅比较 props+state | 类组件优化       |
> | React.memo    | 函数组件 | 浅比较 props       | 函数组件优化     |
> | useMemo       | Hook     | 缓存计算结果       | 避免重复计算     |
> | useCallback   | Hook     | 缓存函数引用       | 稳定子组件 props |
>
> **浅比较的坑**：必须不可变更新（创建新对象/数组），直接修改引用不会触发更新。
>
> **React.memo 可传自定义比较函数**：`React.memo(Comp, (prev, next) => prev.id === next.id)`
>
> **加分点**：浅比较只比引用；`React.memo` 默认不比较 state 和 context。

</details>

---

## 四、渲染机制与性能

**面试官：解释 React 的 Diff 算法和 Key 的作用。**

<details>
<summary>查看回答</summary>

> **Diff 核心假设**：
>
> 1. 不同类型元素产生不同树（直接销毁重建）
> 2. 通过 key 判断哪些元素是同一份（复用而非重建）
>
> **Key 的作用**：帮助 React 在列表中识别哪些项被修改/添加/删除。
>
> **不用 index 做 key 的原因**：列表排序/增删时，index 变化会导致 React 错误复用 DOM，引起状态混乱（如输入框内容串位）。
>
> **可以用 index 的情况**：静态列表永不变化、列表项纯展示无状态。
>
> **加分点**：key 相同但位置变化时 React 会移动 DOM 而非销毁重建。

</details>

---

**面试官：什么是虚拟 DOM？React 为什么要用它？**

<details>
<summary>查看回答</summary>

> 虚拟 DOM 是用 JS 对象描述真实 DOM 的轻量表示。
>
> **为什么用**：
>
> 1. **跨平台**：可映射到 Web/Native/Canvas
> 2. **批量更新**：多次操作合并为一次，减少重排重绘
> 3. **Diff 算法**：只更新变化部分
> 4. **声明式编程**：开发者描述 UI 应该是什么样，React 负责高效更新
>
> **虚拟 DOM 一定快吗？** 不一定。简单场景下直接操作 DOM 可能更快。优势在于可维护性、可预测性和复杂场景的整体性能。
>
> **更新流程**：JSX → 虚拟 DOM → Diff → 真实 DOM 更新。
>
> **加分点**：虚拟 DOM 是手段不是目的；React 18 主要通过并发渲染等运行时机制优化，而非编译优化。

</details>

---

**面试官：React 18 的并发特性 `startTransition` 和 `useDeferredValue` 怎么用？**

<details>
<summary>查看回答</summary>

> **场景**：搜索框输入（紧急）vs 搜索结果渲染（非紧急），后者不应阻塞输入响应。
>
> **startTransition**：在事件处理中标记非紧急更新，可被高优先级打断。
>
> **useDeferredValue**：在组件内返回一个延迟版本的值，传给重组件。
>
> | 特性     | startTransition | useDeferredValue |
> | -------- | --------------- | ---------------- |
> | 控制对象 | state 更新函数  | 某个值           |
> | 使用位置 | 事件处理中      | 组件内部         |
>
> **配合 isPending 显示 loading**：`const [isPending, startTransition] = useTransition()`
>
> **加分点**：并发渲染允许 React 暂停低优先级工作，优先响应用户输入；区分 urgent update 和 transition update。

</details>

---

## 五、React Router

**面试官：React Router v6 相比 v5 有哪些重要变化？**

<details>
<summary>查看回答</summary>

> 1. `<Switch>` → `<Routes>`
> 2. `component/render` → `element={<Comp/>}`
> 3. `useHistory` → `useNavigate`
> 4. 嵌套路由声明式配置 + `<Outlet/>`
> 5. 所有 Route 默认精确匹配
> 6. 移除 `Prompt`（需用 `useBlocker` 或第三方方案）
>
> **加分点**：v6 的 API 设计更偏向 JSX element；嵌套路由中 `<Outlet/>` 类似 Vue 的 `<router-view>`。

</details>

---

**面试官：路由懒加载如何实现？**

<details>
<summary>查看回答</summary>

> 使用 `React.lazy(() => import('./Page'))` + `<Suspense fallback={<Loading/>}>` 包裹路由。
>
> 打包工具识别动态 `import()` 自动 code-splitting，每个路由一个 chunk，首次访问时下载。
>
> **进阶**：鼠标悬停时预加载 `onMouseEnter={() => import('./Page')}`
>
> **加分点**：`React.lazy` 必须配合 `Suspense` 使用；配合 `<ErrorBoundary>` 处理加载失败。

</details>

---

## 六、状态管理

**面试官：Redux 和 Zustand 有什么区别？各自适合什么场景？**

<details>
<summary>查看回答</summary>

> | 维度     | Redux Toolkit  | Zustand            |
> | -------- | -------------- | ------------------ |
> | 学习曲线 | 较陡           | 平缓               |
> | 代码量   | 较多           | 较少               |
> | 订阅粒度 | 连接整个 state | 可按 selector 订阅 |
> | 适用规模 | 大型复杂应用   | 中小型快速开发     |
>
> **选择**：
>
> - 大型应用、多人协作、复杂调试 → Redux
> - 快速开发、轻量状态、Hooks 友好 → Zustand
> - 原子化状态、细粒度订阅 → Jotai/Recoil
> - 简单全局状态 → Context
>
> **加分点**：Redux 三原则（单一数据源、state 只读、纯函数修改）；Zustand 没有 Provider 包裹的设计更轻量。

</details>

---

## 七、进阶与工程化

**面试官：什么是高阶组件（HOC）和 Render Props？现在还用吗？**

<details>
<summary>查看回答</summary>

> **HOC**：接收一个组件返回新组件，用于复用逻辑。
>
> **Render Props**：通过 props 传递渲染函数，子组件决定传什么数据，父组件决定怎么渲染。
>
> **现在还用吗？** Hooks 出现后大部分场景被自定义 Hook 替代。HOC 仍见于某些第三方库；Render Props 在需要动态渲染内容时仍有价值。
>
> | 方案         | 优点             | 缺点                 |
> | ------------ | ---------------- | -------------------- |
> | HOC          | 可复用可组合     | 嵌套地狱、props 冲突 |
> | Render Props | 灵活             | 嵌套复杂             |
> | Hooks        | 简洁无嵌套易测试 | 需遵循规则           |
>
> **加分点**：三者解决的都是「逻辑复用」问题，Hooks 是最现代方案。

</details>

---

**面试官：React 项目有哪些常见性能优化手段？**

<details>
<summary>查看回答</summary>

> **避免不必要渲染**：React.memo、useMemo/useCallback、拆分 state
>
> **代码分割**：React.lazy + Suspense
>
> **大列表**：虚拟滚动（react-window/react-virtuoso）
>
> **避免 render 中创建新对象/函数**：用 useMemo/useCallback 缓存
>
> **合理使用 Context**：拆分 + useMemo 稳定 value
>
> **React 18 并发特性**：startTransition、useDeferredValue
>
> **数据获取**：React Query / SWR 代替手动 useEffect
>
> **生产优化**：gzip/CDN/code splitting
>
> **监控工具**：React DevTools Profiler、Lighthouse
>
> **加分点**：先测量再优化；区分 React 18 自动优化（并发/批处理）和手动优化（memo/虚拟列表）。

</details>

---

## 使用说明

- 点击 **「查看回答」** 即可展开答案
- 格式：面试官提问 → 候选人作答（含要点/对比/加分点）
- 适合面试前快速模拟练习
