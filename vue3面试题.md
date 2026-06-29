# Vue 3 面试题（含隐藏答案 · 详细版）

共 **30 题**，涵盖基础、响应式、Composition API、组件通信、Router、Pinia、进阶特性、Diff 算法、新版本特性与高阶 API。点击「查看答案」即可展开，答案以**口述要点 / 对比 / 核心原理 / 坑点 / 面试加分点**为主，代码已精简，适合面试直接回答。

---

## 一、基础与核心概念

### 1. Vue 3 相比 Vue 2 有哪些主要变化？

<details>
<summary>查看答案</summary>

**1）Composition API（最大变化）**

- 新增 `setup`、`ref`、`reactive`、`computed`、`watch` 等组合式 API
- 解决 Vue 2 中 Options API 在大型组件下「逻辑碎片化」（同一功能被拆到 data/methods/computed 等不同位置）
- 逻辑可以按功能聚合、跨组件复用（Composables 替代 mixin）

**2）响应式系统重写：`Object.defineProperty` → `Proxy`**

| 维度     | Vue 2                                    | Vue 3                  |
| -------- | ---------------------------------------- | ---------------------- |
| 实现     | `Object.defineProperty` 递归劫持每个属性 | `Proxy` 代理整个对象   |
| 新增属性 | 检测不到，需 `Vue.set`                   | 直接监听               |
| 删除属性 | 检测不到，需 `Vue.delete`                | `deleteProperty` 拦截  |
| 数组     | 重写 7 个变更方法                        | 索引/length 都能监听   |
| 嵌套对象 | 初始化时一次性递归                       | 访问时才代理（懒代理） |
| 性能     | 初始化慢、内存高                         | 更快、更省内存         |

**3）性能提升**

- **静态提升（hoistStatic）**：静态节点提到 render 函数外，避免重复创建
- **PatchFlag**：编译期标记动态内容，运行时 diff 只关注动态部分
- **Block Tree**：把动态节点拍平到 Block 数组，跳过静态层级
- **Tree-shaking**：API 全部具名导出，未使用的可被打包工具剔除（包体可减小 50%+）

**4）TypeScript 支持**

- 整个源码用 TS 重写
- props、emits、ref、computed 类型推断更自然
- `<script setup lang="ts">` 体验接近原生 TS

**5）新内置组件 / 特性**

- `<Teleport>`：把 DOM 渲染到组件树外（弹窗常用）
- `<Suspense>`：异步组件 loading 状态管理
- **Fragment**：模板支持多个根节点
- 多个 `v-model`、自定义修饰符
- `<script setup>` 编译宏
- `defineCustomElement`：原生 Web Components 支持

**6）破坏性 API 调整**

- 移除 `$on/$off/$once`（事件总线模式不再推荐，改用第三方库 mitt）
- 移除 filter（用 computed 或 method 代替）
- `v-if` 优先级高于 `v-for`（Vue 2 是反过来）
- 全局 API 调整：`new Vue()` → `createApp()`，`Vue.component` → `app.component`

**面试加分点**：能讲清「为什么改用 Proxy」（解决 Vue 2 三个老痛点：新增属性、删除属性、数组索引），并提到编译优化（PatchFlag/Block）是 Vue 3 性能提升的真正大头。

</details>

---

### 2. `ref` 和 `reactive` 有什么区别？分别适合什么场景？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 维度       | `ref`                                          | `reactive`                      |
| ---------- | ---------------------------------------------- | ------------------------------- |
| 接收类型   | 任意（基本类型 + 对象）                        | 仅对象/数组/Map/Set             |
| 底层实现   | `RefImpl` 类，`{ value }` 包装 + getter/setter | `Proxy` 代理                    |
| JS 中访问  | 必须 `.value`                                  | 直接 `obj.x`                    |
| 模板中访问 | 自动解包（顶层 ref）                           | 直接访问                        |
| 重新赋值   | `count.value = 100` ✅                         | `state = {...}` ❌（丢响应式）  |
| 解构       | 解构出来还是 ref                               | 解构后丢失响应式（需 `toRefs`） |
| 嵌套对象   | 内部对象会自动 `reactive` 化                   | 全部递归代理                    |

**底层关系**：`ref(obj)` 内部其实就是 `reactive(obj)` 包了一层 `.value`。

**场景选择：**

✅ 用 `ref`：

- 基本类型：`const count = ref(0)`
- 整体替换的对象/数组：`const list = ref([])`，后续 `list.value = await fetch(...)`
- 在 Composables 中作为返回值（更易解构、模板自动解包）
- DOM 引用：`<div ref="el">` + `const el = ref(null)`

✅ 用 `reactive`：

- 一组关联状态：`const form = reactive({ name: '', age: 0 })`
- 不需要整体替换的复杂对象
- 类似 Vue 2 `data` 的写法

**典型坑：**

| 操作                                       | 结果             | 正确做法                                           |
| ------------------------------------------ | ---------------- | -------------------------------------------------- |
| `state = { list: [1, 2] }`                 | 丢失响应式       | 改属性 `state.list = [1, 2]`                       |
| `const { count } = reactive({ count: 0 })` | 基本类型断响应式 | `const { count } = toRefs(reactive({ count: 0 }))` |

**面试加分点**：官方推荐**优先用 `ref`**——理由是统一心智模型（不用纠结）、可整体替换、与 Composables 配合更好。

</details>

---

### 3. 为什么 `reactive` 解构会丢失响应式？如何解决？

<details>
<summary>查看答案</summary>

**根本原因**：`reactive` 返回的是 Proxy 对象，响应式追踪发生在访问 Proxy 属性时。解构时 `const { count } = state` 等价于 `const count = state.count`，对于基本类型会取出普通值，与 Proxy 断开，失去响应式。

对象类型解构出来的是引用，不重新赋值仍保持响应式；基本类型会立即断掉。

**解决方案：**

1. **`toRefs`**：整体转换，`const { count } = toRefs(state)`，得到 ref
2. **`toRef`**：单个属性，`const count = toRef(state, 'count')`
3. **不解构**：模板中直接用 `state.count`
4. **Composables 返回 `toRefs(state)`**：调用方可以安全解构
5. **Pinia 中用 `storeToRefs`**：state/getter 转 ref，action 直接解构

**面试加分点**：`toRefs` 内部遍历对象，对每个 key 调用 `toRef`，创建一个 `ObjectRefImpl`，其 `.value` 的 getter/setter 直接读写原对象属性，本质是用 ref 壳包住对原 reactive 的引用。

</details>

---

### 4. `computed` 和 `watch` / `watchEffect` 的区别？

<details>
<summary>查看答案</summary>

**核心区别：**

| 维度     | computed          | watch                    | watchEffect           |
| -------- | ----------------- | ------------------------ | --------------------- |
| 用途     | 派生值            | 监听变化执行副作用       | 副作用 + 自动收集依赖 |
| 返回值   | 有                | 无                       | 无                    |
| 缓存     | ✅ 依赖不变不重算 | ❌                       | ❌                    |
| 依赖收集 | 自动              | 显式指定源               | 自动                  |
| 执行时机 | 惰性读取          | 默认惰性，可 `immediate` | 立即执行              |
| 旧值     | ❌                | ✅                       | ❌                    |
| 异步     | 不推荐            | 推荐                     | 可以                  |

**使用原则：**

- **computed**：由 a、b 算出 c 显示给模板，必须是纯函数
- **watch**：需要监听特定源、拿新旧值、发请求、改 localStorage
- **watchEffect**：副作用依赖多个值懒得列，或同步副作用

**注意点：**

- `computed` 里不要写副作用
- `watch` 监听 reactive 整个对象自动 deep；监听子属性用 getter
- `watchEffect` 中 await 之后的访问不会被追踪
- `watch` 的 `flush` 可选 `pre` / `post` / `sync`

**面试加分点**：知道 `watchPostEffect` / `watchSyncEffect` 快捷方式；能根据场景选择。

</details>

---

### 5. Vue 3 的生命周期钩子有哪些？与 Vue 2 如何对应？

<details>
<summary>查看答案</summary>

**对应关系总表：**

| Vue 2             | Vue 3 Options     | Vue 3 Composition   | 触发时机                                    |
| ----------------- | ----------------- | ------------------- | ------------------------------------------- |
| beforeCreate      | —                 | —                   | setup() 前（基本不用）                      |
| created           | —                 | —                   | setup() 中即可（同步代码就是 created 阶段） |
| beforeMount       | beforeMount       | onBeforeMount       | 渲染函数首次调用前                          |
| mounted           | mounted           | onMounted           | DOM 挂载后                                  |
| beforeUpdate      | beforeUpdate      | onBeforeUpdate      | 响应式数据变更，DOM 更新前                  |
| updated           | updated           | onUpdated           | DOM 更新完成                                |
| **beforeDestroy** | **beforeUnmount** | **onBeforeUnmount** | 卸载前（改名）                              |
| **destroyed**     | **unmounted**     | **onUnmounted**     | 卸载后（改名）                              |
| activated         | activated         | onActivated         | KeepAlive 激活                              |
| deactivated       | deactivated       | onDeactivated       | KeepAlive 失活                              |
| errorCaptured     | errorCaptured     | onErrorCaptured     | 子孙组件抛错                                |

**Vue 3 新增（调试钩子）：**

- `onRenderTracked(e)`：依赖被追踪时（开发调试）
- `onRenderTriggered(e)`：依赖触发更新时（开发调试）
- `onServerPrefetch`：SSR 预取数据

**使用要点：**

- `setup` 中同步代码等价于 `beforeCreate + created`
- 引入 `onMounted` / `onUnmounted` 等钩子注册回调
- Composition 钩子必须在 setup 同步上下文中注册，不能放在 `await` 之后或 `setTimeout` 里

**注意点：**

1. Composition 钩子**必须在 setup 同步上下文中**注册（不能放进 setTimeout、await 之后）
2. 同一个钩子可以注册多次，按注册顺序执行（Composables 中常见）
3. `unmounted` 时机是组件实例被销毁后，此时 DOM 已被移除
4. `onErrorCaptured` 返回 `false` 可阻止错误向上传播

**调用顺序：**

父子组件挂载：父 beforeMount → 子 beforeMount → 子 mounted → 父 mounted

父子组件卸载：父 beforeUnmount → 子 beforeUnmount → 子 unmounted → 父 unmounted

**面试加分点**：能解释「为什么 setup 中没有 beforeCreate / created」——因为 setup 本身在这两个钩子之间执行，把同步代码写在 setup 里就等价。

</details>

---

## 二、Composition API 与 `<script setup>`

### 6. `<script setup>` 相比普通 `setup()` 有什么优势？

<details>
<summary>查看答案</summary>

**核心优势：**

1. **更简洁**：顶层变量自动暴露给模板，不用 return
2. **编译宏**：`defineProps` / `defineEmits` / `defineExpose` / `defineOptions` / `defineSlots` / `defineModel` 无需 import，只在 `<script setup>` 中可用
3. **TS 体验好**：`defineProps<Props>()` 纯类型声明，推断自然
4. **默认私有**：内部状态父组件不可见，需 `defineExpose` 显式暴露
5. **性能更好**：编译期静态分析更多，减少运行时开销
6. **顶层 await**：可直接 `await`，外层配合 `<Suspense>`
7. **可与普通 `<script>` 共存**：3.3+ 可用 `defineOptions` 替代额外 script

**面试加分点**：自动暴露是编译期实现的；默认私有为了组件封装性。

</details>

---

### 7. `defineProps` 和 `defineEmits` 有什么需要注意的点？

<details>
<summary>查看答案</summary>

**注意点：**

1. **编译宏，无需 import**，不能解构再传参
2. **两种声明**：运行时声明（带校验/默认值）或纯类型声明 `defineProps<Props>()`
3. **默认值**：3.3+ 用 `withDefaults(defineProps<Props>(), { count: 0 })`，引用类型用工厂函数
4. **props 只读**：想修改要 emit 通知父组件，或本地用 `ref + watch` 同步
5. **defineEmits 类型写法**：推荐 3.3+ 元组语法 `change: [id: number, name: string]`
6. **解构 props 会丢响应式**（3.5 之前）：用 `toRefs` / `toRef`；3.5+ 编译期支持响应式解构
7. **defineModel（3.4+）**：`const model = defineModel()` 返回 ref，读写自动 emit

**坑点**：props 默认值若是引用类型必须用工厂函数；类型声明形式 3.2 前不支持外部 import 复杂类型。

</details>

---

### 8. 如何在 Vue 3 中实现逻辑复用？Composables 是什么？

<details>
<summary>查看答案</summary>

**定义**：Composables 是利用 Composition API 把有状态逻辑封装成 `useXxx` 函数的模式。

**相比 Vue 2 复用方案：**

| 方案            | 来源清晰 | 命名冲突 | 类型推断 | Tree-shaking |
| --------------- | -------- | -------- | -------- | ------------ |
| Mixin           | ❌       | ❌       | ❌       | ❌           |
| HOC             | 一般     | 一般     | 一般     | 一般         |
| Renderless 组件 | ✅       | ✅       | 一般     | 一般         |
| Composables     | ✅       | ✅       | ✅       | ✅           |

**典型场景**：计数器、鼠标位置、异步数据获取、localStorage 封装等。

**最佳实践**：

- 以 `use` 开头命名
- 返回 ref 集合，便于调用方解构
- 内部副作用在 `onUnmounted` 清理
- 异步逻辑返回响应式状态（data/error/loading），而不是直接 await

**生态**：VueUse 提供 200+ 现成 Composables。

**面试加分点**：Composable 和普通工具函数的区别在于有状态、内部调用响应式 API；必须在 setup 同步上下文中调用。

</details>

---

### 9. `shallowRef` 和 `shallowReactive` 是什么？什么时候用？

<details>
<summary>查看答案</summary>

**核心区别：浅层响应式，只追踪第一层**

| API             | 触发更新条件                         |
| --------------- | ------------------------------------ |
| ref             | `.value` 替换 + 内部对象任意层级变化 |
| shallowRef      | 仅 `.value` 整体替换                 |
| reactive        | 对象任意层级属性变化                 |
| shallowReactive | 仅第一层属性变化                     |

**使用场景：**

1. **大型不可变数据**：表格几千行，用 `shallowRef` 整体替换
2. **第三方实例**：ECharts、地图 SDK，用 `shallowRef` 或 `markRaw` 避免代理
3. **性能优化**：明确不需要深层响应式时
4. **手动触发**：`shallowRef` 修改深层后可用 `triggerRef` 强制触发

**相关 API**：`markRaw`（永久不代理）、`toRaw`（拿原始对象）、`triggerRef`（强制触发）、`customRef`（自定义）

**坑点**：`shallowReactive` 嵌套对象完全无响应式；`markRaw` 不可逆。

**面试加分点**：`reactive` 的递归代理是懒的；`shallowRef`/`shallowReactive` 是永远不代理深层。

</details>

---

## 三、模板、指令与组件通信

### 10. Vue 3 中 `v-model` 有什么变化？

<details>
<summary>查看答案</summary>

**与 Vue 2 的差异：**

|              | Vue 2                 | Vue 3               |
| ------------ | --------------------- | ------------------- |
| 默认 prop    | `value`               | `modelValue`        |
| 默认事件     | `input`               | `update:modelValue` |
| 多个 v-model | ❌（用 .sync）        | ✅ `v-model:title`  |
| 修饰符       | 内置 lazy/number/trim | 支持自定义修饰符    |

**核心变化**：Vue 3 默认 prop 改为 `modelValue`，事件改为 `update:modelValue`，自定义组件要实现双向绑定就按这个对应。

**多个 v-model**：`v-model:name="name"` 等价于 `:name="name" @update:name="name = $event"`

**自定义修饰符**：子组件通过 `modelModifiers` prop 接收修饰符，自己处理转换逻辑。

**defineModel（3.4+ 推荐）**：`const model = defineModel()` 返回一个 ref，读写 `.value` 自动触发 emit，代码最简。

**原生 input 的 v-model：**

| 元素           | prop    | event  |
| -------------- | ------- | ------ |
| text           | value   | input  |
| checkbox/radio | checked | change |
| select         | value   | change |

**面试加分点**：Vue 3 去掉了 `.sync` 合并到 v-model；defineModel 大幅简化了双向绑定代码量。

</details>

---

### 11. `provide` / `inject` 的使用场景和注意事项？

<details>
<summary>查看答案</summary>

**作用**：跨任意层级组件传数据，避免 prop 一层层透传（prop drilling）。

**典型场景**：主题切换、国际化、表单上下文（如 ElementPlus 的 Form/FormItem）、全局用户信息/权限、弹窗根容器。

**基础用法**：父组件 `provide(key, value)`，后代组件 `inject(key, default)`。

**关键注意点：**

1. **必须 provide ref/reactive 才响应式**，普通值不响应
2. **默认值**：`inject('key', default)`，引用类型默认值用工厂函数 `inject('key', () => ({}), true)`
3. **只读保护**：用 `readonly(theme)` 提供，改值走专用方法
4. **Symbol key**：避免命名冲突，TS 能自动推断类型
5. **应用级 provide**：`app.provide('apiBase', '/api')`
6. **可封装成 Composable**：把 provide/inject 隐藏到 `useXxx()` 里，调用方无感知

**何时不用：**

- 跨页面/全局共享 → Pinia
- 父子之间 → props/emit
- 兄弟组件 → 提升父级或 Pinia

**面试加分点**：provide/inject 基于组件树查找，不是 DOM 树；slot 内容的 inject 解析在外层模板所有者组件中；建议 readonly + Symbol key。

</details>

---

### 12. `Teleport` 是做什么的？常见使用场景？

<details>
<summary>查看答案</summary>

**作用**：把组件模板里的一段 DOM 传送到组件树外部的指定节点渲染，但逻辑上仍属于原组件（状态、事件、provide/inject 都正常）。

**为什么需要**：弹窗、Toast 等浮层常被父级的 `overflow:hidden`、`transform`、z-index 影响，挂到 body 下最干净。

**基本用法**：`<Teleport to="body"><div v-if="open" class="modal">...</div></Teleport>`。`to` 支持 CSS 选择器、id、DOM 元素；可用 `:disabled` 条件传送；可配合 `<Transition>`。

**典型场景**：Modal、Toast、Tooltip、Loading 遮罩、右键菜单。

**坑点**：

- 目标元素必须已存在 DOM 中
- SSR 注意 hydration 错位

**面试加分点**：逻辑归属与 DOM 位置分离，类似 React Portal，但更声明式。

</details>

---

### 13. Vue 3 支持多根节点（Fragment），有什么影响？

<details>
<summary>查看答案</summary>

**Vue 2**：模板必须有**唯一根元素**，否则编译报错；**Vue 3** 支持多个并列根节点，编译器会自动包成不渲染真实 DOM 的 **Fragment** 节点。

**好处：**

- 减少无意义包裹元素，HTML 更简洁
- 表格、列表里可直接返回一组 `<tr>`，不用套额外容器

**带来的影响：**

| 方面           | 单根节点       | 多根节点                                          |
| -------------- | -------------- | ------------------------------------------------- |
| `$attrs` 透传  | 自动落到根节点 | 不自动透传，需手动 `v-bind="$attrs"` 指定         |
| `<Transition>` | 正常包裹       | 不支持多根，需用 `<TransitionGroup>` 或包一层容器 |
| `ref` 引用     | 直接拿到根 DOM | 拿到组件实例代理，需给具体节点加 ref              |
| `v-for` diff   | 常规           | 建议加稳定 key，减少无容器带来的 diff 开销        |

**注意点：**

- 多根时父组件传的 `class`、事件监听器会丢失，开发环境会有警告
- 不需要自动透传时可设 `inheritAttrs: false`

**面试加分点**：能讲清「Fragment 是编译期生成的虚拟节点（VNode），运行时其 children 直接铺开渲染，没有对应 DOM」；提到属性透传需 `inheritAttrs: false` + `v-bind="$attrs"` 的搭配用法。

</details>

---

## 四、响应式原理（进阶）

### 14. 简要说明 Vue 3 响应式原理（Proxy 方案）

<details>
<summary>查看答案</summary>

**一句话概括：** Vue 3 用 ES6 `Proxy` 代理对象，在 `get` 时收集依赖、`set` 时触发更新，配合 `effect` 完成响应式副作用调度。

**三个核心概念：**

| 概念       | 作用                                                             |
| ---------- | ---------------------------------------------------------------- |
| `reactive` | 用 `Proxy` 代理对象/数组，拦截 get/set/deleteProperty/ownKeys 等 |
| `ref`      | 对基本类型做 `{ value }` 包装，内部同样走 track/trigger          |
| `effect`   | 副作用函数，组件渲染、computed、watch 底层都是 effect            |

**依赖收集结构**：`targetMap: WeakMap<target, Map<key, Set<effect>>>`。每个 `(target, key)` 对应一个 `dep`（Set），`get` 时 `track` 把当前 `effect` 加入 dep，`set` 时 `trigger` 取出 dep 中所有 effect 重新执行。

**组件更新链路**：render effect 访问响应式数据 → `track` 收集该 effect → 数据变化 → `trigger` 触发 effect → 重新渲染组件。

**相比 Vue 2 (`Object.defineProperty`) 的优势：**

| 能力            | Vue 2                       | Vue 3          |
| --------------- | --------------------------- | -------------- |
| 新增/删除属性   | 需 `Vue.set` / `Vue.delete` | 自动监听       |
| 数组索引/length | 需重写 7 个方法             | 直接监听       |
| Map/Set         | 不支持                      | 支持           |
| 嵌套对象        | 初始化时递归代理            | 访问时才懒代理 |
| 初始化性能      | O(n)                        | O(1)           |

**常见坑点：**

- 解构 `reactive` 对象会丢失响应式，需用 `toRefs`
- 替换整个 `reactive` 引用会丢失响应式
- `ref` 在 JS 中访问需要 `.value`

**面试加分点**：能说出 `Proxy` 不止拦截 get/set，还包括 has/ownKeys/deleteProperty；能解释组件的 render 函数本身就是一个 effect，所以模板中访问的数据会自动建立依赖关系。

</details>

---

### 15. `nextTick` 是做什么的？什么场景必须用？

<details>
<summary>查看答案</summary>

Vue 响应式数据变化后不会立即更新 DOM，而是把组件 push 到异步更新队列，在下一个微任务里批量去重更新。同一 tick 内多次修改同一数据只触发一次更新。

`nextTick` 就是注册一个回调，在「下一次 DOM 更新完成后」执行。

**典型场景：**

1. 改完数据立即操作 DOM（如滚动到底部）
2. 显示元素后立即 focus
3. 基于真实 DOM 测量尺寸 / 初始化 ECharts 等第三方库
4. 测试中等待视图更新

**调用方式**：`await nextTick()` 或 `nextTick(callback)`

**实现原理**：Vue 更新和 nextTick 都基于 `Promise.resolve()` 微任务，保证 nextTick 在更新之后执行。

**与 watch flush 的关系**：`watch(source, cb, { flush: 'post' })` 等价于 cb 内部 await nextTick。

**面试加分点**：nextTick 基于微任务，比 setTimeout 更早执行；Vue 异步更新是为了批量去重，避免重复 render。

</details>

---

## 五、Vue Router 4

### 16. Vue Router 4 相对 Vue Router 3 有哪些重要变化？

<details>
<summary>查看答案</summary>

**主要变化：**

1. **创建方式**：从 `new VueRouter({ mode: 'history' })` 改为 `createRouter({ history: createWebHistory() })`
2. **mode 变工厂函数**：`createWebHistory` / `createWebHashHistory` / `createMemoryHistory`
3. **`addRoutes` 移除**：改为 `addRoute`，支持 `removeRoute` / `hasRoute`
4. **通配符变化**：V3 用 `path: '*'`，V4 用 `path: '/:pathMatch(.*)*'`
5. **导航守卫**：`next()` 可选，可直接 `return false` / `'/login'` / `{ name: 'login' }`
6. **`<router-view>` 支持作用域插槽**：方便组合 KeepAlive、Transition
7. **`<router-link>` 支持作用域插槽**：可自定义渲染
8. **Composition API**：`useRouter` / `useRoute`
9. **TS 支持完善**：可扩展 RouteMeta 类型
10. **异步路由**：直接 `component: () => import('./X.vue')`

**面试加分点**：mode 拆成工厂函数的好处是按需打包；`useRouter` / `useRoute` 必须在 setup 同步上下文调用。

</details>

---

### 17. 路由懒加载如何实现？有什么好处？

<details>
<summary>查看答案</summary>

路由懒加载就是访问路由时才动态加载对应组件。

**实现方式**：路由配置里 `component: () => import('@/views/Home.vue')`，打包工具识别动态 `import()` 会自动代码分割，每个路由一个 chunk，首次访问时才下载。

**好处：**

1. 首屏 JS 体积小，加载快
2. 不访问的页面不下载
3. 单个路由更新只影响对应 chunk，缓存更细

**进阶方向：**

- `webpackChunkName` 命名 chunk，便于调试
- `webpackPrefetch` 预加载可能访问的下一个路由
- 配合 `<Suspense>` 做 loading 和错误兜底
- 网络抖动时做 chunk 加载失败重试

**面试加分点**：动态 import 的路径要让打包工具能静态分析，不能完全运行时拼接；提到「路由懒加载 + Suspense + Loading」是 Vue 3 常见最佳实践。

</details>

---

## 六、Pinia 与状态管理

### 18. Pinia 相比 Vuex 有什么优势？为什么 Vue 3 推荐 Pinia？

<details>
<summary>查看答案</summary>

Pinia 是 Vue 官方推荐的状态管理，相比 Vuex 4 更简洁、TS 更好、体积更小。

| 维度     | Vuex 4           | Pinia                        |
| -------- | ---------------- | ---------------------------- |
| Mutation | 必须有           | 移除，直接改或在 action 中改 |
| Module   | 嵌套 + namespace | 独立 store，扁平化           |
| TS 支持  | 需手写大量类型   | 自动推断                     |
| 体积     | ~10kb            | ~1kb                         |
| API 风格 | Options          | 类似 Composition API         |

**核心优势：**

1. **无 mutation 样板代码**：action 里直接 `this.count++` 或写 async
2. **setup 风格 store**：可用 `ref`/`computed`/`watch`，和组件写法一致
3. **多个 store 独立文件**：没有 module 嵌套，按需引入
4. **插件生态**：持久化、日志、撤销重做等
5. **store 间可互相引用**：一个 store 里直接调另一个 store

**面试加分点**：Pinia 取消 mutation 是因为 Vue 3 响应式 + DevTools 已能追踪变更；Vuex 进入维护模式。

</details>

---

### 19. Pinia 中 `storeToRefs` 是干什么的？

<details>
<summary>查看答案</summary>

storeToRefs 解决 Pinia store 解构丢响应式的问题。

Pinia store 本质是一个 reactive 对象，直接 `const { count } = store` 会取出普通值，失去响应式。用 `storeToRefs(store)` 可以把 state 和 getters 转成 ref，解构后仍保持响应式。

**使用规则：**

- **state、getters** → 用 `storeToRefs`
- **actions** → 直接从 store 解构（函数不需要响应式包装）

**与 `toRefs` 的区别：**

|          | `toRefs`           | `storeToRefs`                    |
| -------- | ------------------ | -------------------------------- |
| 转换内容 | 所有可枚举属性     | 仅 state + getters，跳过 actions |
| 用途     | 任意 reactive 对象 | 专为 Pinia store 设计            |

**面试加分点**：根本原因和 `reactive` 解构丢响应式一样，都是 Proxy 对象的属性在解构时被取值，断开了与 Proxy 的关系；actions 是函数，不受影响。

</details>

---

## 七、性能、构建与工程化

### 20. Vue 3 有哪些常见性能优化手段？

<details>
<summary>查看答案</summary>

性能优化分 **Vue 3 自带编译优化** 和 **开发实践** 两个层面。

**Vue 3 编译/运行时优化：**

1. **静态提升**：模板里静态节点提到 render 函数外，不复创
2. **PatchFlag**：编译期给动态节点打标记，diff 只比动态部分
3. **Block Tree**：动态节点拍平，diff 不递归整棵树
4. **Tree-shaking**：API 具名导出，未使用代码打包剔除
5. **SSR 优化**：静态内容直接拼字符串

**开发实践优化：**

| 手段                                | 场景                                     |
| ----------------------------------- | ---------------------------------------- |
| `v-show` vs `v-if`                  | 频繁切换用 show，条件渲染用 if           |
| `v-for` 加稳定 key                  | 不用 index，用唯一 id                    |
| `v-memo`                            | 大数据子树条件缓存                       |
| `shallowRef` / `markRaw`            | 大数组、第三方实例避免深层代理           |
| `defineAsyncComponent` + 路由懒加载 | 首屏分包                                 |
| `<KeepAlive>`                       | tab 切换缓存组件实例                     |
| `computed`                          | 派生值缓存                               |
| 虚拟滚动                            | 大列表只渲染可视区                       |
| 防抖/节流                           | 搜索、scroll、resize                     |
| Web Worker                          | 重计算放后台线程                         |
| 图片懒加载                          | 首屏外图片                               |
| 构建优化                            | gzip、CDN、关闭 devtools、sourcemap 按需 |
| `v-once`                            | 只渲染一次的内容                         |
| 避免不必要响应式                    | 常量用 `Object.freeze` / `markRaw`       |

**性能监控工具**：Vue DevTools Performance、Chrome Lighthouse、`onRenderTriggered` 调试多余渲染。

**面试加分点**：区分编译期优化（静态提升/PatchFlag/Block Tree）和运行时/工程优化，根据具体场景给方案。

</details>

---

## 八、补充重点面试题

### 21. Vue 3 的全局 API 有哪些变化？为什么用 createApp？

<details>
<summary>查看答案</summary>

**核心变化：从全局 Vue 对象改为应用实例**

- Vue 2：`new Vue({ router, store, render: h => h(App) }).$mount('#app')`，插件/组件/指令都挂在全局 `Vue` 上
- Vue 3：`createApp(App)` 返回应用实例 `app`，再 `app.use(...)`、`app.component(...)`、`app.mount('#app')`
- 一个页面可创建多个独立应用实例，配置/插件互不干扰

**为什么改成 createApp？**

1. **避免全局污染**：Vue 2 的插件、组件、指令都挂在全局 Vue 上，多实例测试/微前端场景容易冲突
2. **Tree-shaking 友好**：不再依赖 `Vue.xxx` 全局对象，API 可以按需导入
3. **单元测试更友好**：每个测试用例可创建独立 app 实例
4. **微前端支持**：多个 Vue 应用可共存于同一页面

**常见全局 API 调整：**

| Vue 2                    | Vue 3                            |
| ------------------------ | -------------------------------- |
| `Vue.component`          | `app.component`                  |
| `Vue.directive`          | `app.directive`                  |
| `Vue.use`                | `app.use`                        |
| `Vue.mixin`              | `app.mixin`                      |
| `Vue.filter`             | 移除                             |
| `Vue.nextTick`           | `import { nextTick } from 'vue'` |
| `Vue.set` / `Vue.delete` | 不再需要                         |
| `Vue.observable`         | `reactive`                       |

**面试加分点**：能说出 createApp 让每个应用拥有独立的配置、插件、组件注册空间；能解释 Vue 3 移除 filter 是鼓励用 computed 或 method 替代。

</details>

---

### 22. Vue 3 的插槽有哪些？作用域插槽怎么用？

<details>
<summary>查看答案</summary>

**插槽类型**

| 类型       | 父组件写法                                  | 子组件写法                              |
| ---------- | ------------------------------------------- | --------------------------------------- |
| 默认插槽   | `<Child>默认内容</Child>`                   | `<slot></slot>`                         |
| 具名插槽   | `<template #header>头部</template>`         | `<slot name="header"></slot>`           |
| 作用域插槽 | `<Child v-slot="{ user, age }">...</Child>` | `<slot :user="user" :age="age"></slot>` |

**作用域插槽本质**：子组件把数据作为 props 传给父组件的渲染函数，父组件通过 `v-slot="{ ... }"` 解构接收。

**`<script setup>` 中定义插槽**：3.3+ 可用 `defineSlots()` 做类型声明（模板里写 `<slot>` 仍可直接使用）。

**与 Vue 2 的差异**

|              | Vue 2                 | Vue 3                |
| ------------ | --------------------- | -------------------- |
| 默认插槽     | `slot` / `slot-scope` | `v-slot` / `#`       |
| 作用域插槽   | `slot-scope`          | `v-slot="{ data }"`  |
| 动态插槽名   | `#[dynamicName]`      | 支持                 |
| 插槽内容编译 | 编译为函数            | 编译为函数，性能更好 |

**常见坑点**：注意解构命名冲突，如 `v-slot="{ user }"` 与插槽内容里 `v-for="user in list"` 同名会覆盖。

**面试加分点**：能解释作用域插槽的本质是「子组件把数据作为 props 传给父组件的渲染函数」；能区分 `v-slot` 在组件上和 `<template>` 上的用法。

</details>

---

### 23. Vue 3 中如何封装自定义指令？

<details>
<summary>查看答案</summary>

**自定义指令钩子**

Vue 3 指令钩子与组件生命周期对齐：`created` → `beforeMount` → `mounted` → `beforeUpdate` → `updated` → `beforeUnmount` → `unmounted`。

| 常用钩子    | 用途                                                    |
| ----------- | ------------------------------------------------------- |
| `mounted`   | DOM 已插入，常用：聚焦、权限判断、初始化第三方 DOM 插件 |
| `updated`   | 绑定值更新后执行                                        |
| `unmounted` | 清理副作用                                              |

**钩子参数**：`el`（绑定 DOM）、`binding`（含 `value/arg/modifiers/instance`）、`vnode`、`prevVnode`。

**典型场景**：

- `v-focus`：mounted 中 `el.focus()`，输入框自动聚焦
- `v-permission`：根据 `binding.value` 判断角色，无权限则 `el.remove()`
- `v-lazy`：图片懒加载，监听 scroll 或 IntersectionObserver

**注册方式**：

- 局部：`<script setup>` 中 `import { vXxx } from './directives/xxx'`，模板里 `v-xxx`
- 全局：`app.directive('focus', { mounted(el) { el.focus() } })`

**面试加分点**：能说出 Vue 3 指令钩子从 `bind/inserted/update/unbind` 改为与组件生命周期一致；能解释 `binding.value` 和 `binding.arg` 的区别。

</details>

---

### 24. `<Suspense>` 和 `defineAsyncComponent` 怎么用？

<details>
<summary>查看答案</summary>

**`<Suspense>` 作用**

Vue 3 新增内置组件，在异步依赖（异步组件、顶层 `await` 的 `<script setup>`）解析期间显示 fallback 内容。

**基本用法**：`<Suspense>` 包两个插槽——`#default` 放异步组件，`#fallback` 放 Loading。

**`defineAsyncComponent`**

- 最简：`defineAsyncComponent(() => import('./AsyncComp.vue'))`
- 完整配置可指定 `loadingComponent`、`errorComponent`、`delay`、`timeout`、`onError` 重试逻辑

**异步 setup**：`<script setup>` 里顶层写 `await fetch(...)` 会隐式触发外层 Suspense。

**与路由懒加载结合**：路由 `component: () => import('@/views/About.vue')` 天然异步，配合 `<router-view v-slot="{ Component }">` + `<Suspense>` 实现切换 Loading 兜底。

**面试加分点**：能说出 `<Suspense>` 目前只支持一个默认插槽和一个 fallback 插槽；能解释异步组件配合路由懒加载实现代码分割。

</details>

---

### 25. Vue 3 中的 `<KeepAlive>` 是什么？怎么控制缓存？

<details>
<summary>查看答案</summary>

**作用**

缓存动态组件实例，避免切换时反复创建/销毁，保留组件状态。

**基本用法**：`<KeepAlive><component :is="currentTab" /></KeepAlive>`

**控制缓存**

| 属性      | 作用                                          |
| --------- | --------------------------------------------- |
| `include` | 只缓存匹配的组件名，可传逗号字符串或数组      |
| `exclude` | 不缓存匹配的组件名                            |
| `max`     | 限制缓存数量，超出按 LRU 销毁最久未访问的实例 |

**配合路由缓存**：`<router-view v-slot="{ Component }"><KeepAlive><component :is="Component" /></KeepAlive></router-view>`

**生命周期钩子**：被缓存组件激活/失活时触发 `onActivated` / `onDeactivated`，可用来恢复滚动、刷新数据等。

**注意点**：

- 子组件必须有 `name` 选项（或文件名，`<script setup>` 中用 `defineOptions({ name: 'Home' })`）
- 缓存的是组件实例，不是 DOM
- 异步组件首次加载后也会被缓存

**面试加分点**：能说出 KeepAlive 适合 tab 切换、表单填写一半切换等场景；能解释 `include`/`exclude` 匹配组件的 `name` 选项。

</details>

---

---

## 九、高频进阶面试题

### 26. Vue 3 的 Diff 算法原理？与 Vue 2 有什么区别？

<details>
<summary>查看答案</summary>

**一句话概括：** Vue 3 采用「快速 Diff」算法，在双端对比基础上引入最长递增子序列（LIS）处理乱序节点，配合编译期 PatchFlag 跳过静态节点，diff 效率大幅提升。

**Diff 流程五步走：**

1. **同序列头部对比**：从左端开始，相同类型 + key 的节点直接 patch，指针后移
2. **同序列尾部对比**：从右端开始，相同的 patch，指针前移
3. **仅新增节点**：旧序列已遍历完，新序列有剩余 → 直接挂载
4. **仅删除节点**：新序列已遍历完，旧序列有剩余 → 直接卸载
5. **乱序部分（核心）**：建立新序列 key → index 的映射表，求出最长递增子序列（LIS），属于 LIS 的节点不动，其余移动/新建/删除

**与 Vue 2 Diff 的对比：**

| 维度     | Vue 2                                  | Vue 3                        |
| -------- | -------------------------------------- | ---------------------------- |
| 算法     | 双端交叉对比（头头、尾尾、头尾、尾头） | 双端同序列 + LIS 处理乱序    |
| 静态节点 | 全量 diff                              | PatchFlag 标记，跳过静态节点 |
| 树结构   | 递归遍历整棵树                         | Block Tree 拍平动态节点      |
| 移动策略 | 逻辑简单，移动次数可能多               | LIS 保证最少移动             |
| 编译优化 | 无                                     | 静态提升 + PatchFlag + Block |

**PatchFlag 的作用：**

编译器在生成 VNode 时给动态节点打上标记位（如 TEXT=1、CLASS=2、STYLE=4、PROPS=8），运行时 diff 时只比较标记对应的动态部分，而非所有 props。

**Block Tree 机制：**

模板编译时把动态节点收集到最近的 Block（根节点、v-if、v-for 都会创建 Block）的 `dynamicChildren` 数组里，diff 时直接遍历这个扁平数组，不用递归整棵树。

**坑点：**

- v-for 不加 key 或用 index 作 key，会退化为就地 patch，导致状态混乱
- 动态 key 变化会导致节点无法复用，全量重建
- v-if / v-for 嵌套会产生新 Block，影响 Block Tree 拍平效果

**面试加分点：** 能说出「最长递增子序列」的作用是找出不需要移动的最大节点集合，剩余节点才做 DOM 移动，从而保证 DOM 操作最少。能区分「编译期优化」和「运行时 Diff」是两个独立层级的优化。

</details>

---

### 27. Vue 3 组件通信方式有哪几种？各适用什么场景？

<details>
<summary>查看答案</summary>

**通信方式总表：**

| 方式                 | 方向              | 适用场景                           | 备注                               |
| -------------------- | ----------------- | ---------------------------------- | ---------------------------------- |
| `props` / `emit`     | 父 → 子 / 子 → 父 | 父子之间常规数据流                 | 最基本、最推荐                     |
| `v-model`            | 双向              | 表单组件双向绑定                   | 本质是 props + emit 语法糖         |
| `provide` / `inject` | 祖先 → 后代       | 跨层级（主题、国际化、表单上下文） | 避免 prop drilling                 |
| `Pinia`              | 任意组件          | 全局/跨页面共享状态                | 官方推荐状态管理                   |
| `$attrs`             | 父 → 子(透传)     | 二次封装组件透传属性               | 配合 `inheritAttrs: false`         |
| `expose` / `ref`     | 父 → 子(命令式)   | 父组件主动调用子组件方法           | `<script setup>` 需 `defineExpose` |
| `mitt`（第三方）     | 任意组件          | 兄弟/跨层级事件通知                | Vue 3 移除了 `$on/$off`            |
| `作用域插槽`         | 子 → 父(渲染)     | 子组件向父组件暴露数据用于渲染     | 如表格组件自定义列                 |

**选择原则（从简单到复杂）：**

1. 父子 → props / emit / v-model
2. 跨层级（同一组件树）→ provide / inject
3. 全局跨页面 → Pinia
4. 偶发事件通知 → mitt
5. 命令式调用 → expose + ref

**与 Vue 2 的变化：**

| Vue 2                    | Vue 3                      |
| ------------------------ | -------------------------- |
| `$on` / `$off` / `$once` | 移除，用 mitt 替代         |
| `.sync` 修饰符           | 合并到 `v-model:xxx`       |
| `$children`              | 移除，用 ref 替代          |
| `$listeners`             | 合并到 `$attrs`            |
| EventBus                 | 不再推荐，用 Pinia 或 mitt |

**坑点：**

- `provide` 普通值无响应式，必须 provide ref/reactive
- `expose` 在 `<script setup>` 中默认什么都不暴露，必须显式声明
- mitt 要在 `onUnmounted` 中 `off`，否则内存泄漏

**面试加分点：** 能根据“谁和谁通信” + “数据流方向”快速给出方案；能解释 Vue 3 移除 `$on/$listeners` 的原因是为了让数据流更可追踪，避免隐式耦合。

</details>

---

### 28. Vue 3.3–3.5 带来了哪些重要新特性？

<details>
<summary>查看答案</summary>

**按版本整理：**

| 版本 | 重要特性                              | 一句话说明                                                                        |
| ---- | ------------------------------------- | --------------------------------------------------------------------------------- |
| 3.3  | `defineOptions`                       | 在 `<script setup>` 中声明组件 name、inheritAttrs 等选项，不再需要额外 `<script>` |
| 3.3  | `defineSlots`                         | 为插槽提供 TS 类型声明                                                            |
| 3.3  | 泛型组件 `<script setup generic="T">` | 组件 props/emit/slots 可用泛型，适合通用组件库                                    |
| 3.3  | `defineEmits` 元组语法                | 更简洁的类型声明：`{ change: [id: number] }`                                      |
| 3.4  | `defineModel`                         | 一行实现双向绑定，返回 ref，读写自动 emit                                         |
| 3.4  | `v-bind` 同名简写                     | `:id` 等价于 `:id="id"`（类似 JS 对象简写）                                       |
| 3.4  | `watch` 的 `once` 选项                | 只触发一次就自动停止                                                              |
| 3.5  | Reactive Props Destructure            | `const { count = 0 } = defineProps<{ count?: number }>()` 解构后仍保持响应式      |
| 3.5  | `useTemplateRef`                      | 显式获取模板 ref，解决变量名和 ref 属性同名的混淆                                 |
| 3.5  | `useId`                               | 生成唯一 ID，SSR/CSR 一致，解决 aria 关联和 hydration 问题                        |
| 3.5  | Deferred Teleport                     | `<Teleport defer>` 延迟到同次更新周期末尾再传送，解决目标元素未渲染问题           |
| 3.5  | `onWatcherCleanup`                    | watch/watchEffect 副作用清理函数，类似 React useEffect 返回值                     |

**对日常开发影响最大的三个：**

1. **defineModel**：把之前 5-8 行的双向绑定样板代码缩减为 1 行
2. **Reactive Props Destructure**：解决了「解构 props 丢响应式」这个 Vue 3 从诹生就存在的坑
3. **defineOptions**：不再需要为了一个 `name` 属性写两个 script 块

**坑点：**

- Reactive Props Destructure 仅在 3.5+ 生效，旧版本解构仍丢响应式
- `defineModel` 返回的 ref 不能解构出属性，必须整体使用 `.value`
- 泛型组件的类型参数不能从外部 import（3.3 限制，后续版本已放开）

**面试加分点：** 能跟进最新版本特性说明你在持续关注 Vue 生态；能解释 defineModel 内部本质是编译宏展开为 `modelValue` prop + `update:modelValue` emit + 一个 ref 同步层。

</details>

---

### 29. `effectScope` 是什么？解决什么问题？

<details>
<summary>查看答案</summary>

**一句话概括：** `effectScope` 是一个“副作用容器”，可以批量收集并统一销毁内部所有响应式副作用（computed、watch、watchEffect）。

**解决的问题：**

在组件外（如全局 store、测试、库开发）创建的 computed/watch 不会自动在组件卸载时清理，导致内存泄漏。`effectScope` 提供了一个统一的生命周期边界。

**核心 API：**

| API                  | 作用                                         |
| -------------------- | -------------------------------------------- |
| `effectScope()`      | 创建 scope 实例                              |
| `scope.run(fn)`      | 在 scope 内执行 fn，内部的 effect 被自动收集 |
| `scope.stop()`       | 停止 scope 内所有 effect                     |
| `getCurrentScope()`  | 获取当前活动的 scope                         |
| `onScopeDispose(fn)` | 在当前 scope 被停止时执行清理回调            |

**使用场景：**

1. **Pinia store 内部**：每个 store 自动拥有一个 scope，`$dispose()` 时统一清理
2. **Composable 库开发**：封装可销毁的功能模块，调用方一句 `stop()` 即可释放全部资源
3. **单元测试**：每个测试用例创建独立 scope，beforeEach 中 stop 上一个
4. **非组件的全局状态**：手动管理生命周期
5. **嵌套 scope**：父 scope stop 会级联停止子 scope，`detached: true` 可分离

**与组件的关系：**

每个组件实例内部自动创建了一个 effectScope，组件卸载时自动 stop，所以组件内的 computed/watch 不用手动清理。`effectScope` 主要是给「组件外」场景用的。

**坑点：**

- `scope.run()` 返回 fn 的返回值，忽略返回值会丢失 computed 的 ref 引用
- `onScopeDispose` 必须在 scope 活动期间调用，否则报警
- detached scope 不会被父 scope 级联清理，必须手动 stop

**面试加分点：** 能说出「Pinia 每个 store 就是一个 effectScope」；能解释「组件内不需要手动用 effectScope，因为组件本身就是一个 scope」。

</details>

---

### 30. 什么场景下需要用 render 函数 / `h()` 而不是模板？

<details>
<summary>查看答案</summary>

**核心区别：**

| 维度     | 模板（Template）            | render 函数 / `h()`       |
| -------- | --------------------------- | ------------------------- |
| 可读性   | 更直观，类 HTML             | 纯 JS，灵活但冷门         |
| 编译优化 | 可享受 PatchFlag/Block Tree | 无编译优化，跑时全量 diff |
| 动态能力 | 受限于模板语法              | 完全 JS 表达能力          |
| TS 支持  | 一般                        | 更好（纯 TS 上下文）      |
| JSX      | 不支持                      | 可用 JSX/TSX              |

**适合用 render 函数的场景：**

1. **高度动态的组件**：根据 props 动态决定渲染哪个标签 / 组件（如通用 Button 组件根据 `tag` prop 渲染 a / button / router-link）
2. **组件库开发**：封装底层组件（如 Element Plus 的表单 renderer）
3. **递归组件**：树形结构渲染，模板递归不直观
4. **程序化生成 VNode**：根据配置 JSON 动态生成表单/页面
5. **函数式组件**：简单包装组件，无状态
6. **插槽处理复杂**：需要对 slots 做过滤、包装、克隆等操作

**h() 函数参数：** `h(type, props?, children?)` —— type 可以是字符串标签、组件对象、Fragment/Teleport/Suspense；children 可以是字符串、数组、插槽函数对象。

**与模板的配合使用：**

- 大部分业务组件用模板（享受编译优化）
- 底层/通用组件用 render（享受灵活性）
- 可在模板组件中定义局部 render 组件（通过 `defineComponent` + render）

**坑点：**

- render 函数无法享受模板编译优化（PatchFlag/静态提升），性能略低
- JSX 需额外配置插件（`@vitejs/plugin-vue-jsx`）
- `h()` 的事件写法是 `onClick`（camelCase），不是 `@click`
- render 函数中不能使用 `v-model`、`v-show` 等指令，需手动实现

**面试加分点：** 能说清「模板最终也被编译为 render 函数，两者本质相同」；能解释「业务组件优先模板，底层组件优先 render」的选择逻辑；知道 Vue 3 的 `createRenderer` 可创建自定义渲染器（如渲染到 Canvas/终端）。

</details>

---

## 使用说明

- 在 Cursor / VSCode / Qoder 等 Markdown 预览中，点击 **「查看答案」** 即可展开。
- 每题答案均包含：**核心要点 → 对比/原理 → 坑点 → 面试加分点**（代码已大幅精简，以口述为主）
- 若预览环境不支持 `<details>`，请使用支持 HTML 的 Markdown 阅读器查看。
