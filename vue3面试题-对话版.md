# Vue 3 面试对话版（一问一答 · 点击展开答案）

共 **30 题**，模拟真实面试场景。点击「查看回答」展开答案。

---

## 一、基础与核心概念

**面试官：Vue 3 相比 Vue 2 有哪些主要变化？**

<details>
<summary>查看回答</summary>

> 六个方面：
>
> 1. **Composition API**：新增 `setup`、`ref`、`reactive` 等组合式 API，解决 Options API 在大型组件中逻辑碎片化问题，逻辑可按功能聚合、跨组件复用（Composables 替代 mixin）
> 2. **响应式系统重写**：`Object.defineProperty` → `Proxy`，解决新增/删除属性检测不到、数组索引不响应等 Vue 2 三大痛点，且懒代理性能更好
> 3. **编译优化**：静态提升（hoistStatic）、PatchFlag 标记动态内容、Block Tree 拍平动态节点、Tree-shaking 具名导出（包体可减小 50%+）
> 4. **TypeScript 支持**：源码用 TS 重写，props/emits/ref 类型推断更自然
> 5. **新内置组件**：`<Teleport>`、`<Suspense>`、Fragment（多根节点）、多 v-model、`<script setup>`
> 6. **破坏性调整**：移除 `$on/$off/$once`、filter；`v-if` 优先级高于 `v-for`；`new Vue()` → `createApp()`
>
> **加分点**：编译优化（PatchFlag/Block Tree）是 Vue 3 性能提升的真正大头，比 Proxy 改造影响更大。

</details>

---

**面试官：`ref` 和 `reactive` 有什么区别？分别适合什么场景？**

<details>
<summary>查看回答</summary>

> | 维度      | `ref`                   | `reactive`                      |
> | --------- | ----------------------- | ------------------------------- |
> | 接收类型  | 任意（基本类型 + 对象） | 仅对象/数组/Map/Set             |
> | JS 中访问 | 必须 `.value`           | 直接 `obj.x`                    |
> | 重新赋值  | `count.value = 100` ✅  | `state = {...}` ❌（丢响应式）  |
> | 解构      | 解构出来还是 ref        | 解构后丢失响应式（需 `toRefs`） |
>
> **场景选择**：
>
> - 基本类型、整体替换的数组、DOM 引用、Composables 返回值 → `ref`
> - 一组关联状态、不需要整体替换的复杂对象 → `reactive`
>
> **底层关系**：`ref(obj)` 内部其实就是 `reactive(obj)` 包了一层 `.value`。
>
> **加分点**：官方推荐优先用 `ref`——统一心智模型、可整体替换、与 Composables 配合更好。

</details>

---

**面试官：为什么 `reactive` 解构会丢失响应式？如何解决？**

<details>
<summary>查看回答</summary>

> **根本原因**：`reactive` 返回的是 Proxy 对象，响应式追踪发生在访问 Proxy 属性时。解构基本类型时等价于取出普通值，与 Proxy 断开联系。
>
> **解决方案**：
>
> 1. `toRefs(state)` — 整体转换，得到 ref 集合
> 2. `toRef(state, 'count')` — 单个属性转换
> 3. 不解构，模板中直接 `state.count`
> 4. Composables 返回 `toRefs(state)`
> 5. Pinia 中用 `storeToRefs`
>
> **加分点**：`toRefs` 内部对每个 key 创建 `ObjectRefImpl`，其 `.value` 的 getter/setter 直接读写原对象属性，本质是用 ref 壳包住对原 reactive 的引用。

</details>

---

**面试官：`computed` 和 `watch` / `watchEffect` 的区别？**

<details>
<summary>查看回答</summary>

> | 维度     | computed          | watch              | watchEffect           |
> | -------- | ----------------- | ------------------ | --------------------- |
> | 用途     | 派生值            | 监听变化执行副作用 | 副作用 + 自动收集依赖 |
> | 缓存     | ✅ 依赖不变不重算 | ❌                 | ❌                    |
> | 依赖收集 | 自动              | 显式指定源         | 自动                  |
> | 执行时机 | 惰性读取          | 默认惰性           | 立即执行              |
> | 旧值     | ❌                | ✅                 | ❌                    |
>
> **使用原则**：
>
> - 由 a、b 算出 c 给模板用 → `computed`（纯函数）
> - 需要监听特定源、拿新旧值、发请求 → `watch`
> - 副作用依赖多个值懒得列 → `watchEffect`
>
> **注意**：`computed` 里不要写副作用；`watchEffect` 中 await 之后的访问不会被追踪。

</details>

---

**面试官：Vue 3 的生命周期钩子有哪些？与 Vue 2 如何对应？**

<details>
<summary>查看回答</summary>

> **关键对应**：
>
> - `beforeCreate / created` → `setup()` 中同步代码即等价
> - `beforeMount / mounted` → `onBeforeMount / onMounted`
> - `beforeUpdate / updated` → `onBeforeUpdate / onUpdated`
> - `beforeDestroy / destroyed` → `onBeforeUnmount / onUnmounted`（改名）
> - `activated / deactivated` → `onActivated / onDeactivated`
>
> **Vue 3 新增**：`onRenderTracked`、`onRenderTriggered`（调试用）、`onServerPrefetch`（SSR）
>
> **要点**：
>
> - Composition 钩子必须在 setup 同步上下文中注册
> - 父子组件挂载顺序：父 beforeMount → 子 beforeMount → 子 mounted → 父 mounted
>
> **加分点**：setup 中没有 beforeCreate/created 是因为 setup 本身就在这两个钩子之间执行。

</details>

---

## 二、Composition API 与 `<script setup>`

**面试官：`<script setup>` 相比普通 `setup()` 有什么优势？**

<details>
<summary>查看回答</summary>

> 1. **更简洁**：顶层变量自动暴露给模板，不用 return
> 2. **编译宏**：`defineProps`/`defineEmits`/`defineExpose`/`defineOptions`/`defineModel` 无需 import
> 3. **TS 体验好**：`defineProps<Props>()` 纯类型声明
> 4. **默认私有**：内部状态父组件不可见，需 `defineExpose` 显式暴露
> 5. **性能更好**：编译期静态分析更多
> 6. **顶层 await**：可直接 `await`，配合 `<Suspense>`
>
> **加分点**：自动暴露是编译期实现的；默认私有是为了组件封装性。

</details>

---

**面试官：`defineProps` 和 `defineEmits` 有什么需要注意的点？**

<details>
<summary>查看回答</summary>

> 1. **编译宏无需 import**，不能解构再传参
> 2. **两种声明**：运行时声明（带校验/默认值）或纯类型声明 `defineProps<Props>()`
> 3. **默认值**：3.3+ 用 `withDefaults()`，引用类型用工厂函数
> 4. **props 只读**：想修改要 emit 通知父组件
> 5. **解构 props 会丢响应式**（3.5 之前）：用 `toRefs`/`toRef`；3.5+ 编译期支持响应式解构
> 6. **defineModel（3.4+）**：返回 ref，读写自动 emit，一行实现双向绑定
>
> **坑点**：props 默认值引用类型必须工厂函数；类型声明形式 3.2 前不支持外部 import 复杂类型。

</details>

---

**面试官：如何在 Vue 3 中实现逻辑复用？Composables 是什么？**

<details>
<summary>查看回答</summary>

> Composables 是利用 Composition API 把有状态逻辑封装成 `useXxx` 函数的模式。
>
> **相比 Vue 2 复用方案**：
>
> - Mixin：来源不清晰、命名冲突、无类型推断
> - Composables：来源清晰 ✅、无冲突 ✅、TS 推断好 ✅、可 Tree-shaking ✅
>
> **最佳实践**：
>
> - 以 `use` 开头命名
> - 返回 ref 集合，便于调用方解构
> - 内部副作用在 `onUnmounted` 清理
> - 异步逻辑返回响应式状态（data/error/loading）
>
> **加分点**：Composable 和普通工具函数的区别在于有状态、内部调用响应式 API；必须在 setup 同步上下文中调用。VueUse 提供 200+ 现成 Composables。

</details>

---

**面试官：`shallowRef` 和 `shallowReactive` 是什么？什么时候用？**

<details>
<summary>查看回答</summary>

> **浅层响应式，只追踪第一层**：
>
> - `shallowRef`：仅 `.value` 整体替换才触发更新
> - `shallowReactive`：仅第一层属性变化触发更新
>
> **使用场景**：
>
> 1. 大型不可变数据（表格几千行，整体替换）
> 2. 第三方实例（ECharts、地图 SDK），避免 Proxy 代理
> 3. 明确不需要深层响应式的性能优化
>
> **相关 API**：`markRaw`（永久不代理）、`toRaw`（拿原始对象）、`triggerRef`（强制触发）
>
> **加分点**：`reactive` 的递归代理是懒的（访问时才代理）；`shallowRef`/`shallowReactive` 是永远不代理深层。

</details>

---

## 三、模板、指令与组件通信

**面试官：Vue 3 中 `v-model` 有什么变化？**

<details>
<summary>查看回答</summary>

> |              | Vue 2                 | Vue 3               |
> | ------------ | --------------------- | ------------------- |
> | 默认 prop    | `value`               | `modelValue`        |
> | 默认事件     | `input`               | `update:modelValue` |
> | 多个 v-model | ❌（用 .sync）        | ✅ `v-model:title`  |
> | 修饰符       | 内置 lazy/number/trim | 支持自定义修饰符    |
>
> **defineModel（3.4+ 推荐）**：`const model = defineModel()` 返回 ref，读写 `.value` 自动触发 emit，代码最简。
>
> **加分点**：Vue 3 去掉了 `.sync` 合并到 v-model；defineModel 大幅简化了双向绑定代码量。

</details>

---

**面试官：`provide` / `inject` 的使用场景和注意事项？**

<details>
<summary>查看回答</summary>

> **作用**：跨任意层级组件传数据，避免 prop drilling。
>
> **典型场景**：主题切换、国际化、表单上下文、全局用户信息。
>
> **注意点**：
>
> 1. 必须 provide ref/reactive 才有响应式，普通值不响应
> 2. 用 `readonly()` 保护数据，改值走专用方法
> 3. 用 Symbol key 避免命名冲突，TS 自动推断
> 4. 可封装成 Composable 隐藏 provide/inject 细节
>
> **何时不用**：跨页面/全局 → Pinia；父子 → props/emit；兄弟 → 提升父级或 Pinia。
>
> **加分点**：provide/inject 基于组件树查找；建议 readonly + Symbol key。

</details>

---

**面试官：`Teleport` 是做什么的？常见使用场景？**

<details>
<summary>查看回答</summary>

> **作用**：把 DOM 传送到组件树外部指定节点渲染，但逻辑上仍属于原组件（状态、事件、provide/inject 正常）。
>
> **为什么需要**：弹窗、Toast 等浮层常被父级 `overflow:hidden`、`transform`、z-index 影响，挂到 body 下最干净。
>
> **用法**：`<Teleport to="body">...</Teleport>`，`to` 支持 CSS 选择器；可用 `:disabled` 条件传送。
>
> **典型场景**：Modal、Toast、Tooltip、Loading 遮罩、右键菜单。
>
> **加分点**：逻辑归属与 DOM 位置分离，类似 React Portal 但更声明式。

</details>

---

**面试官：Vue 3 支持多根节点（Fragment），有什么影响？**

<details>
<summary>查看回答</summary>

> Vue 3 模板支持多个并列根节点，编译器自动包成 Fragment（不渲染真实 DOM）。
>
> **影响**：
> | 方面 | 多根节点时 |
> |------|---|
> | `$attrs` 透传 | 不自动透传，需手动 `v-bind="$attrs"` |
> | `<Transition>` | 不支持多根，需包一层容器 |
> | `ref` 引用 | 拿组件实例代理，需给具体节点加 ref |
>
> **加分点**：Fragment 是编译期生成的虚拟节点，运行时其 children 直接铺开渲染；多根时需 `inheritAttrs: false` + `v-bind="$attrs"` 指定透传目标。

</details>

---

## 四、响应式原理（进阶）

**面试官：简要说明 Vue 3 响应式原理（Proxy 方案）**

<details>
<summary>查看回答</summary>

> **一句话**：用 `Proxy` 代理对象，`get` 时收集依赖、`set` 时触发更新，配合 `effect` 完成响应式调度。
>
> **三个核心**：
>
> - `reactive`：Proxy 代理，拦截 get/set/deleteProperty/ownKeys
> - `ref`：`{ value }` 包装 + getter/setter，内部同样走 track/trigger
> - `effect`：副作用函数，组件渲染、computed、watch 底层都是 effect
>
> **依赖收集结构**：`targetMap: WeakMap<target, Map<key, Set<effect>>>`
>
> **组件更新链路**：render effect 访问数据 → track 收集 → 数据变化 → trigger 触发 → 重新渲染
>
> **vs Vue 2**：解决新增/删除属性、数组索引、Map/Set 不支持三大问题；嵌套对象懒代理，初始化 O(1)。
>
> **加分点**：Proxy 不止拦截 get/set，还有 has/ownKeys/deleteProperty；组件的 render 函数本身就是一个 effect。

</details>

---

**面试官：`nextTick` 是做什么的？什么场景必须用？**

<details>
<summary>查看回答</summary>

> Vue 数据变化后不会立即更新 DOM，而是异步批量去重更新。`nextTick` 注册回调在 DOM 更新完成后执行。
>
> **典型场景**：
>
> 1. 改完数据立即操作 DOM（滚动到底部）
> 2. 显示元素后立即 focus
> 3. 基于真实 DOM 测量尺寸 / 初始化第三方库
>
> **实现原理**：基于 `Promise.resolve()` 微任务，保证在 Vue 更新之后执行。
>
> **加分点**：nextTick 比 setTimeout 更早执行（微任务 vs 宏任务）；Vue 异步更新是为了批量去重，避免重复 render。

</details>

---

## 五、Vue Router 4

**面试官：Vue Router 4 相对 Vue Router 3 有哪些重要变化？**

<details>
<summary>查看回答</summary>

> 1. **创建方式**：`new VueRouter()` → `createRouter({ history: createWebHistory() })`
> 2. **mode 变工厂函数**：`createWebHistory`/`createWebHashHistory`/`createMemoryHistory`
> 3. **`addRoutes` 移除**：改为 `addRoute` + `removeRoute`
> 4. **通配符**：`'*'` → `'/:pathMatch(.*)*'`
> 5. **导航守卫**：`next()` 可选，可直接 return false/路由对象
> 6. **Composition API**：`useRouter`/`useRoute`
> 7. **TS 支持**：可扩展 RouteMeta 类型
>
> **加分点**：mode 拆成工厂函数的好处是按需打包；`useRouter`/`useRoute` 必须在 setup 同步上下文调用。

</details>

---

**面试官：路由懒加载如何实现？有什么好处？**

<details>
<summary>查看回答</summary>

> **实现**：`component: () => import('@/views/Home.vue')`，打包工具自动代码分割，每个路由一个 chunk。
>
> **好处**：首屏 JS 体积小、不访问的页面不下载、单个路由更新只影响对应 chunk。
>
> **进阶**：`webpackPrefetch` 预加载下一个路由；配合 `<Suspense>` 做 loading 兜底；chunk 加载失败做重试。
>
> **加分点**：动态 import 路径要让打包工具能静态分析；「路由懒加载 + Suspense + Loading」是 Vue 3 常见最佳实践。

</details>

---

## 六、Pinia 与状态管理

**面试官：Pinia 相比 Vuex 有什么优势？为什么 Vue 3 推荐 Pinia？**

<details>
<summary>查看回答</summary>

> | 维度     | Vuex 4           | Pinia                      |
> | -------- | ---------------- | -------------------------- |
> | Mutation | 必须有           | 移除，直接改或 action 中改 |
> | Module   | 嵌套 + namespace | 独立 store，扁平化         |
> | TS 支持  | 需手写大量类型   | 自动推断                   |
> | 体积     | ~10kb            | ~1kb                       |
> | API 风格 | Options          | 类似 Composition API       |
>
> **核心优势**：无 mutation 样板代码；setup 风格 store；多 store 独立文件按需引入；store 间可互相引用。
>
> **加分点**：Pinia 取消 mutation 是因为 Vue 3 响应式 + DevTools 已能追踪变更；Vuex 进入维护模式。

</details>

---

**面试官：Pinia 中 `storeToRefs` 是干什么的？**

<details>
<summary>查看回答</summary>

> 解决 Pinia store 解构丢响应式的问题。
>
> Pinia store 本质是 reactive 对象，直接解构基本类型属性会丢失响应式。`storeToRefs` 把 state 和 getters 转成 ref，解构后仍保持响应式。
>
> **使用规则**：
>
> - state、getters → 用 `storeToRefs`
> - actions → 直接解构（函数不需要响应式包装）
>
> **vs `toRefs`**：`storeToRefs` 只转 state + getters，跳过 actions，专为 Pinia 设计。
>
> **加分点**：根本原因和 `reactive` 解构丢响应式一样——Proxy 属性在解构时被取值断开联系。

</details>

---

## 七、性能、构建与工程化

**面试官：Vue 3 有哪些常见性能优化手段？**

<details>
<summary>查看回答</summary>

> **Vue 3 编译/运行时优化**：
>
> 1. 静态提升：静态节点提到 render 外
> 2. PatchFlag：编译期标记动态内容，diff 只比动态部分
> 3. Block Tree：动态节点拍平，不递归整棵树
> 4. Tree-shaking：API 具名导出，未用代码剔除
>
> **开发实践优化**：
>
> - `v-show` vs `v-if`：频繁切换用 show
> - `v-for` 加稳定 key，不用 index
> - `v-memo`：大数据子树条件缓存
> - `shallowRef`/`markRaw`：大数组、第三方实例
> - 路由懒加载 + `defineAsyncComponent`
> - `<KeepAlive>` 缓存组件
> - 虚拟滚动、防抖/节流、图片懒加载
> - `v-once`、`Object.freeze`/`markRaw` 避免不必要响应式
>
> **加分点**：区分编译期优化（PatchFlag/Block Tree）和运行时/工程优化，根据场景给方案。

</details>

---

## 八、补充重点面试题

**面试官：Vue 3 的全局 API 有哪些变化？为什么用 createApp？**

<details>
<summary>查看回答</summary>

> **核心变化**：从全局 `Vue` 对象改为应用实例。`createApp(App)` 返回独立 `app`，插件/组件/指令都挂在实例上。
>
> **为什么改**：
>
> 1. 避免全局污染（多实例/微前端/测试互不干扰）
> 2. Tree-shaking 友好（不依赖全局 Vue.xxx）
> 3. 单元测试每个用例可创建独立 app
>
> **常见调整**：`Vue.component` → `app.component`；`Vue.use` → `app.use`；移除 `Vue.filter`；`Vue.set/delete` 不再需要。
>
> **加分点**：createApp 让每个应用拥有独立配置空间；移除 filter 是鼓励用 computed/method 替代。

</details>

---

**面试官：Vue 3 的插槽有哪些？作用域插槽怎么用？**

<details>
<summary>查看回答</summary>

> **三种插槽**：
>
> - 默认插槽：`<Child>内容</Child>` + `<slot></slot>`
> - 具名插槽：`<template #header>` + `<slot name="header">`
> - 作用域插槽：`<Child v-slot="{ user }">` + `<slot :user="user">`
>
> **作用域插槽本质**：子组件把数据作为 props 传给父组件的渲染函数，父组件通过 `v-slot="{ ... }"` 解构接收。
>
> **与 Vue 2 差异**：`slot-scope` → `v-slot` / `#` 语法糖。
>
> **加分点**：3.3+ 可用 `defineSlots()` 做 TS 类型声明；能区分 `v-slot` 在组件上和 `<template>` 上的用法。

</details>

---

**面试官：Vue 3 中如何封装自定义指令？**

<details>
<summary>查看回答</summary>

> Vue 3 指令钩子与组件生命周期对齐：`created` → `beforeMount` → `mounted` → `beforeUpdate` → `updated` → `beforeUnmount` → `unmounted`。
>
> **常用钩子**：
>
> - `mounted`：DOM 已插入，适合聚焦、权限判断、第三方初始化
> - `updated`：绑定值更新后执行
> - `unmounted`：清理副作用
>
> **钩子参数**：`el`（DOM）、`binding`（含 value/arg/modifiers）、`vnode`
>
> **典型场景**：`v-focus`、`v-permission`、`v-lazy`（图片懒加载）
>
> **加分点**：Vue 3 从 `bind/inserted/update/unbind` 改为与组件生命周期一致的命名。

</details>

---

**面试官：`<Suspense>` 和 `defineAsyncComponent` 怎么用？**

<details>
<summary>查看回答</summary>

> **Suspense**：在异步依赖解析期间显示 fallback 内容。`#default` 放异步组件，`#fallback` 放 Loading。
>
> **defineAsyncComponent**：
>
> - 最简：`defineAsyncComponent(() => import('./X.vue'))`
> - 完整配置：可指定 loadingComponent、errorComponent、delay、timeout
>
> **异步 setup**：`<script setup>` 中顶层 `await` 会隐式触发外层 Suspense。
>
> **与路由懒加载结合**：`<router-view v-slot>` + `<Suspense>` 实现切换 Loading 兜底。
>
> **加分点**：Suspense 目前只支持一个 default 和一个 fallback 插槽。

</details>

---

**面试官：`<KeepAlive>` 是什么？怎么控制缓存？**

<details>
<summary>查看回答</summary>

> **作用**：缓存动态组件实例，避免切换时反复创建/销毁。
>
> **控制缓存**：
>
> - `include`：只缓存匹配的组件名
> - `exclude`：不缓存匹配的
> - `max`：限制缓存数量，超出按 LRU 策略销毁
>
> **生命周期**：被缓存组件激活/失活触发 `onActivated`/`onDeactivated`。
>
> **注意**：子组件必须有 `name`（`<script setup>` 中用 `defineOptions({ name: 'Home' })`）；缓存的是组件实例不是 DOM。
>
> **加分点**：适合 tab 切换、表单填写一半切换等场景；`include`/`exclude` 匹配的是组件的 `name`。

</details>

---

## 九、高频进阶面试题

**面试官：Vue 3 的 Diff 算法原理？与 Vue 2 有什么区别？**

<details>
<summary>查看回答</summary>

> Vue 3 采用「快速 Diff」算法，配合编译期 PatchFlag 跳过静态节点。
>
> **五步流程**：
>
> 1. 同序列头部对比（左端相同直接 patch）
> 2. 同序列尾部对比（右端相同直接 patch）
> 3. 仅新增节点 → 挂载
> 4. 仅删除节点 → 卸载
> 5. 乱序部分 → 建 key→index 映射表，求最长递增子序列（LIS），属于 LIS 的节点不动，其余移动/新建/删除
>
> **vs Vue 2**：
> | 维度 | Vue 2 | Vue 3 |
> |------|---|---|
> | 算法 | 双端交叉对比 | 双端 + LIS |
> | 静态节点 | 全量 diff | PatchFlag 跳过 |
> | 树结构 | 递归整棵树 | Block Tree 拍平 |
> | 移动策略 | 移动次数可能多 | LIS 保证最少移动 |
>
> **加分点**：LIS 的作用是找出不需要移动的最大节点集合，保证 DOM 操作最少。编译期优化和运行时 Diff 是两个独立层级。

</details>

---

**面试官：Vue 3 组件通信方式有哪几种？各适用什么场景？**

<details>
<summary>查看回答</summary>

> | 方式             | 方向        | 适用场景               |
> | ---------------- | ----------- | ---------------------- |
> | props / emit     | 父↔子       | 父子常规数据流         |
> | v-model          | 双向        | 表单组件双向绑定       |
> | provide / inject | 祖先→后代   | 跨层级（主题/国际化）  |
> | Pinia            | 任意        | 全局/跨页面状态        |
> | $attrs           | 父→子透传   | 二次封装组件           |
> | expose / ref     | 父→子命令式 | 父组件调子组件方法     |
> | mitt             | 任意        | 兄弟/跨层级事件通知    |
> | 作用域插槽       | 子→父渲染   | 子向父暴露数据用于渲染 |
>
> **选择原则**：父子→props/emit → 跨层级→provide/inject → 全局→Pinia → 命令式→expose+ref
>
> **加分点**：Vue 3 移除 `$on/$listeners` 是为了让数据流更可追踪，避免隐式耦合。

</details>

---

**面试官：Vue 3.3–3.5 带来了哪些重要新特性？**

<details>
<summary>查看回答</summary>

> | 版本 | 特性                       | 说明                                  |
> | ---- | -------------------------- | ------------------------------------- |
> | 3.3  | `defineOptions`            | script setup 中声明 name/inheritAttrs |
> | 3.3  | `defineSlots`              | 插槽 TS 类型声明                      |
> | 3.3  | 泛型组件                   | `<script setup generic="T">`          |
> | 3.4  | `defineModel`              | 一行实现双向绑定                      |
> | 3.4  | `v-bind` 同名简写          | `:id` 等价于 `:id="id"`               |
> | 3.4  | `watch` 的 `once` 选项     | 只触发一次                            |
> | 3.5  | Reactive Props Destructure | 解构 props 仍保持响应式               |
> | 3.5  | `useTemplateRef`           | 显式获取模板 ref                      |
> | 3.5  | `useId`                    | 生成唯一 ID（SSR 一致）               |
> | 3.5  | `onWatcherCleanup`         | watch 副作用清理函数                  |
>
> **影响最大的三个**：defineModel（双向绑定从 5-8 行缩到 1 行）、Reactive Props Destructure（解决解构丢响应式老坑）、defineOptions（不再需要两个 script 块）。
>
> **加分点**：能跟进最新版本特性说明持续关注 Vue 生态。

</details>

---

**面试官：`effectScope` 是什么？解决什么问题？**

<details>
<summary>查看回答</summary>

> **一句话**：副作用容器，批量收集并统一销毁内部所有响应式副作用（computed/watch/watchEffect）。
>
> **解决的问题**：组件外创建的 computed/watch 不会自动清理，导致内存泄漏。effectScope 提供统一的生命周期边界。
>
> **核心 API**：
>
> - `effectScope()` 创建 scope
> - `scope.run(fn)` 在 scope 内执行，effect 被自动收集
> - `scope.stop()` 停止所有 effect
> - `onScopeDispose(fn)` 清理回调
>
> **使用场景**：Pinia store 内部、Composable 库开发、单元测试、非组件全局状态。
>
> **加分点**：Pinia 每个 store 就是一个 effectScope；组件内不需要手动用，因为组件本身就是一个 scope。

</details>

---

**面试官：什么场景下需要用 render 函数 / `h()` 而不是模板？**

<details>
<summary>查看回答</summary>

> **适合 render 函数的场景**：
>
> 1. 高度动态组件（根据 props 决定渲染哪个标签/组件）
> 2. 组件库开发（底层通用组件）
> 3. 递归组件（树形结构）
> 4. 程序化生成 VNode（配置 JSON 动态生成页面）
> 5. 插槽处理复杂（过滤/包装/克隆 slots）
>
> **模板 vs render**：
>
> - 模板：可读性好、享受编译优化（PatchFlag/静态提升）
> - render：灵活性强、完全 JS 表达能力、TS 支持更好
>
> **选择原则**：业务组件优先模板（享受编译优化）；底层/通用组件优先 render（享受灵活性）。
>
> **加分点**：模板最终也被编译为 render 函数，两者本质相同；`h()` 的事件写法是 `onClick`（camelCase）。

</details>

---

## 使用说明

- 点击 **「查看回答」** 即可展开答案
- 格式：面试官提问 → 候选人作答（含要点/对比/加分点）
- 适合面试前快速模拟练习
