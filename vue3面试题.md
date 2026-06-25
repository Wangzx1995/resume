# Vue 3 面试题（含隐藏答案 · 详细版）

共 **25 题**，涵盖基础、响应式、Composition API、组件通信、Router、Pinia、进阶特性与补充重点。点击「查看答案」即可展开，每题都包含**原理 / 对比 / 代码示例 / 坑点 / 面试加分点**。

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

```js
// ❌ 直接赋值会丢响应式
let state = reactive({ list: [] });
state = { list: [1, 2] }; // 丢失响应式！

// ✅ 改属性
state.list = [1, 2];

// ❌ 解构丢失响应式
const { count } = reactive({ count: 0 });

// ✅ 用 toRefs
const { count } = toRefs(reactive({ count: 0 }));
```

**面试加分点**：官方推荐**优先用 `ref`**——理由是统一心智模型（不用纠结）、可整体替换、与 Composables 配合更好。

</details>

---

### 3. 为什么 `reactive` 解构会丢失响应式？如何解决？

<details>
<summary>查看答案</summary>

**根本原因**：

`reactive` 返回的是一个 **Proxy 对象**。响应式追踪发生在「访问 Proxy 的属性时」（触发 `get` 拦截器收集依赖）。

```js
const state = reactive({ count: 0 });
// state 是 Proxy，state.count 访问时会 track

const { count } = state;
// 等价于 const count = state.count
// 这一步取出的就是普通的 number 0
// 之后访问 count 与 Proxy 完全无关 → 失去响应式
```

对**对象类型**的属性，解构出来的是引用，**只要不重新赋值还是响应式**；但**基本类型**值拷贝，立刻断掉。

```js
const state = reactive({
  count: 0, // 基本类型，解构后断
  user: { age: 18 }, // 对象，解构后 user.age 仍响应式（但 user 整体替换会断）
});
```

**解决方案：**

**方案 1：`toRefs` —— 整体转换**

```js
const state = reactive({ count: 0, name: "Vue" });
const { count, name } = toRefs(state);
// count、name 都是 ref，与原 state 双向绑定
count.value++; // state.count 也变成 1
```

**方案 2：`toRef` —— 单个属性**

```js
const state = reactive({ count: 0 });
const count = toRef(state, "count");
// Vue 3.3+ 还支持 toRef(() => state.count) 响应式 getter
```

**方案 3：不解构，直接访问**

```js
const state = reactive({ count: 0 });
// 模板里直接写 state.count
```

**方案 4：Composables 里返回 `toRefs(state)`**

```js
function useUser() {
  const state = reactive({ name: "", age: 0 });
  return toRefs(state); // 调用方可解构且保持响应式
}
const { name, age } = useUser();
```

**方案 5：Pinia 中用 `storeToRefs`**

```js
const store = useUserStore();
const { name, count } = storeToRefs(store); // state/getter 转 ref
const { login } = store; // action 直接解构
```

**面试加分点**：能说出「`toRefs` 内部就是遍历对象，对每个 key 调 `toRef`，`toRef` 创建一个 `ObjectRefImpl`，其 `.value` 的 getter/setter 直接读写原对象的属性」，本质是用 ref 的壳包住对原 reactive 对象的引用。

</details>

---

### 4. `computed` 和 `watch` / `watchEffect` 的区别？

<details>
<summary>查看答案</summary>

**对比总览：**

| 维度         | `computed`         | `watch`                        | `watchEffect`         |
| ------------ | ------------------ | ------------------------------ | --------------------- |
| 用途         | 派生**值**         | 监听变化执行**副作用**         | 副作用 + 自动收集依赖 |
| 是否有返回值 | 有（ComputedRef）  | 无                             | 无                    |
| 是否缓存     | ✅ 依赖不变不重算  | ❌                             | ❌                    |
| 依赖收集     | 自动               | 显式指定源                     | 自动                  |
| 执行时机     | 惰性（被读取才算） | 默认惰性，可 `immediate: true` | 立即执行一次          |
| 能拿旧值     | ❌                 | ✅ `(newVal, oldVal)`          | ❌                    |
| 异步操作     | 不推荐             | 推荐                           | 可以                  |

**1）computed —— 派生状态**

```js
const list = ref([1, 2, 3, 4]);
const evens = computed(() => list.value.filter((n) => n % 2 === 0));
// 多次访问 evens.value 只计算一次；list 变化才重新计算
```

支持 getter/setter：

```js
const fullName = computed({
  get: () => `${first.value} ${last.value}`,
  set: (val) => {
    [first.value, last.value] = val.split(" ");
  },
});
```

**2）watch —— 显式监听源**

```js
watch(count, (newVal, oldVal) => {
  /* ... */
});

// 监听多个源
watch([a, b], ([newA, newB], [oldA, oldB]) => {});

// 监听 reactive 的某个属性必须用 getter
watch(
  () => state.count,
  (n, o) => {},
);

// 深度监听
watch(state, () => {}, { deep: true });

// 立即执行
watch(source, cb, { immediate: true });

// 控制执行时机（DOM 更新后）
watch(source, cb, { flush: "post" });
```

**3）watchEffect —— 自动追踪**

```js
watchEffect(() => {
  // 函数内用到的所有 ref/reactive 属性都会被追踪
  console.log(count.value, state.name);
});
```

**关键差异**：

- `watchEffect` 立即执行一次（用于收集依赖），`watch` 默认懒执行
- `watchEffect` 不能拿旧值
- `watchEffect` 容易意外追踪你不想追踪的依赖（条件分支里的访问）

**何时选哪个？**

| 场景                           | 推荐          |
| ------------------------------ | ------------- |
| 由 a、b 算出 c 显示给模板      | `computed`    |
| a 变了发请求 / 改 localStorage | `watch`       |
| 需要新旧值对比                 | `watch`       |
| 一个副作用依赖好几个值，懒得列 | `watchEffect` |
| 同步副作用（写日志、调试）     | `watchEffect` |

**坑点：**

- `computed` 里**不要写副作用**（修改其他状态、发请求），它应是纯函数
- `watch` 监听 reactive 整个对象会自动 deep；监听 reactive 的子属性用 getter
- `watchEffect` 中的异步代码（await 之后的访问）**不会被追踪**

**面试加分点**：提到 `watchPostEffect`、`watchSyncEffect`（3.2+）的快捷方式，以及 `flush` 三种取值（`pre` 默认 / `post` DOM 后 / `sync` 同步）。

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

**使用示例：**

```vue
<script setup>
import { onMounted, onUnmounted } from "vue";

// setup 中同步代码 = beforeCreate + created
console.log("component creating");

onMounted(() => {
  console.log("DOM mounted");
});

onUnmounted(() => {
  console.log("cleanup");
});
</script>
```

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

**1）更简洁——顶层绑定自动暴露**

```vue
<!-- 普通 setup -->
<script>
import { ref } from "vue";
export default {
  setup() {
    const count = ref(0);
    const inc = () => count.value++;
    return { count, inc }; // 必须 return
  },
};
</script>

<!-- script setup -->
<script setup>
import { ref } from "vue";
const count = ref(0);
const inc = () => count.value++;
// 无需 return，自动暴露给模板
</script>
```

**2）编译宏（不需要 import，只在 `<script setup>` 中可用）**

| 宏                      | 作用                           |
| ----------------------- | ------------------------------ |
| `defineProps`           | 声明 props                     |
| `defineEmits`           | 声明 emits                     |
| `defineExpose`          | 暴露给父组件（默认是封闭的）   |
| `defineOptions`（3.3+） | 设置 name、inheritAttrs 等选项 |
| `defineSlots`（3.3+）   | 声明插槽类型                   |
| `defineModel`（3.4+）   | 简化 v-model 实现              |

**3）更好的 TS 体验**

```vue
<script setup lang="ts">
// 纯类型声明，编译期推断
const props = defineProps<{
  title: string;
  count?: number;
}>();

const emit = defineEmits<{
  change: [id: number, name: string];
  submit: [];
}>();
</script>
```

**4）默认私有**：内部状态对父组件不可见，需 `defineExpose({ ... })` 显式暴露。

**5）性能更好**：编译期静态分析更多，能识别哪些变量是响应式、哪些是常量，减少运行时开销。

**6）顶层 `await` 支持**

```vue
<script setup>
const data = await fetch("/api").then((r) => r.json());
// 需要外层用 <Suspense> 包裹
</script>
```

**与普通 `<script>` 共存**：

```vue
<script>
// 用于声明 name（3.3 之前）、自定义选项等
export default { name: "MyComponent", inheritAttrs: false };
</script>

<script setup>
// 主体逻辑
</script>
```

3.3+ 后可用 `defineOptions({ name: 'MyComponent' })` 替代上面的写法。

**面试加分点**：知道「自动暴露给模板」是编译期实现的（编译器把模板中的标识符与 setup 顶层变量对应）；知道默认私有是为了组件封装性，避免父组件随意访问。

</details>

---

### 7. `defineProps` 和 `defineEmits` 有什么需要注意的点？

<details>
<summary>查看答案</summary>

**1）编译宏，无需 import**

```vue
<script setup>
const props = defineProps(["title", "count"]);
const emit = defineEmits(["change", "submit"]);
</script>
```

**2）两种声明方式：运行时 vs 类型**

```ts
// 运行时声明（带校验、默认值）
const props = defineProps({
  title: { type: String, required: true },
  count: { type: Number, default: 0 },
});

// 纯类型声明（更简洁，TS 友好）
interface Props {
  title: string;
  count?: number;
}
const props = defineProps<Props>();
```

**3）类型声明默认值（3.3+ 直接支持）：**

```ts
const props = withDefaults(defineProps<Props>(), {
  count: 0,
  list: () => [], // 引用类型必须用工厂函数
});

// 3.3+ 可直接在解构里写
const { count = 0, list = [] } = defineProps<Props>();
```

**4）props 是只读的**

```js
const props = defineProps(["count"]);
props.count++; // ❌ 直接报警告

// ✅ 想要修改：emit 通知父组件
emit("update:count", props.count + 1);

// ✅ 想要本地副本：用 ref + watch 同步，或 computed
const localCount = ref(props.count);
watch(
  () => props.count,
  (v) => (localCount.value = v),
);
```

**5）defineEmits 的两种类型写法**

```ts
// 旧写法（数组形式）
const emit = defineEmits(["change", "submit"]);

// 对象式（带校验）
const emit = defineEmits({
  change: (id: number) => typeof id === "number",
});

// 类型声明（推荐 3.3+ 元组语法）
const emit = defineEmits<{
  change: [id: number, name: string];
  submit: [];
}>();
```

**6）解构 props 会丢失响应式（3.5 之前）**

```js
// ❌ 3.5 之前
const { count } = defineProps(['count']) // count 不再响应式

// ✅ 用 toRefs / toRef
const props = defineProps(['count'])
const { count } = toRefs(props)

// ✅ Vue 3.5+ 默认支持解构响应式（编译期处理）
const { count = 0 } = defineProps<{ count?: number }>()
```

**7）`defineModel`（3.4+）简化 v-model**

```vue
<script setup>
const model = defineModel(); // 自动声明 modelValue + update:modelValue
const title = defineModel("title"); // 具名 v-model:title
</script>

<template>
  <input v-model="model" />
</template>
```

**坑点汇总：**

- defineProps/Emits 是宏，**不能解构再传参**：`const { defineProps } = ...` ❌
- 类型声明形式不支持外部 import 的复杂类型（3.2 中受限，3.3+ 已大幅改善）
- props 默认值若是引用类型必须用工厂函数（避免共享引用）

</details>

---

### 8. 如何在 Vue 3 中实现逻辑复用？Composables 是什么？

<details>
<summary>查看答案</summary>

**定义**：Composables（组合式函数）是利用 Composition API 把**有状态的逻辑**封装成可复用函数的模式，约定以 `use` 开头命名。

**对比 Vue 2 复用方案：**

| 方案            | 来源清晰    | 命名冲突    | 类型推断 | Tree-shaking |
| --------------- | ----------- | ----------- | -------- | ------------ |
| Mixin           | ❌ 隐式合并 | ❌ 容易     | ❌ 差    | ❌           |
| 高阶组件 HOC    | 一般        | 一般        | 一般     | 一般         |
| Renderless 组件 | ✅          | ✅          | 一般     | 一般         |
| **Composables** | ✅ 显式调用 | ✅ 自己起名 | ✅ 完美  | ✅           |

**典型示例：**

```ts
// useCounter.ts
import { ref, computed } from "vue";

export function useCounter(initial = 0) {
  const count = ref(initial);
  const double = computed(() => count.value * 2);
  const inc = () => count.value++;
  const dec = () => count.value--;
  const reset = () => (count.value = initial);

  return { count, double, inc, dec, reset };
}
```

```vue
<script setup>
const { count, double, inc } = useCounter(10);
</script>
```

**带生命周期的例子（鼠标位置）：**

```ts
import { ref, onMounted, onUnmounted } from "vue";

export function useMouse() {
  const x = ref(0);
  const y = ref(0);

  const update = (e: MouseEvent) => {
    x.value = e.pageX;
    y.value = e.pageY;
  };

  onMounted(() => window.addEventListener("mousemove", update));
  onUnmounted(() => window.removeEventListener("mousemove", update));

  return { x, y };
}
```

**异步数据获取：**

```ts
export function useFetch<T>(url: string) {
  const data = ref<T | null>(null);
  const error = ref<Error | null>(null);
  const loading = ref(true);

  fetch(url)
    .then((r) => r.json())
    .then((d) => (data.value = d))
    .catch((e) => (error.value = e))
    .finally(() => (loading.value = false));

  return { data, error, loading };
}
```

**最佳实践：**

1. 命名以 `use` 开头（约定）
2. 输入参数可以是普通值或 ref（用 `unref` / `toValue` 统一处理）
3. 返回 ref 或 reactive（推荐返回 ref 集合，便于解构）
4. 在 Composable 内部注册的副作用要在 `onUnmounted` 清理
5. 异步逻辑应返回响应式状态，而不是直接 await（让组件能即时拿到 loading 等状态）

**生态库**：[VueUse](https://vueuse.org)（200+ 现成 Composables，如 `useLocalStorage`、`useDebounce`、`useIntersectionObserver`）。

**面试加分点**：能区分 Composable（组合式函数，有状态）和普通工具函数（无状态、不调用响应式 API）；能说出「Composables 必须在 setup 同步上下文中调用」（因为内部可能注册生命周期钩子）。

</details>

---

### 9. `shallowRef` 和 `shallowReactive` 是什么？什么时候用？

<details>
<summary>查看答案</summary>

**核心区别：浅层响应式（只追踪第一层）**

| API               | 触发更新的条件                           |
| ----------------- | ---------------------------------------- |
| `ref`             | `.value` 整体替换 + 内部对象任意层级变化 |
| `shallowRef`      | **仅** `.value` 整体替换                 |
| `reactive`        | 对象任意层级属性变化                     |
| `shallowReactive` | **仅**第一层属性变化                     |

**示例对比：**

```js
const deep = ref({ a: { b: 1 } });
deep.value.a.b = 2; // ✅ 触发更新

const shallow = shallowRef({ a: { b: 1 } });
shallow.value.a.b = 2; // ❌ 不触发
shallow.value = { a: { b: 2 } }; // ✅ 整体替换才触发

const r1 = reactive({ a: { b: 1 } });
r1.a.b = 2; // ✅ 触发

const r2 = shallowReactive({ a: { b: 1 } });
r2.a.b = 2; // ❌ 不触发
r2.a = { b: 2 }; // ✅ 第一层变才触发
```

**使用场景：**

**1）大型不可变数据**：列表数据量大且整体替换（如表格几千行）

```js
const tableData = shallowRef([]);
// 拉取后整体替换，避免对每行/每列做深度代理
fetchList().then((list) => (tableData.value = list));
```

**2）第三方实例**：ECharts、Three.js、地图 SDK，本身有内部状态、不该被代理

```js
const chart = shallowRef(null);
onMounted(() => {
  chart.value = echarts.init(el.value); // 不会代理 echarts 实例内部
});
```

更彻底的做法是用 `markRaw`：

```js
chart.value = markRaw(echarts.init(el.value)); // 永久标记不可代理
```

**3）性能优化**：明确知道不需要深层响应式时

**4）配合 `triggerRef` 手动触发**

```js
const data = shallowRef({ list: [] });
data.value.list.push(1); // 不会触发更新
triggerRef(data); // 手动触发
```

**相关 API 一览：**

| API               | 用途                     |
| ----------------- | ------------------------ |
| `shallowRef`      | 浅层 ref                 |
| `shallowReactive` | 浅层 reactive            |
| `shallowReadonly` | 浅层只读                 |
| `markRaw(obj)`    | 标记永远不被代理         |
| `toRaw(proxy)`    | 拿原始对象               |
| `triggerRef(ref)` | 强制触发 shallowRef 更新 |
| `customRef`       | 自定义 ref（节流防抖等） |

**坑点**：

- `shallowReactive` 的嵌套对象**完全不是响应式**（不像 reactive 会递归）
- `markRaw` 不可逆，加了就没法恢复
- 用 `shallowRef` 替换大对象比 `ref` 性能好很多（避免递归代理整个树）

**面试加分点**：能说出「reactive 的递归代理是懒的（访问到子属性时才代理子对象）」，所以并非一开始就有大成本；shallowRef/Reactive 是更进一步的「永远不代理深层」。

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
| 多个 v-model | ❌（要用 .sync）      | ✅ `v-model:title`  |
| 修饰符       | 内置 lazy/number/trim | 支持自定义修饰符    |

**1）基础用法（Vue 3）：**

父组件：

```vue
<MyInput v-model="text" />
<!-- 等价于 -->
<MyInput :modelValue="text" @update:modelValue="text = $event" />
```

子组件：

```vue
<script setup>
defineProps(["modelValue"]);
defineEmits(["update:modelValue"]);
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

**2）多个 v-model：**

```vue
<!-- 父 -->
<UserForm v-model:name="name" v-model:age="age" />
```

```vue
<!-- 子 -->
<script setup>
defineProps(["name", "age"]);
defineEmits(["update:name", "update:age"]);
</script>
```

**3）自定义修饰符：**

```vue
<!-- 父 -->
<MyInput v-model.capitalize="text" />
```

```vue
<!-- 子（修饰符通过 modelModifiers prop 接收） -->
<script setup>
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) },
});
const emit = defineEmits(["update:modelValue"]);

function onInput(e) {
  let val = e.target.value;
  if (props.modelModifiers.capitalize) {
    val = val.charAt(0).toUpperCase() + val.slice(1);
  }
  emit("update:modelValue", val);
}
</script>

<template>
  <input :value="modelValue" @input="onInput" />
</template>
```

具名 v-model 的修饰符：`v-model:title.capitalize` → 子组件接收 `titleModifiers` prop。

**4）`defineModel`（Vue 3.4+ 推荐）：**

子组件直接：

```vue
<script setup>
const model = defineModel(); // 简化 modelValue + emit
const [title, titleMods] = defineModel("title", { default: "" });
</script>

<template>
  <input v-model="model" />
</template>
```

`defineModel` 返回的是一个 ref，赋值即触发 emit，可读可写。

**5）原生 input 的 `v-model` 实际：**

| 元素                      | prop    | event  |
| ------------------------- | ------- | ------ |
| `<input type="text">`     | value   | input  |
| `<input type="checkbox">` | checked | change |
| `<input type="radio">`    | checked | change |
| `<select>`                | value   | change |

**面试加分点**：知道 Vue 3 完全去掉了 `.sync` 修饰符（合并到 v-model）；提到 3.4 引入的 `defineModel` 大幅简化了双向绑定写法。

</details>

---

### 11. `provide` / `inject` 的使用场景和注意事项？

<details>
<summary>查看答案</summary>

**作用**：跨任意层级组件传递数据，避免 prop 一层一层透传（prop drilling）。

**典型场景：**

- 主题切换（dark/light）
- 国际化（locale）
- 表单库的字段 ↔ 表单上下文（如 ElementPlus 的 Form 与 FormItem）
- 用户登录信息、权限
- 弹窗树深处获取根容器实例

**基础用法：**

```vue
<!-- 父组件 -->
<script setup>
import { provide, ref } from "vue";

const theme = ref("dark");
provide("theme", theme); // 提供响应式数据
provide("changeTheme", (t) => (theme.value = t)); // 也可提供方法
</script>
```

```vue
<!-- 任意后代组件 -->
<script setup>
import { inject } from "vue";

const theme = inject("theme", "light"); // 第二参数：默认值
const changeTheme = inject("changeTheme");
</script>
```

**关键注意点：**

**1）默认不响应式 → 必须 provide ref/reactive**

```js
// ❌ 普通值不响应式
provide("count", 0);

// ✅ 提供 ref
const count = ref(0);
provide("count", count);
```

**2）默认值的三种形式**

```js
inject("theme"); // 没找到返回 undefined
inject("theme", "light"); // 默认值（普通值）
inject("factory", () => ({}), true); // 第三参数 true：默认值是工厂函数
```

**3）只读保护，防止子组件意外修改**

```js
import { readonly } from "vue";
provide("theme", readonly(theme)); // 子组件只能读，不能改
provide("changeTheme", (t) => (theme.value = t)); // 改值要走专用方法
```

**4）使用 Symbol 作为 key 避免冲突**

```js
// keys.ts
export const ThemeKey = Symbol() as InjectionKey<Ref<string>>

// 父
provide(ThemeKey, theme)

// 子
const theme = inject(ThemeKey) // 自动推断为 Ref<string>
```

**5）应用级 provide**

```js
// main.ts
const app = createApp(App);
app.provide("apiBase", "/api");
// 全局可注入，常用于插件
```

**6）封装成 Composable 更优雅**

```ts
// useTheme.ts
const ThemeKey: InjectionKey<{ theme: Ref<string>; set: (t: string) => void }> =
  Symbol();

export function provideTheme() {
  const theme = ref("light");
  const set = (t: string) => (theme.value = t);
  provide(ThemeKey, { theme, set });
}

export function useTheme() {
  const ctx = inject(ThemeKey);
  if (!ctx) throw new Error("useTheme must be used after provideTheme");
  return ctx;
}
```

**何时不要用 provide/inject？**

- 跨页面/全局共享 → 用 Pinia
- 父子之间 → 用 props/emit 即可
- 兄弟组件 → 提升到共同父级或用 Pinia
- 滥用会让数据来源模糊，组件难以独立测试

**面试加分点**：能讲清「provide/inject 的查找是基于组件树而非 DOM 树」（slot 内容的 inject 解析在外层模板的所有者组件中）；提到 readonly 保护和 Symbol key 的最佳实践。

</details>

---

### 12. `Teleport` 是做什么的？常见使用场景？

<details>
<summary>查看答案</summary>

**功能**：把组件模板中的一段 DOM **传送（teleport）** 到组件树外部的指定 DOM 节点中渲染，但**逻辑上仍属于原组件**（响应式状态、事件冒泡、provide/inject 都正常）。

**为什么需要它？**

弹窗、Toast、下拉菜单等浮层组件常遇到的问题：

- 父级有 `overflow: hidden` → 弹窗被裁剪
- 父级有 `transform`、`filter` → 影响 fixed 定位
- 父级 z-index 层级混乱 → 弹窗被其他元素覆盖

最干净的解决方案就是把弹窗 DOM 直接挂到 `<body>` 下。

**基本用法：**

```vue
<template>
  <button @click="open = true">打开弹窗</button>

  <Teleport to="body">
    <div v-if="open" class="modal">
      <h3>{{ title }}</h3>
      <button @click="open = false">关闭</button>
    </div>
  </Teleport>
</template>

<script setup>
import { ref } from "vue";
const open = ref(false);
const title = ref("Hello");
</script>
```

注意：`title` 仍然是当前组件的状态，弹窗里直接用 —— 这就是「逻辑上仍属于原组件」。

**`to` 接受的值：**

```vue
<Teleport to="body">       <!-- CSS 选择器 -->
<Teleport to="#modal-root"> <!-- id -->
<Teleport :to="targetEl">  <!-- DOM 元素 -->
```

**禁用 teleport（条件传送）：**

```vue
<Teleport to="body" :disabled="isMobile">
  <!-- 移动端不传送，桌面端传送到 body -->
</Teleport>
```

**多个 Teleport 到同一目标**：会按顺序追加，不会互相覆盖。

**配合 Transition：**

```vue
<Teleport to="body">
  <Transition name="fade">
    <div v-if="show" class="modal">...</div>
  </Transition>
</Teleport>
```

**典型场景：**

1. Modal / Dialog 弹窗
2. Toast / Notification
3. Tooltip / Popover
4. 全屏 Loading 遮罩
5. 右键菜单

**坑点：**

- 目标元素必须**已存在于 DOM**（在 Vue 挂载之前），否则报警告
- SSR 中要用 `<Teleport disabled>` 或 `defer`（3.5+）避免 hydration 错位

**面试加分点**：解释「逻辑归属」与「DOM 位置」分离的设计——它解决了 React 中也存在的 portal 问题，但比手动 portal 更声明式。

</details>

---

### 13. Vue 3 支持多根节点（Fragment），有什么影响？

<details>
<summary>查看答案</summary>

**Vue 2**：模板必须有**唯一根元素**，否则编译报错。需要的时候要包一层无意义的 `<div>`。

**Vue 3**：模板可以有多个并列根节点，多余的会被包装成内部的 **Fragment** 节点（不渲染真实 DOM）。

```vue
<!-- Vue 2 必须包一层 -->
<template>
  <div>
    <header>...</header>
    <main>...</main>
  </div>
</template>

<!-- Vue 3 可以直接 -->
<template>
  <header>...</header>
  <main>...</main>
</template>
```

**好处：**

- 减少无意义包裹元素，HTML 更简洁
- 表格、列表场景非常实用：

```vue
<!-- 复用一组 tr -->
<template>
  <tr>
    <td>Name</td>
    <td>{{ name }}</td>
  </tr>
  <tr>
    <td>Age</td>
    <td>{{ age }}</td>
  </tr>
</template>
```

**带来的影响：**

**1）`$attrs` 自动透传规则变化**

单根节点：非 props 属性自动落到根节点

```vue
<!-- 父 -->
<MyBtn class="big" data-id="1" @click="..." />

<!-- 子（单根） -->
<template>
  <button>click</button>
  <!-- class、data-id、@click 自动落到 button -->
</template>
```

多根节点：Vue **不知道**该落到哪个节点，**不会自动透传**，需要手动绑定 `v-bind="$attrs"`：

```vue
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <!-- 手动指定 -->
</template>
```

否则父组件传的 class、监听器全丢失（开发环境会有警告）。

**2）`<Transition>` 仍需单根**

```vue
<!-- ❌ 多根节点不会被 Transition 包裹 -->
<Transition>
  <header v-if="show" />
  <main v-if="show" />
</Transition>

<!-- ✅ 用 TransitionGroup -->
<TransitionGroup>
  <header key="h" v-if="show" />
  <main key="m" v-if="show" />
</TransitionGroup>
```

**3）`ref` 无法引用整个组件根 DOM**

单根：`templateRef` 直接拿到根 DOM；多根：拿到的是一个组件实例代理，访问 DOM 需要在内部具体节点上加 ref。

**4）`v-for` 中的 fragment 需要 key 提示**

由于没有真实节点容器，多根 fragment 在 v-for 里的 diff 性能略有影响，建议合理使用 key。

**面试加分点**：能讲清「Fragment 是编译期生成的虚拟节点（VNode），运行时其 children 直接铺开渲染，没有对应 DOM」；提到属性透传需 `inheritAttrs: false` + `v-bind="$attrs"` 的搭配用法。

</details>

---

## 四、响应式原理（进阶）

### 14. 简要说明 Vue 3 响应式原理（Proxy 方案）

<details>
<summary>查看答案</summary>

**核心模块**：`@vue/reactivity`（独立包，可脱离 Vue 单独使用）

**三大基石：`reactive` / `ref` / `effect`**

**1）数据结构：依赖图**

```
targetMap (WeakMap)
└── target (原对象)
    └── depsMap (Map)
        └── key (属性名)
            └── dep (Set<ReactiveEffect>)
```

每个 `(target, key)` 对应一个 `dep`（依赖集合），每个 `effect` 是一个会重新执行的副作用函数。

**2）`reactive` 实现要点**

```js
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      const result = Reflect.get(target, key, receiver);
      track(target, key); // 收集当前 effect 到 (target, key) 的 dep
      // 嵌套对象懒代理（访问到时才包成 reactive）
      return isObject(result) ? reactive(result) : result;
    },
    set(target, key, value, receiver) {
      const oldValue = target[key];
      const result = Reflect.set(target, key, value, receiver);
      if (oldValue !== value) {
        trigger(target, key); // 触发该 dep 中所有 effect 重新执行
      }
      return result;
    },
    deleteProperty(target, key) {
      const had = hasOwn(target, key);
      const result = Reflect.deleteProperty(target, key);
      if (had) trigger(target, key);
      return result;
    },
    has(target, key) {
      /* track for 'in' operator */
    },
    ownKeys(target) {
      /* track for Object.keys / for...in */
    },
  });
}
```

**3）`effect` 副作用追踪**

```js
let activeEffect = null;

function effect(fn) {
  const e = () => {
    activeEffect = e;
    fn(); // fn 中访问的所有 reactive 属性都会调 track，把 e 收进对应 dep
    activeEffect = null;
  };
  e(); // 立即执行一次以收集依赖
}

function track(target, key) {
  if (!activeEffect) return;
  // 把 activeEffect 加入 targetMap[target][key] 对应的 dep
}

function trigger(target, key) {
  // 取出 dep 中所有 effect，调度执行
}
```

`computed` 内部就是一个特殊的 lazy effect；`watch` 是带调度的 effect；组件渲染本身也是一个 effect（render effect）。

**4）`ref` 实现要点**

```js
class RefImpl {
  constructor(value) {
    this._value = isObject(value) ? reactive(value) : value;
    this.dep = new Set();
  }
  get value() {
    track(this, "value");
    return this._value;
  }
  set value(newVal) {
    if (newVal !== this._value) {
      this._value = isObject(newVal) ? reactive(newVal) : newVal;
      trigger(this, "value");
    }
  }
}
```

**5）相比 Vue 2 (`Object.defineProperty`) 的优势**

| 能力            | Vue 2               | Vue 3                            |
| --------------- | ------------------- | -------------------------------- |
| 监听新增属性    | ❌（需 Vue.set）    | ✅                               |
| 监听删除属性    | ❌（需 Vue.delete） | ✅（deleteProperty）             |
| 数组索引/length | ❌（重写 7 个方法） | ✅                               |
| Map / Set       | ❌                  | ✅（专门的 collection handlers） |
| 嵌套对象代理    | 初始化递归（贵）    | 懒代理（访问才代理）             |
| 性能            | O(n) 初始化         | O(1) 初始化                      |

**6）局限**

- Proxy 不支持 IE11（Vue 3 也不支持 IE11）
- 解构基本类型属性会丢响应式（前面第 3 题）
- 替换整个 reactive 引用会丢响应式

**面试加分点**：能画出 `WeakMap → Map → Set` 的依赖图结构；能区分 `track`（收集）/ `trigger`（触发）；能说出「组件渲染是一个 effect，render 函数访问的响应式数据自动被收集」。

</details>

---

### 15. `nextTick` 是做什么的？什么场景必须用？

<details>
<summary>查看答案</summary>

**Vue 的更新机制**：响应式数据变化后，**不会立即更新 DOM**，而是把组件 push 到一个**异步更新队列**中，在下一个微任务里**批量去重**地更新。

→ 同一 tick 内多次修改同一数据，只触发一次更新（性能优化）。

**`nextTick` 的作用**：注册一个回调，在「下一次 DOM 更新完成后」执行。

**典型场景：**

**1）改完数据立即操作 DOM**

```vue
<script setup>
import { ref, nextTick } from "vue";
const list = ref([]);
const containerRef = ref(null);

async function addItem() {
  list.value.push("new item");
  // ❌ 此时 DOM 还没更新，scrollHeight 还是旧值
  // containerRef.value.scrollTop = containerRef.value.scrollHeight

  await nextTick();
  // ✅ 现在 DOM 已更新
  containerRef.value.scrollTop = containerRef.value.scrollHeight;
}
</script>
```

**2）显示元素后立即聚焦**

```vue
<script setup>
const showInput = ref(false);
const inputRef = ref(null);

async function toggle() {
  showInput.value = true;
  await nextTick();
  inputRef.value.focus(); // 此时 input 才真正存在
}
</script>
```

**3）基于真实 DOM 测量尺寸 / 初始化第三方库**

```js
async function init() {
  data.value = newData;
  await nextTick();
  myChart.resize(); // ECharts 等需要 DOM 已更新
}
```

**4）测试中等待视图更新**

```js
import { nextTick } from "vue";
test("updates view", async () => {
  wrapper.vm.count++;
  await nextTick();
  expect(wrapper.text()).toContain("1");
});
```

**两种调用方式：**

```js
// 1. callback 形式
nextTick(() => {
  /* DOM 已更新 */
});

// 2. Promise 形式（推荐 + async/await）
await nextTick();
```

**实现原理（简化）：**

```js
const queue = new Set();
let isFlushing = false;

function queueJob(job) {
  queue.add(job);
  if (!isFlushing) {
    isFlushing = true;
    Promise.resolve().then(flushJobs); // 微任务批处理
  }
}

function flushJobs() {
  for (const job of queue) job();
  queue.clear();
  isFlushing = false;
}

function nextTick(fn) {
  return Promise.resolve().then(fn);
}
```

→ Vue 的更新和 nextTick 都基于 **`Promise.resolve()` 微任务**，保证 nextTick 回调在更新之后执行。

**与 watch flush 的关系：**

```js
watch(source, cb); // flush: 'pre'，DOM 更新前
watch(source, cb, { flush: "post" }); // DOM 更新后（等价于 cb 内部 await nextTick）
watch(source, cb, { flush: "sync" }); // 同步（不推荐，无批处理）
```

`watchEffect` 同理，也有 `watchPostEffect` / `watchSyncEffect` 快捷方式。

**面试加分点**：知道 nextTick 是基于微任务（Promise）而非宏任务（setTimeout），所以比 setTimeout 更早执行；能解释「为什么 Vue 异步更新」——批量去重避免重复 render，提升性能。

</details>

---

## 五、Vue Router 4

### 16. Vue Router 4 相对 Vue Router 3 有哪些重要变化？

<details>
<summary>查看答案</summary>

**主要变化清单：**

**1）只能用于 Vue 3**（与 Vue 2 完全不兼容）

**2）创建方式：函数式 API**

```js
// Vue Router 3
import VueRouter from "vue-router";
Vue.use(VueRouter);
const router = new VueRouter({
  mode: "history",
  routes,
});
new Vue({ router }).$mount("#app");

// Vue Router 4
import { createRouter, createWebHistory } from "vue-router";
const router = createRouter({
  history: createWebHistory(), // 用工厂函数代替 mode
  routes,
});
createApp(App).use(router).mount("#app");
```

**3）`mode` 选项移除，改用 history 工厂函数**

| Vue Router 3       | Vue Router 4             |
| ------------------ | ------------------------ | ---------- |
| `mode: 'history'`  | `createWebHistory()`     |
| `mode: 'hash'`     | `createWebHashHistory()` |
| `mode: 'abstract'` | `createMemoryHistory()`  | （SSR 用） |

**4）`addRoutes` 移除，使用 `addRoute`**

```js
// V3
router.addRoutes([{...}])

// V4
router.addRoute({...})
router.addRoute('parentName', {...}) // 添加嵌套路由
router.removeRoute('name')
router.hasRoute('name')
```

**5）通配符语法变化**

```js
// V3
{ path: '*', component: NotFound }

// V4: 必须用参数 + 正则
{ path: '/:pathMatch(.*)*', component: NotFound }
```

**6）导航守卫变化**

- `next()` 变为可选：可直接 `return false` / `return '/login'` / `return { name: 'login' }`
- 守卫返回值即可控制导航
- 支持抛出 Error 触发 router 错误处理器

```js
// V4 推荐写法
router.beforeEach((to, from) => {
  if (!isAuthenticated && to.meta.requiresAuth) {
    return { name: "login" }; // 等价于 next({ name: 'login' })
  }
  // 不返回或返回 true 表示放行
});
```

**7）`<router-view>` 支持作用域插槽**，方便组合 KeepAlive、Transition

```vue
<router-view v-slot="{ Component, route }">
  <Transition :name="route.meta.transition || 'fade'">
    <KeepAlive>
      <component :is="Component" :key="route.path" />
    </KeepAlive>
  </Transition>
</router-view>
```

**8）`<router-link>` 也支持作用域插槽（自定义渲染）**

```vue
<router-link to="/about" custom v-slot="{ navigate, href, isActive }">
  <button :class="{ active: isActive }" @click="navigate">{{ href }}</button>
</router-link>
```

**9）Composition API：`useRouter` / `useRoute`**

```vue
<script setup>
import { useRouter, useRoute } from "vue-router";
const router = useRouter();
const route = useRoute();

const goHome = () => router.push("/");
console.log(route.params.id);
</script>
```

**10）TypeScript 支持完善**：路由 meta、params 都可声明类型。

```ts
// 全局扩展 RouteMeta
declare module "vue-router" {
  interface RouteMeta {
    requiresAuth?: boolean;
    title?: string;
  }
}
```

**11）异步路由不再用 `() => import().then(m => Vue.extend(m.default))` 这种 hack**

直接 `component: () => import('./X.vue')` 即可。

**面试加分点**：能说出 V4 把 mode 拆成多个 history 工厂函数的好处——按需打包（hash 模式不会打包 history 实现，反之亦然）；提到 `useRouter` 必须在 setup 同步上下文调用。

</details>

---

### 17. 路由懒加载如何实现？有什么好处？

<details>
<summary>查看答案</summary>

**实现方式：动态 import**

```js
// router/index.ts
const routes = [
  {
    path: "/",
    component: () => import("@/views/Home.vue"), // 关键：箭头函数 + 动态 import
  },
  {
    path: "/about",
    component: () => import("@/views/About.vue"),
  },
];
```

打包工具（Webpack / Vite）识别到动态 `import()` 会**自动代码分割**，每个文件单独打包成一个 chunk（如 `about-a1b2c3.js`），首次访问该路由时才下载。

**好处：**

1. **首屏 bundle 减小**：首页只加载首页相关代码，其他路由按需加载
2. **首屏速度提升**：JS 体积小 → 解析、执行更快
3. **资源利用更合理**：用户没访问的页面永远不下载
4. **缓存粒度更细**：单个路由更新只需重下对应 chunk，其他缓存仍有效

**进阶用法：**

**1）命名 chunk（Webpack）**

```js
{
  path: '/user',
  component: () => import(
    /* webpackChunkName: "user" */ '@/views/User.vue'
  )
}
```

→ 打包出来的文件名带 `user` 前缀，便于调试和监控。

**2）多个路由打到同一个 chunk**

```js
() => import(/* webpackChunkName: "user" */ '@/views/UserList.vue')
() => import(/* webpackChunkName: "user" */ '@/views/UserDetail.vue')
// 上面两个会被打到 user.[hash].js
```

**3）Webpack 预加载/预获取**

```js
// 预加载（与父 chunk 并行下载，浏览器空闲时）
() => import(/* webpackPrefetch: true */ '@/views/About.vue')

// 预获取（更激进，立即下载）
() => import(/* webpackPreload: true */ '@/views/Critical.vue')
```

**4）路由分组（按业务模块）**

```js
const userRoutes = [
  { path: "/user/list", component: () => import("@/views/user/List.vue") },
  { path: "/user/detail", component: () => import("@/views/user/Detail.vue") },
];
```

**5）配合异步组件 + Suspense（Vue 3）**

```vue
<router-view v-slot="{ Component }">
  <Suspense>
    <template #default><component :is="Component" /></template>
    <template #fallback><Loading /></template>
  </Suspense>
</router-view>
```

**6）失败重试**

```js
function lazyLoad(importFn, retries = 3) {
  return () => importFn().catch(err => {
    if (retries > 0) return lazyLoad(importFn, retries - 1)()
    throw err
  })
}
{ path: '/x', component: lazyLoad(() => import('@/views/X.vue')) }
```

→ 处理网络抖动导致 chunk 加载失败的情况。

**结合 Vite：**

- Vite 在开发环境直接用浏览器原生 ESM，不打包，懒加载效果天然
- 生产环境用 Rollup 自动 code-splitting

**面试加分点**：知道动态 import 必须是字符串字面量（或带模板字符串的相对路径），不能完全运行时拼接，否则打包工具无法分析；提到「路由懒加载 + Suspense + Loading 组件」的最佳实践。

</details>

---

## 六、Pinia 与状态管理

### 18. Pinia 相比 Vuex 有什么优势？为什么 Vue 3 推荐 Pinia？

<details>
<summary>查看答案</summary>

**Pinia 是 Vue 官方推荐的新一代状态管理库，作者 Eduardo（同时也是 Vue Router 维护者）。**

**对比表：**

| 维度        | Vuex 4                  | Pinia                             |
| ----------- | ----------------------- | --------------------------------- |
| API 风格    | Options 风格 + 复杂概念 | 类 Composition API                |
| Mutation    | ✅ 必须                 | ❌ 移除（直接改或在 action 中改） |
| Module      | 嵌套 module + namespace | 多个独立 store，扁平化            |
| TS 支持     | 复杂（需大量手写类型）  | 完美（自动推断）                  |
| 体积        | ~10kb                   | ~1kb                              |
| DevTools    | ✅                      | ✅（time travel、state edit）     |
| SSR         | ✅                      | ✅                                |
| 异步 action | ✅ 必须用 actions       | ✅ 直接 async 函数                |

**Pinia 的核心优势：**

**1）API 极简**

```js
// pinia
import { defineStore } from "pinia";

export const useCounter = defineStore("counter", {
  state: () => ({ count: 0, name: "tom" }),
  getters: {
    double: (state) => state.count * 2,
  },
  actions: {
    increment() {
      this.count++;
    },
    async fetchUser() {
      const user = await api.get("/me");
      this.name = user.name;
    },
  },
});
```

```js
// vuex 同样功能
export default new Vuex.Store({
  state: () => ({ count: 0, name: "tom" }),
  getters: {
    double: (state) => state.count * 2,
  },
  mutations: {
    INCREMENT(state) {
      state.count++;
    },
    SET_NAME(state, name) {
      state.name = name;
    },
  },
  actions: {
    increment({ commit }) {
      commit("INCREMENT");
    },
    async fetchUser({ commit }) {
      const user = await api.get("/me");
      commit("SET_NAME", user.name);
    },
  },
});
```

→ 同样的功能，Pinia 没有 mutations 的样板代码。

**2）setup 风格 store（更符合 Composition API 心智）**

```js
import { defineStore } from "pinia";
import { ref, computed } from "vue";

export const useCounter = defineStore("counter", () => {
  // state
  const count = ref(0);
  // getters
  const double = computed(() => count.value * 2);
  // actions
  const increment = () => count.value++;

  return { count, double, increment };
});
```

可以使用任何 Composition API（包括 `watch`、`onMounted` 等）。

**3）多个 store 扁平化**

```js
// userStore.ts
export const useUser = defineStore('user', { ... })

// cartStore.ts
export const useCart = defineStore('cart', { ... })

// 组件中
const user = useUser()
const cart = useCart()
```

不需要 module 嵌套和 namespace 字符串。

**4）TS 支持完美**

state、getters、actions 全部自动推断，无需手写类型声明。

```ts
const store = useCounter();
store.count; // number
store.double; // number
store.increment(); // void
store.unknownProp; // 编译报错
```

**5）支持 Store 之间互相引用**

```js
export const useCart = defineStore("cart", () => {
  const userStore = useUser(); // 直接调用另一个 store
  const items = ref([]);
  return { items, userStore };
});
```

**6）插件机制**

```js
const pinia = createPinia();
pinia.use(({ store }) => {
  store.$onAction(({ name, args, after, onError }) => {
    console.log(`Action ${name} called`);
  });
});
```

可轻松实现持久化（pinia-plugin-persistedstate）、撤销重做等。

**7）热重载（HMR）开箱即用**

```js
if (import.meta.hot) {
  import.meta.hot.accept(acceptHMRUpdate(useCounter, import.meta.hot));
}
```

**面试加分点**：能说出「Pinia 取消 mutation」是因为 Vue 3 的响应式 + DevTools 已能追踪所有变更，mutation 这层抽象成了多余的样板代码；Pinia 是 Vue 官方推荐，Vuex 进入维护模式。

</details>

---

### 19. Pinia 中 `storeToRefs` 是干什么的？

<details>
<summary>查看答案</summary>

**问题**：Pinia 的 store 实例本身是一个 reactive 对象。直接解构会**丢失响应式**（与 reactive 解构丢响应式同理）。

```js
const store = useCounter();
const { count, name } = store; // ❌ 解构出来是普通值，不再响应式

count++; // 不会触发更新
```

**`storeToRefs` 解决这个问题**：把 state 和 getters 转成 ref，解构后仍保持响应式。

```js
import { storeToRefs } from "pinia";

const store = useCounter();
const { count, name, double } = storeToRefs(store); // ✅ ref，保持响应式
const { increment, fetchUser } = store; // ✅ action 直接解构，不需要 storeToRefs
```

**关键规则：**

- **state、getters → 用 `storeToRefs`**（响应式数据）
- **actions → 直接从 store 解构**（普通函数，不需要响应式包装）

**为什么 actions 不需要？**

actions 是普通方法，绑定到 store 实例（this 指向 store）。直接解构出来调用 `increment()` 仍能正常工作，只是 `this` 已经被 Pinia 内部用箭头函数或绑定处理好了。

**与 `toRefs` 的区别：**

|          | `toRefs`           | `storeToRefs`                                   |
| -------- | ------------------ | ----------------------------------------------- |
| 来源     | Vue 核心           | Pinia                                           |
| 转换内容 | 所有可枚举属性     | 仅 state + getters，跳过 actions 和非响应式属性 |
| 用途     | 任意 reactive 对象 | 专为 Pinia store 设计                           |

如果用 `toRefs(store)` 也能转 state，但会把 actions 也变成 ref（不需要的开销，且语义不对）。

**模板中的等价写法：**

模板里直接 `store.count`、`store.increment()` 也行，无需 `storeToRefs`（模板自动解包），但代码冗长。

**完整使用模式：**

```vue
<script setup>
import { storeToRefs } from "pinia";
import { useCounter } from "@/stores/counter";

const counterStore = useCounter();
const { count, double } = storeToRefs(counterStore);
const { increment, decrement } = counterStore;
</script>

<template>
  <div>
    <p>{{ count }} (double: {{ double }})</p>
    <button @click="increment">+</button>
    <button @click="decrement">-</button>
  </div>
</template>
```

**实现原理简化：**

```js
function storeToRefs(store) {
  const refs = {};
  for (const key in store) {
    const value = store[key];
    // state 是 ref（在 setup store 中）或 reactive 属性
    // getters 内部是 computed
    if (isRef(value) || isReactive(value)) {
      refs[key] = toRef(store, key);
    }
    // 跳过 action
  }
  return refs;
}
```

**面试加分点**：能解释「state 解构丢响应式」的根本原因（Pinia store 内部用 reactive 实现，与第 3 题同根同源），并知道 actions 因为是函数所以不受影响。

</details>

---

## 七、性能、构建与工程化

### 20. Vue 3 有哪些常见性能优化手段？

<details>
<summary>查看答案</summary>

性能优化分为**框架层面（编译/运行时）**和**开发实践**两个维度。

---

### 框架层面（Vue 3 自带，无需手动）

**1）静态提升（Static Hoisting）**

- 模板中的静态节点在编译期被提升到 render 函数外，每次 render 不重新创建

```js
// 编译前
<div>
  <p>static</p>
  <p>{{ dynamic }}</p>
</div>;

// 编译后（伪代码）
const _hoisted = _createVNode("p", null, "static"); // 提到外部
function render() {
  return _createVNode("div", null, [
    _hoisted,
    _createVNode("p", null, ctx.dynamic),
  ]);
}
```

**2）PatchFlag（补丁标记）**

- 编译期给动态节点打上 flag（如 `1` 表示 textContent 动态、`2` 表示 class 动态）
- 运行时 diff 只对比有 flag 的部分，跳过静态属性

**3）Block Tree**

- 把动态节点拍平到一个数组，diff 时只遍历这个数组，不再深度递归整棵树
- 大幅减少 diff 比较次数

**4）Tree-shaking**

- API 全部具名导出（`import { ref } from 'vue'`），未使用的可被剔除
- Vue 2 的 `Vue.xxx` 全局对象无法 tree-shake

**5）SSR 优化**

- 静态内容直接拼字符串，不走 createVNode

---

### 开发实践（手动优化）

**1）合理用 `v-show` vs `v-if`**

- 频繁切换 → `v-show`（CSS 切换）
- 条件渲染（不常变） → `v-if`（DOM 创建/销毁）

**2）`v-for` 必须加 `key`**

- 用稳定唯一 id（不要用 index）
- 帮助 diff 算法精准定位变化

**3）`v-memo`（3.2+）—— 条件性缓存子树**

```vue
<div v-memo="[item.id, item.selected]">
  <!-- 只有 id 或 selected 变化时才重新渲染 -->
  <ExpensiveItem :data="item" />
</div>
```

**4）`shallowRef` / `shallowReactive` / `markRaw`**

- 不需要深层响应式时使用，避免不必要的 Proxy 开销

```js
const bigList = shallowRef([]); // 大数组整体替换
const chartInstance = markRaw(echarts.init(el)); // 第三方实例不代理
```

**5）异步组件 + 路由懒加载**

```js
import { defineAsyncComponent } from "vue";

const Heavy = defineAsyncComponent({
  loader: () => import("./Heavy.vue"),
  loadingComponent: Loading,
  errorComponent: Error,
  delay: 200,
  timeout: 3000,
});
```

**6）`<KeepAlive>` 缓存组件实例**

```vue
<KeepAlive :include="['UserList', 'UserDetail']" :max="10">
  <component :is="currentTab" />
</KeepAlive>
```

避免 tab 切换时重新渲染、重新拉数据。

**7）合理使用 `computed` 缓存**

- 派生值优先用 computed（自动缓存），不要写在 methods 里被多次调用

**8）大列表虚拟滚动**

- 列表长度 > 1000 时，用 `vue-virtual-scroller`、`vue-virtual-list` 等
- 只渲染可视区 + 少量缓冲区

**9）合理粒度的组件拆分**

- 频繁变化的部分独立成子组件，缩小 re-render 范围
- Vue 3 的 PatchFlag 已减少这种需求，但仍是有效手段

**10）防抖 / 节流**

- 输入搜索、scroll 监听、resize 监听等
- 用 VueUse 的 `useDebouncedRef`、`useThrottleFn`

```js
import { useDebouncedRef } from "@vueuse/core";
const search = useDebouncedRef("", 300);
```

**11）Web Worker 处理重计算**

- 避免阻塞主线程

**12）图片懒加载**

```vue
<img v-lazy="src" />
<!-- 或原生 -->
<img :src="src" loading="lazy" />
```

**13）生产构建优化**

- 关闭 devtools：`app.config.devtools = false`（生产环境）
- 关闭 productionTip
- Source map 按需开启
- 使用 `defineAsyncComponent` 拆分大组件
- gzip / brotli 压缩
- CDN 加载常用库（Vue、Element Plus 等）

**14）合理使用 `v-once`** —— 永远只渲染一次

```vue
<header v-once>
  <h1>{{ siteName }}</h1>
</header>
```

**15）避免不必要的响应式**

- 配置常量、字典：用 `Object.freeze` 或 `markRaw`，不要 reactive

```js
const STATUS_MAP = Object.freeze({ 1: "激活", 2: "禁用" });
```

**16）Pinia 用 `storeToRefs` 时按需解构**

- 只取需要的字段，避免把整个 store 转成 refs

**17）路由级代码分割 + 预加载关键路由**

```js
{
  component: () => import(/* webpackPrefetch: true */ "@/views/Home.vue");
}
```

---

**性能监控工具：**

- Vue DevTools 的 Performance 面板
- Chrome Performance / Lighthouse
- `onRenderTracked` / `onRenderTriggered` 调试不必要的 re-render

**面试加分点**：能区分「Vue 3 编译期优化」（PatchFlag/Block Tree/静态提升）和「开发实践优化」（KeepAlive/虚拟滚动/markRaw），并能根据场景给出具体方案而非泛泛而谈。

</details>

---

## 八、补充重点面试题

### 21. Vue 3 的全局 API 有哪些变化？为什么用 createApp？

<details>
<summary>查看答案</summary>

**核心变化：从全局 Vue 对象改为应用实例**

```js
// Vue 2
import Vue from "vue";
import VueRouter from "vue-router";
import Vuex from "vuex";

Vue.use(VueRouter);
Vue.use(Vuex);
Vue.component("MyComp", MyComp);
Vue.directive("focus", focusDirective);
Vue.filter("capitalize", capitalize);

new Vue({
  router,
  store,
  render: (h) => h(App),
}).$mount("#app");
```

```js
// Vue 3
import { createApp } from "vue";
import { createRouter } from "vue-router";
import { createPinia } from "pinia";
import App from "./App.vue";

const app = createApp(App);

app.use(createRouter(...));
app.use(createPinia());
app.component("MyComp", MyComp);
app.directive("focus", focusDirective);
app.config.globalProperties.$api = api;

app.mount("#app");

// 可创建多个独立应用
const app2 = createApp(App2);
app2.mount("#app2");
```

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

**1）默认插槽**

```vue
<!-- 父 -->
<Child>默认内容</Child>

<!-- 子 -->
<template>
  <slot></slot>
</template>
```

**2）具名插槽**

```vue
<!-- 父 -->
<Child>
  <template #header>头部</template>
  <template #footer>底部</template>
</Child>

<!-- 子 -->
<template>
  <slot name="header"></slot>
  <slot></slot>
  <slot name="footer"></slot>
</template>
```

**3）作用域插槽**

子组件向父组件传递数据：

```vue
<!-- 子 -->
<template>
  <slot :user="user" :age="age"></slot>
</template>

<script setup>
const user = ref({ name: "Tom" });
const age = ref(20);
</script>
```

```vue
<!-- 父 -->
<Child v-slot="{ user, age }">
  {{ user.name }} - {{ age }}
</Child>
```

**`<script setup>` 中定义插槽**

```vue
<script setup>
const slots = defineSlots(); // Vue 3.3+
</script>
```

**与 Vue 2 的差异**

|              | Vue 2                 | Vue 3                |
| ------------ | --------------------- | -------------------- |
| 默认插槽     | `slot` / `slot-scope` | `v-slot` / `#`       |
| 作用域插槽   | `slot-scope`          | `v-slot="{ data }"`  |
| 动态插槽名   | `#[dynamicName]`      | 支持                 |
| 插槽内容编译 | 编译为函数            | 编译为函数，性能更好 |

**常见坑点**

```vue
<!-- ❌ 解构时命名冲突 -->
<Child v-slot="{ user }">
  <span v-for="user in list" :key="user.id">{{ user.name }}</span>
</Child>
```

**面试加分点**：能解释作用域插槽的本质是「子组件把数据作为 props 传给父组件的渲染函数」；能区分 `v-slot` 在组件上和 `<template>` 上的用法。

</details>

---

### 23. Vue 3 中如何封装自定义指令？

<details>
<summary>查看答案</summary>

**自定义指令钩子**

Vue 3 的指令钩子与组件生命周期对齐：

```js
const myDirective = {
  created(el, binding, vnode, prevVnode) {}, // 绑定前
  beforeMount(el, binding, vnode, prevVnode) {}, // 挂载前
  mounted(el, binding, vnode, prevVnode) {}, // 挂载后
  beforeUpdate(el, binding, vnode, prevVnode) {}, // 更新前
  updated(el, binding, vnode, prevVnode) {}, // 更新后
  beforeUnmount(el, binding, vnode, prevVnode) {}, // 卸载前
  unmounted(el, binding, vnode, prevVnode) {}, // 卸载后
};
```

**示例：v-focus**

```js
// directives/focus.js
export const vFocus = {
  mounted(el) {
    el.focus();
  },
};
```

```vue
<script setup>
import { vFocus } from "./directives/focus";
</script>

<template>
  <input v-focus />
</template>
```

**示例：v-permission（权限控制）**

```js
export const vPermission = {
  mounted(el, binding) {
    const required = binding.value; // 'admin'
    const userRole = useUserStore().role;
    if (!userRole.includes(required)) {
      el.remove(); // 无权限移除元素
    }
  },
};
```

**钩子参数**

| 参数      | 说明                                    |
| --------- | --------------------------------------- |
| el        | 指令绑定的 DOM                          |
| binding   | 包含 value、arg、modifiers、instance 等 |
| vnode     | 当前虚拟节点                            |
| prevVnode | 上一个虚拟节点                          |

```js
binding.value; // 指令值
binding.arg; // 参数 v-my:arg
binding.modifiers; // 修饰符 v-my.mod
binding.instance; // 组件实例
```

**全局注册**

```js
app.directive("focus", {
  mounted(el) {
    el.focus();
  },
});
```

**面试加分点**：能说出 Vue 3 指令钩子从 `bind/inserted/update/unbind` 改为与组件生命周期一致；能解释 `binding.value` 和 `binding.arg` 的区别。

</details>

---

### 24. `<Suspense>` 和 `defineAsyncComponent` 怎么用？

<details>
<summary>查看答案</summary>

**`<Suspense>` 作用**

`<Suspense>` 是 Vue 3 新增内置组件，用于在异步依赖（如异步组件、异步 setup）解析期间显示 fallback 内容。

```vue
<template>
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <Loading />
    </template>
  </Suspense>
</template>
```

**`defineAsyncComponent`**

Vue 3 推荐用 `defineAsyncComponent` 定义异步组件：

```js
import { defineAsyncComponent } from "vue";

const AsyncComp = defineAsyncComponent(() => import("./AsyncComp.vue"));
```

**带 loading/error 配置**

```js
const AsyncComp = defineAsyncComponent({
  loader: () => import("./AsyncComp.vue"),
  loadingComponent: Loading,
  errorComponent: Error,
  delay: 200, // 延迟显示 loading，避免闪烁
  timeout: 3000, // 超时时间
  onError(error, retry, fail, attempts) {
    if (attempts <= 3) retry();
    else fail();
  },
});
```

**异步 setup 与 Suspense**

```vue
<script setup>
const data = await fetch("/api").then((r) => r.json());
</script>
```

顶层 `await` 的 `<script setup>` 会隐式触发 Suspense。

**与路由懒加载结合**

```js
const routes = [
  {
    path: "/about",
    component: () => import("@/views/About.vue"),
  },
];
```

```vue
<router-view v-slot="{ Component }">
  <Suspense>
    <template #default>
      <component :is="Component" />
    </template>
    <template #fallback>
      <Loading />
    </template>
  </Suspense>
</router-view>
```

**面试加分点**：能说出 `<Suspense>` 目前只支持一个默认插槽和一个 fallback 插槽；能解释异步组件配合路由懒加载实现代码分割。

</details>

---

### 25. Vue 3 中的 `<KeepAlive>` 是什么？怎么控制缓存？

<details>
<summary>查看答案</summary>

**作用**

`<KeepAlive>` 是 Vue 内置组件，用于缓存动态组件的实例，避免组件切换时反复创建/销毁，保留组件状态。

```vue
<template>
  <KeepAlive>
    <component :is="currentTab" />
  </KeepAlive>
</template>
```

**控制缓存**

**1）`include` / `exclude`**

```vue
<KeepAlive include="Home,User">
  <component :is="currentTab" />
</KeepAlive>

<!-- 也可以用数组 -->
<KeepAlive :include="['Home', 'User']">
  <component :is="currentTab" />
</KeepAlive>

<KeepAlive exclude="Cache">
  <component :is="currentTab" />
</KeepAlive>
```

**2）`max` 限制缓存数量**

```vue
<KeepAlive :max="10">
  <component :is="currentTab" />
</KeepAlive>
```

LRU 策略：超出 max 时，最久未访问的组件会被销毁。

**3）配合路由缓存**

```vue
<router-view v-slot="{ Component }">
  <KeepAlive>
    <component :is="Component" />
  </KeepAlive>
</router-view>
```

**生命周期钩子**

被 KeepAlive 缓存的组件会触发 `onActivated` 和 `onDeactivated`：

```vue
<script setup>
import { onActivated, onDeactivated } from "vue";

onActivated(() => {
  console.log("组件被激活");
});

onDeactivated(() => {
  console.log("组件被缓存");
});
</script>
```

**注意点**

- KeepAlive 要求子组件有 `name` 选项（或文件名，在 script setup 中用 defineOptions）
- 缓存的是组件实例，不是 DOM
- 异步组件首次加载后也会被缓存

```vue
<script setup>
defineOptions({ name: "Home" });
</script>
```

**面试加分点**：能说出 KeepAlive 适合 tab 切换、表单填写一半切换等场景；能解释 `include`/`exclude` 匹配组件的 `name` 选项。

</details>

---

## 使用说明

- 在 Cursor / VSCode / Qoder 等 Markdown 预览中，点击 **「查看答案」** 即可展开。
- 每题答案均包含：**核心要点 → 代码示例 → 对比/原理 → 坑点 → 面试加分点**
- 若预览环境不支持 `<details>`，请使用支持 HTML 的 Markdown 阅读器查看。
