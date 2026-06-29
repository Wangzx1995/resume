# React 面试题（口述版）

共 **24 题**，涵盖基础、Hooks、组件通信、渲染机制、Router、状态管理与进阶特性。所有答案已精简为口述要点，可直接用于面试口头表达。

---

## 零、React 基础入门

### 1. JSX 是什么？它和 JavaScript 有什么关系？

<details>
<summary>查看答案</summary>

**JSX 的本质：** JavaScript 的语法扩展，允许在 JS 中书写类似 HTML 的结构。它会被 Babel 编译为 `React.createElement(type, props, ...children)` 调用，最终生成 JS 对象（虚拟 DOM）。

**JSX 规则：**

- 必须有一个根元素（或用 `<></>` Fragment）
- 标签必须正确闭合，自定义组件大写开头
- `{}` 内只能放表达式，不能放语句
- `class` → `className`，`for` → `htmlFor`

**为什么需要 JSX：** 声明式描述 UI、与 JS 无缝集成、编译时检查减少运行时错误。

**面试加分点**：JSX 只是语法糖，最终编译成 JS 对象；`{}` 只能放表达式不能放语句（if/for 不行）。

</details>

---

### 2. props 和 state 有什么区别？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 维度         | props             | state                |
| ------------ | ----------------- | -------------------- |
| 来源         | 父组件传入        | 组件内部管理         |
| 可修改性     | 只读，不可修改    | 可通过 setState 修改 |
| 作用         | 组件间通信        | 组件内部数据         |
| 变化触发渲染 | 是                | 是                   |
| 使用场景     | 接收外部数据/回调 | 维护组件自身状态     |

**关键点：**

- props 不可变，如果子组件想修改数据，需要通过回调通知父组件
- React 数据流是单向的：父 → 子（通过 props），子 → 父（通过回调函数）

**面试加分点**：props 是父组件的 state 或常量向下传递；强调 React 单向数据流。

</details>

---

### 3. 什么是受控组件和非受控组件？

<details>
<summary>查看答案</summary>

**受控组件：** 表单元素的值由 React state 控制，每次输入变化通过 onChange 更新 state → value 绑定 state。

**非受控组件：** 表单元素值由 DOM 自身管理，通过 ref 获取值 → 用 defaultValue 设初始值。

**核心区别表：**

| 维度       | 受控组件    | 非受控组件         |
| ---------- | ----------- | ------------------ |
| 数据来源   | React state | DOM                |
| 实时获取值 | 容易        | 需要 ref           |
| 表单验证   | 实时验证    | 提交时验证         |
| 适用场景   | 大多数表单  | 简单表单、文件上传 |

**选择建议：** 大多数情况用受控；文件输入 `<input type="file">` 必须非受控（浏览器安全限制不允许 JS 设置文件值）。

**面试加分点**：受控组件让 React 拥有数据的单一真相源；文件输入只能非受控。

</details>

---

### 4. React 的事件机制是什么？什么是合成事件？

<details>
<summary>查看答案</summary>

**合成事件：** React 封装了一套跨浏览器兼容的事件系统，所有事件处理器接收的是 React 的 SyntheticEvent 对象，不是原生事件。

**事件委托：**

- React 17 之前：事件委托到 document
- React 17+：事件委托到 root 容器（解决多 React 实例共存问题）

**合成事件 vs 原生事件：**

| 特性   | 合成事件            | 原生事件         |
| ------ | ------------------- | ---------------- |
| 兼容性 | 跨浏览器统一        | 各浏览器有差异   |
| 性能   | 事件委托减少监听器  | 每个节点单独绑定 |
| 事件池 | React 17 前有事件池 | 无               |

**面试加分点**：事件委托的好处 — 减少内存占用、动态子元素也能响应；React 17 改为委托到 root 容器解决了微前端等多实例场景的问题。

</details>

---

### 5. React 中条件渲染和列表渲染有哪些方式？

<details>
<summary>查看答案</summary>

**条件渲染方式：**

1. if/else — 直接在函数体中返回不同 JSX
2. 三元运算符 — `{condition ? <A /> : <B />}`
3. 逻辑与 `&&` — `{hasMessage && <Message />}`

**列表渲染：** 用 `.map()` 遍历数组，每个项必须加 key

**条件渲染坑点：** `&&` 左侧是 0 时会渲染出数字 0，应该写成 `count > 0 && <span>有消息</span>`

**列表渲染坑点：**

- 不要用 index 做 key（列表会排序/增删时导致错误复用）
- 用稳定唯一 ID 做 key
- 可以用 index 的场景：静态列表、纯展示、永不变化

**面试加分点**：条件渲染本质是返回不同 JSX；`&&` 可能渲染出 falsy 值；列表必须加 key 且要稳定唯一。

</details>

---

## 一、基础与核心概念

### 6. React 18 相比之前版本有哪些主要变化？

<details>
<summary>查看答案</summary>

**六大变化：**

1. **并发渲染** — 可中断的渲染机制，高优先级更新（输入）可打断低优先级更新（列表渲染）。核心 API：`startTransition`、`useDeferredValue`

2. **自动批处理** — React 18 前只有事件处理中 setState 会合并；18 后 Promise、setTimeout 中的 setState 也自动合并为一次渲染

3. **Suspense 增强** — SSR 支持 Suspense，实现选择性注水

4. **新 Hooks** — `useId`（SSR 稳定ID）、`useTransition`（标记非紧急更新）、`useDeferredValue`（延迟值更新）、`useSyncExternalStore`（安全订阅外部状态）

5. **Strict Mode 变化** — 开发环境故意双重挂载，检测副作用是否正确清理

6. **createRoot API** — 新的挂载方式替代 `ReactDOM.render`

**面试加分点**：区分「并发渲染」和「并发模式」；自动批处理减少不必要的渲染次数；`useId` 解决了 SSR 中 ID 不稳定的痛点。

</details>

---

### 7. 类组件和函数组件有什么区别？为什么现在推荐函数组件？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 维度       | 类组件                | 函数组件              |
| ---------- | --------------------- | --------------------- |
| 状态管理   | this.state / setState | useState / useReducer |
| 生命周期   | 完整生命周期钩子      | useEffect 等 Hooks    |
| this 指向  | 需要绑定或箭头函数    | 无 this，闭包捕获     |
| 复用逻辑   | HOC / render props    | 自定义 Hook           |
| 性能优化   | shouldComponentUpdate | React.memo / useMemo  |
| 新特性支持 | 不支持新 Hooks        | 全面支持 React 18     |

**推荐函数组件的原因：**

1. Hooks 让状态逻辑复用更简单，避免 HOC 嵌套地狱
2. 没有 this 指向问题，代码更直观
3. 更符合函数式编程思想
4. React 团队主力方向，新特性优先支持
5. 组件更小、更易测试

**类组件仍有用武之地：** 错误边界（componentDidCatch）、老项目维护

**面试加分点**：函数组件在 16.8 前是无状态组件，Hooks 让它拥有完整能力；能对比 Hooks 和 HOC 在逻辑复用上的优劣。

</details>

---

### 8. `useState` 和 `useReducer` 有什么区别？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 维度     | useState     | useReducer              |
| -------- | ------------ | ----------------------- |
| 适用状态 | 简单、独立   | 复杂、相互关联          |
| 更新逻辑 | 组件内直接写 | 集中到 reducer 函数     |
| 可读性   | 少量状态清晰 | 复杂逻辑更可维护        |
| 可预测性 | 一般         | 强（类似 Redux 单向流） |

**选择建议：**

- 状态简单、互不影响 → useState
- 多个状态经常一起变化 → useReducer
- 状态转换逻辑复杂（如表单多步骤）→ useReducer

**坑点：** 更新对象/数组状态时必须返回新引用，不能直接修改原对象（React 用引用比较检测变化）。

**面试加分点**：useState 内部其实是简化版的 useReducer；复杂表单、多状态联动场景 useReducer 更有优势。

</details>

---

### 9. `useEffect`、`useLayoutEffect` 和 `useInsertionEffect` 的区别？

<details>
<summary>查看答案</summary>

**执行时机对比：**

| Hook               | 执行时机                         | 阻塞渲染？ | 典型场景                 |
| ------------------ | -------------------------------- | ---------- | ------------------------ |
| useEffect          | 渲染提交到屏幕后（异步）         | 否         | 数据请求、订阅、事件绑定 |
| useLayoutEffect    | DOM 变更后、浏览器绘制前（同步） | 是         | 测量 DOM、防止布局抖动   |
| useInsertionEffect | DOM 变更前（同步）               | 是         | CSS-in-JS 样式注入       |

**执行顺序：** useInsertionEffect → useLayoutEffect → useEffect

**选择建议：**

- 绝大多数场景用 useEffect
- 需要同步读取 DOM 布局信息防止闪烁 → useLayoutEffect
- CSS-in-JS 库内部使用 → useInsertionEffect
- SSR 中避免 useLayoutEffect（服务端没有 DOM 绘制）

**面试加分点**：useLayoutEffect 时机类似 componentDidMount；useInsertionEffect 无法访问 ref（DOM 还没插入）。

</details>

---

### 10. React 的生命周期如何在函数组件中对应？

<details>
<summary>查看答案</summary>

**映射关系：**

| 类组件                | 函数组件对应                  |
| --------------------- | ----------------------------- |
| constructor           | useState 初始化               |
| render                | 函数体本身                    |
| componentDidMount     | `useEffect(() => {}, [])`     |
| componentDidUpdate    | `useEffect(() => {}, [deps])` |
| componentWillUnmount  | useEffect 返回的清理函数      |
| shouldComponentUpdate | React.memo + useMemo          |
| componentDidCatch     | 需借助 react-error-boundary   |

**关键理解：**

- 函数组件每次渲染都是独立闭包，Effect 捕获当前渲染的 props/state
- 依赖数组必须诚实填写，否则出现闭包陷阱
- 空依赖 `[]` = 只在挂载/卸载时执行
- Hooks 不能放在 if/for 中 — 必须保证调用顺序稳定

**面试加分点**：函数组件每次渲染是独立闭包；Hooks 依赖调用顺序来对应状态。

</details>

---

## 二、Hooks 基础

### 11. 什么是 Hooks？它解决了什么问题？

<details>
<summary>查看答案</summary>

**定义：** React 16.8 引入的特性，让函数组件能使用 state 和其他 React 特性。

**解决三大问题：**

1. **类组件痛点** — this 指向难理解、生命周期逻辑分散、状态复用困难
2. **函数组件无法使用状态** — Hooks 让函数组件拥有完整能力
3. **逻辑复用更简单** — 自定义 Hook 替代 HOC/render props，无嵌套

**常用 Hooks：**

| Hook        | 作用                  |
| ----------- | --------------------- |
| useState    | 定义状态              |
| useEffect   | 处理副作用            |
| useContext  | 读取 Context          |
| useRef      | 获取 DOM / 保存可变值 |
| useMemo     | 缓存计算结果          |
| useCallback | 缓存函数引用          |
| useReducer  | 复杂状态管理          |

**面试加分点**：Hooks 让函数组件从「无状态」变成「有状态」；能对比 Hooks 和 HOC 在代码简洁性上的差异。

</details>

---

### 12. React 常用 Hooks 有哪些？各自的作用是什么？

<details>
<summary>查看答案</summary>

**七大常用 Hooks：**

1. **useState** — 定义组件内部状态，状态变化触发重新渲染
2. **useEffect** — 处理副作用（数据请求、订阅、DOM 操作），返回清理函数在卸载或依赖变化前执行
3. **useContext** — 读取 React Context，避免层层传 props
4. **useRef** — 获取 DOM 引用或保存不触发渲染的可变值（修改 .current 不会重新渲染）
5. **useMemo** — 缓存复杂计算结果，避免每次渲染重新计算
6. **useCallback** — 缓存函数引用，配合 React.memo 优化子组件渲染
7. **useReducer** — 复杂状态管理，多状态联动场景

**是否触发渲染：**

- 触发渲染：useState、useReducer
- 不触发渲染：useEffect、useContext、useRef、useMemo、useCallback

**面试加分点**：按使用频率记忆 — useState 最常用，useEffect 次之，useMemo/useCallback 用于性能优化。

</details>

---

### 13. 使用 Hooks 有哪些基本规则？

<details>
<summary>查看答案</summary>

**三条规则：**

1. **只在最顶层调用** — 不能在 if、for、嵌套函数中调用，确保每次渲染调用顺序一致
2. **只在 React 函数中调用** — 函数组件或自定义 Hook 中，不能在普通 JS 函数中调用
3. **自定义 Hook 必须以 use 开头** — 让 ESLint 能按 Hook 规则检查

**原因：** React 通过 Hook 的调用顺序来对应每个 Hook 的状态。如果顺序变化（如条件调用），React 无法正确匹配状态。

**ESLint 规则：**

- `react-hooks/rules-of-hooks` — 检查调用位置
- `react-hooks/exhaustive-deps` — 检查依赖完整性

**面试加分点**：Hooks 依赖调用顺序（链表结构）；ESLint 的两条规则分别检查什么。

</details>

---

## 三、组件通信与状态提升

### 14. React 中有哪些组件通信方式？

<details>
<summary>查看答案</summary>

**六种方式：**

| 方式                          | 适用场景                            |
| ----------------------------- | ----------------------------------- |
| Props                         | 父子通信                            |
| 状态提升                      | 兄弟/少量共享（放到最近公共父组件） |
| Context                       | 跨多层且变化不频繁                  |
| ref + forwardRef              | 调用子组件方法                      |
| 全局状态管理（Redux/Zustand） | 跨页面/高频变化                     |
| 事件总线（mitt）              | 不推荐，难维护                      |

**Context 注意事项：**

- Context 值变化会导致所有消费组件重新渲染
- 不要把所有状态放一个 Context
- 可用 useMemo 包装 value，拆分 Provider

**面试加分点**：Context 不是状态管理方案，只是依赖注入；能区分状态提升和全局状态管理的适用边界。

</details>

---

### 15. Context 有什么性能问题？如何优化？

<details>
<summary>查看答案</summary>

**性能问题：** Provider 的 value 变化时，所有消费该 Context 的组件都会重新渲染，即使只用了其中一部分数据。

**四种优化方案：**

1. **拆分 Context** — 按主题拆分（UserContext、ThemeContext 分开），消费者只订阅需要的
2. **useMemo 稳定 value** — 避免每次渲染创建新对象引用
3. **组件内按需消费** — 只消费需要的 Context，而非一个大 Context
4. **使用状态管理库** — Zustand/Redux 有更细粒度的订阅机制

**面试加分点**：Context 的重新渲染是「全量通知」机制；最佳实践是按主题拆分 Context。

</details>

---

### 16. `React.memo`、`useMemo` 和 `PureComponent` 有什么区别？

<details>
<summary>查看答案</summary>

**对比表：**

| API           | 层级     | 比较方式              | 使用场景         |
| ------------- | -------- | --------------------- | ---------------- |
| PureComponent | 类组件   | 浅比较 props 和 state | 类组件性能优化   |
| React.memo    | 函数组件 | 浅比较 props          | 函数组件性能优化 |
| useMemo       | Hook     | 缓存计算结果          | 缓存昂贵计算     |
| useCallback   | Hook     | 缓存函数引用          | 稳定子组件 props |

**关键点：**

- React.memo 支持自定义比较函数（第二个参数）
- 浅比较只比较引用 → 必须用不可变数据才能生效
- React.memo 默认不比较 state 和 context 变化

**面试加分点**：浅比较只比较引用，不变异数据才能生效；React.memo 不阻止 context 导致的重新渲染。

</details>

---

## 四、渲染机制与性能

### 17. 解释 React 的 Diff 算法和 Key 的作用。

<details>
<summary>查看答案</summary>

**Diff 算法三个假设：**

1. 不同类型的元素产生不同的树 → 直接销毁重建
2. 同类型元素只更新变化的属性，保留 DOM 节点
3. 通过 key 判断哪些子元素是同一份 → 复用而非重建

**核心策略：** 同层比较，不跨层级移动节点

**Key 的作用：**

- 帮助 React 识别列表项的增删改
- key 相同但位置变化 → React 移动 DOM 而非销毁重建
- 没有 key 或用 index → 列表变化时导致不必要的 DOM 操作和状态错乱

**不建议用 index 做 key：** 列表会排序/过滤/增删时、列表项有受控组件时

**可以用 index：** 静态列表、纯展示、永不变化

**面试加分点**：key 是给 React diff 算法用的，不是给开发者看的；key 相同位置变化时 React 会移动 DOM。

</details>

---

### 18. 什么是虚拟 DOM？React 为什么要用它？

<details>
<summary>查看答案</summary>

**定义：** 用 JavaScript 对象描述真实 DOM 结构的轻量表示。JSX 编译为 createElement 调用，生成虚拟 DOM 对象。

**四个好处：**

1. **跨平台** — 虚拟 DOM 是平台无关的描述，可映射到 Web、Native、Canvas
2. **批量更新** — 多次 DOM 操作合并为一次，减少重排重绘
3. **Diff 算法** — 只更新真正变化的部分
4. **声明式编程** — 开发者描述 UI 该长什么样，React 负责高效更新

**虚拟 DOM 一定快吗？** 不一定。直接操作 DOM 在简单场景可能更快。虚拟 DOM 的优势在于可维护性、可预测性，以及复杂场景下的整体性能。

**更新流程：** JSX → 虚拟 DOM → Diff → 最小化真实 DOM 更新

**面试加分点**：虚拟 DOM 是手段不是目的；React 的优势在运行时调度，Vue 在编译时优化。

</details>

---

### 19. React 18 的并发特性 `startTransition` 和 `useDeferredValue` 怎么用？

<details>
<summary>查看答案</summary>

**问题场景：** 搜索框输入时，输入响应是紧急更新，搜索结果列表渲染是非紧急更新。列表渲染阻塞输入会导致卡顿。

**两个 API：**

| 特性     | startTransition        | useDeferredValue |
| -------- | ---------------------- | ---------------- |
| 控制对象 | state 更新函数         | 某个值           |
| 使用位置 | 事件处理中             | 组件内部         |
| 典型场景 | 区分输入和搜索结果更新 | 子组件接收延迟值 |

**核心思想：**

- startTransition 包裹的更新标记为「可中断的低优先级」
- useDeferredValue 返回一个延迟版本的值，高优先级更新完成后才更新
- 配合 `useTransition` 的 `isPending` 可以显示 loading 状态

**面试加分点**：并发渲染允许 React 暂停低优先级工作，优先响应用户输入；能区分 urgent update 和 transition update。

</details>

---

## 五、React Router

### 20. React Router v6 相比 v5 有哪些重要变化？

<details>
<summary>查看答案</summary>

**七大变化：**

1. **Switch → Routes** — 更语义化
2. **component/render → element** — 直接传 JSX 而非组件引用
3. **useHistory → useNavigate** — API 更简洁，支持 navigate(-1) 返回
4. **嵌套路由声明式配置** — 子路由直接嵌套在父路由内，配合 `<Outlet />` 渲染
5. **默认精确匹配** — 不再需要 exact 属性
6. **通配符用 `*`** — 子路由需在父路由也写 `*`
7. **Prompt 组件移除** — 拦截离开需用 useBlocker 或第三方方案

**面试加分点**：v6 的 API 更偏向 JSX element，路由配置更声明式；嵌套路由中 `<Outlet />` 充当子路由的占位渲染。

</details>

---

### 21. 路由懒加载如何实现？

<details>
<summary>查看答案</summary>

**方案：** React.lazy + Suspense

- `React.lazy(() => import('./Page'))` — 动态导入组件
- `<Suspense fallback={<Loading />}>` — 包裹懒加载组件，提供加载态

**与 Vue Router 对比：**

| 特性     | Vue Router       | React Router                 |
| -------- | ---------------- | ---------------------------- |
| 语法     | `() => import()` | `React.lazy(() => import())` |
| 加载态   | 路由级配置       | Suspense fallback            |
| 错误处理 | 路由钩子         | ErrorBoundary                |

**进阶：** 可通过 `onMouseEnter` 预加载即将访问的页面组件

**面试加分点**：React.lazy 必须配合 Suspense 使用；动态 import 会被打包工具自动 code-splitting。

</details>

---

## 六、状态管理

### 22. Redux 和 Zustand 有什么区别？

<details>
<summary>查看答案</summary>

**对比表：**

| 维度     | Redux Toolkit  | Zustand            |
| -------- | -------------- | ------------------ |
| 学习曲线 | 较陡           | 平缓               |
| 代码量   | 较多           | 较少               |
| 中间件   | 丰富           | 较少但够用         |
| TS 支持  | 好             | 优秀               |
| 订阅粒度 | 连接整个 state | 可按 selector 订阅 |
| 适用规模 | 大型复杂应用   | 中小型快速开发     |

**选择建议：**

| 场景                             | 推荐         |
| -------------------------------- | ------------ |
| 大型应用、多人协作、需要调试工具 | Redux        |
| 快速开发、轻量状态、Hooks 友好   | Zustand      |
| 原子化状态、细粒度订阅           | Jotai/Recoil |
| 简单全局状态                     | Context      |

**面试加分点**：Redux 三原则 — 单一数据源、state 只读、纯函数修改；Zustand 无需 Provider 包裹是其设计优势。

</details>

---

## 七、进阶与工程化

### 23. 什么是高阶组件（HOC）和 Render Props？现在还用吗？

<details>
<summary>查看答案</summary>

**HOC：** 接收一个组件，返回一个增强后的新组件。用于复用横切关注点（如日志、权限、数据获取）。

**Render Props：** 通过 props 传递一个渲染函数，由使用者决定渲染什么内容。

**与 Hooks 对比：**

| 方案         | 优点                 | 缺点                 |
| ------------ | -------------------- | -------------------- |
| HOC          | 可复用、可组合       | 嵌套地狱、props 冲突 |
| Render Props | 灵活                 | 嵌套复杂             |
| Hooks        | 简洁、无嵌套、易测试 | 需遵循规则           |

**现状：** Hooks 出现后大部分场景被自定义 Hook 替代；HOC 仍在第三方库中使用；Render Props 在动态渲染场景仍有价值。

**面试加分点**：HOC 和 Render Props 解决「逻辑复用」问题，Hooks 是更现代的方案；HOC 常见问题 — displayName 丢失、props 覆盖。

</details>

---

### 24. React 项目有哪些常见性能优化手段？

<details>
<summary>查看答案</summary>

**十大优化手段：**

1. **避免不必要渲染** — React.memo、useMemo、useCallback
2. **代码分割** — React.lazy + Suspense 按路由/组件拆分
3. **虚拟列表** — react-window/react-virtualized 处理长列表
4. **避免 render 中创建新对象/函数** — 用 useMemo/useCallback 缓存
5. **合理使用 Context** — 拆分 Context、useMemo 稳定 value
6. **图片懒加载** — `loading="lazy"` 属性
7. **React 18 并发特性** — startTransition、useDeferredValue
8. **避免滥用 useEffect** — 数据获取考虑 React Query/SWR
9. **生产构建优化** — gzip/brotli、CDN、合理 code splitting
10. **性能分析工具** — React DevTools Profiler、Chrome Performance

**面试加分点**：强调「先测量再优化」；区分 React 18 自动优化（并发、批处理）和手动优化（memo、虚拟列表）。

</details>

---

## 使用说明

- 本文档为**口述版**，删除了所有多行代码块，以要点、表格和一句话思路替代
- 适用于面试前快速复习和口头表达练习
- 详细代码示例版请参考 `react面试题.md`
