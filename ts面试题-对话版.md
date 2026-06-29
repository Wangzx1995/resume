# TypeScript 面试对话版（一问一答 · 点击展开答案）

共 **20 题**，模拟真实面试场景。点击「查看回答」展开答案。

---

## 一、基础类型与核心概念

**面试官：TypeScript 相比 JavaScript 有什么核心优势？**

<details>
<summary>查看回答</summary>

> 主要有五个方面：
>
> 1. **静态类型检查**：编译期发现类型错误，避免运行时崩溃，IDE 补全和重构支持更强
> 2. **可读性和可维护性**：类型即文档，函数签名直接说明输入输出，大型项目重构更安全
> 3. **先进语言特性**：泛型、枚举、装饰器、接口、抽象类等，ES 标准尚未完全落地的能力
> 4. **工程化支持**：`tsconfig.json` 统一配置，与 ESLint、Vite、Webpack 深度集成
> 5. **团队协作**：接口契约明确，前后端联调沟通成本低
>
> **补充要点**：TS 不是替代品而是超集，所有 JS 都是合法 TS；类型只在编译期存在，运行时会被擦除（Type Erasure）。

</details>

---

**面试官：`any`、`unknown`、`never`、`void` 有什么区别？**

<details>
<summary>查看回答</summary>

> | 类型      | 含义         | 可赋值给任意类型？ | 可被任意类型赋值？ | 典型场景             |
> | --------- | ------------ | ------------------ | ------------------ | -------------------- |
> | `any`     | 关闭类型检查 | ✅                 | ✅                 | 兼容老代码、临时绕过 |
> | `unknown` | 安全的 any   | ❌                 | ✅                 | 不确定类型的外部数据 |
> | `never`   | 永不发生     | ✅（底类型）       | ❌                 | 抛错函数、穷尽检查   |
> | `void`    | 无返回值     | ❌                 | ❌                 | 函数没有 return      |
>
> **关键区别**：
>
> - `any` 直接关闭检查，能随意调方法（危险）
> - `unknown` 使用前必须先用 `typeof`/`instanceof` 收窄才能操作（安全）
> - `never` 是所有类型的子类型，常用于 switch 的 default 做穷尽性检查
> - `void` 专门标注函数无显式返回值
>
> **加分点**：`unknown` 是 TS 3.0 引入的，设计目标就是替代不安全的 `any`。

</details>

---

**面试官：`interface` 和 `type` 有什么区别？什么时候用哪个？**

<details>
<summary>查看回答</summary>

> | 特性               | `interface`             | `type`          |
> | ------------------ | ----------------------- | --------------- |
> | 扩展方式           | `extends`               | `&` 交叉类型    |
> | 同名声明           | ✅ 自动合并（声明合并） | ❌ 重复声明报错 |
> | 联合/元组/基础类型 | ❌ 不行                 | ✅ 可以         |
> | 性能               | 稍好                    | 稍差            |
>
> **选择建议**：
>
> - 定义对象/类结构 → `interface`（官方推荐优先用）
> - 需要声明合并（扩展第三方库类型）→ `interface`
> - 联合类型、元组、映射类型、复杂类型运算 → `type`
>
> **加分点**：声明合并对扩展第三方库类型非常有用，比如给 `Window` 接口添加自定义属性。

</details>

---

**面试官：TypeScript 中的基本类型有哪些？`null` 和 `undefined` 有什么区别？**

<details>
<summary>查看回答</summary>

> **基本类型**：`string`、`number`、`boolean`、`null`、`undefined`、`symbol`、`bigint`
>
> **扩展类型**：`any`、`unknown`、`void`、`never`、`tuple`、`enum`、`object`、`literal`（字面量类型）
>
> **null vs undefined**：
> | 维度 | `null` | `undefined` |
> |------|---|---|
> | 语义 | 显式表示「无值」 | 表示「未定义」 |
> | 默认值 | 不会自动赋值 | 变量未初始化时的默认值 |
> | 可选参数 | 较少直接使用 | 函数可选参数默认值场景多 |
>
> **重要配置**：开启 `strictNullChecks` 后，`null` 和 `undefined` 不能随意赋值给其他类型，需要用联合类型 `string | null` 显式声明。这是 TS 最重要的严格模式选项之一。

</details>

---

**面试官：什么是类型推断？TypeScript 在什么情况下会推断类型？**

<details>
<summary>查看回答</summary>

> 类型推断是 TS 在没有显式注解时，根据上下文自动推导类型的能力。
>
> **四种推断场景**：
>
> 1. **变量初始化**：`let a = 1` 推断为 `number`
> 2. **函数返回值**：根据 return 语句自动推断
> 3. **上下文类型推断**：如事件回调中 `event` 会被推断为对应事件类型
> 4. **泛型参数推断**：`identity("hello")` 中 T 被推断为 `"hello"` 字面量类型
>
> **何时需要显式注解**：
>
> - 需要更宽类型时（如联合类型）
> - 函数参数必须注解
> - 复杂对象提高可读性
>
> **加分点**：`const` 声明时推断更窄（字面量类型），`let` 声明推断更宽（基础类型）。

</details>

---

## 二、泛型

**面试官：什么是泛型？有什么实际应用场景？**

<details>
<summary>查看回答</summary>

> 泛型是在定义函数、接口、类时不指定具体类型，使用类型参数，调用时再传入具体类型，实现「类型级别的参数化」。
>
> **实际场景**：
>
> 1. **通用 API 请求函数**：`request<User>('/api/user')` 返回 `Promise<User>`
> 2. **工具函数**：如 `pick<T, K>(obj, keys)` 从对象中选取部分属性
> 3. **React 组件 props**：`ListProps<T>` 让列表组件支持任意类型数据
> 4. **数据结构**：`Stack<T>` 泛型栈
>
> **加分点**：泛型让代码既灵活又安全；泛型约束 `extends` 可以限制类型参数必须满足某种结构。

</details>

---

**面试官：泛型约束 `extends` 和默认类型 `=` 怎么用？**

<details>
<summary>查看回答</summary>

> **泛型约束**：用 `extends` 限制泛型参数必须满足某种结构。
>
> - `T extends { length: number }` — T 必须有 length 属性
> - `K extends keyof T` — K 必须是 T 的键之一
>
> **默认类型**：给泛型参数一个默认值，调用时可以不传。
>
> - `interface ResponseData<T = any>` — 不指定 T 时默认为 any
>
> **组合使用**：约束和默认可以同时用，如 `function createArray<T = string>(length: number, value: T): T[]`
>
> **加分点**：`keyof T` 拿到对象类型的键联合类型，`T[K]` 是索引访问类型，两者组合是实现类型安全属性访问的基础。

</details>

---

## 三、高级类型

**面试官：常用内置工具类型有哪些？它们分别做了什么？**

<details>
<summary>查看回答</summary>

> | 工具类型         | 作用                          |
> | ---------------- | ----------------------------- |
> | `Partial<T>`     | 所有属性变为可选              |
> | `Required<T>`    | 所有属性变为必选              |
> | `Readonly<T>`    | 所有属性变为只读              |
> | `Pick<T, K>`     | 从 T 中选取 K 属性            |
> | `Omit<T, K>`     | 从 T 中排除 K 属性            |
> | `Record<K, T>`   | 构造键为 K、值为 T 的对象类型 |
> | `Exclude<T, U>`  | 从联合类型 T 中排除 U         |
> | `Extract<T, U>`  | 从联合类型 T 中提取 U         |
> | `NonNullable<T>` | 排除 null/undefined           |
> | `ReturnType<T>`  | 获取函数返回类型              |
> | `Parameters<T>`  | 获取函数参数元组类型          |
>
> **加分点**：这些工具类型都是基于映射类型实现的，能手写 `Partial`、`Pick`、`Readonly` 说明理解了底层原理。

</details>

---

**面试官：什么是映射类型（Mapped Types）？如何手写实现 Partial？**

<details>
<summary>查看回答</summary>

> 映射类型是遍历已有类型的每个属性，生成新类型的能力。
>
> **手写 Partial**：`type MyPartial<T> = { [P in keyof T]?: T[P] }`
>
> **关键语法**：
>
> - `keyof T`：获取 T 的所有键联合类型
> - `in`：遍历键联合
> - `T[P]`：索引访问，获取属性值类型
> - `?` / `readonly`：添加修饰符
> - `-?` / `-readonly`：移除修饰符
>
> **手写 Readonly**：`type MyReadonly<T> = { readonly [P in keyof T]: T[P] }`
>
> **手写 Pick**：`type MyPick<T, K extends keyof T> = { [P in K]: T[P] }`
>
> **加分点**：映射类型是 TS 类型系统的核心能力之一，`keyof` + `in` + `T[P]` 的组合能实现绝大多数工具类型。

</details>

---

**面试官：条件类型、infer 和分布式条件类型是什么？**

<details>
<summary>查看回答</summary>

> **条件类型**：`T extends U ? X : Y`，类似三元表达式，根据类型关系返回不同类型。
>
> **infer**：在条件类型中声明一个待推断的类型变量。
>
> - 提取数组元素类型：`T extends (infer E)[] ? E : never`
> - 提取函数返回类型：`T extends (...args: any[]) => infer R ? R : never`
>
> **分布式条件类型**：当泛型参数是联合类型时，会自动分发到每个成员上。
>
> - `ToArray<string | number>` 结果是 `string[] | number[]`（不是 `(string | number)[]`）
> - 阻止分布：用元组包裹 `[T] extends [any]`
>
> **加分点**：`infer` 只能在 `extends` 条件类型的 true 分支中使用；分布式条件类型是实现 `Exclude`、`Extract` 等工具类型的基础。

</details>

---

## 四、类型推断、断言与守卫

**面试官：类型断言和类型守卫有什么区别？**

<details>
<summary>查看回答</summary>

> **类型断言**：告诉编译器「我知道这个值是什么类型」，用 `as` 语法，**没有运行时检查**，是编译期的「欺骗」。
>
> **类型守卫**：在运行时检查类型，帮助 TS 自动收窄类型，**更安全**。
>
> **常见类型守卫方式**：
> | 方式 | 示例 |
> |------|------|
> | `typeof` | `typeof x === 'string'` |
> | `instanceof` | `x instanceof Date` |
> | `in` | `'name' in x` |
> | 字面量判断 | `x.type === 'success'` |
> | 自定义守卫 | `function isFish(pet): pet is Fish` |
>
> **加分点**：类型断言滥用会失去类型安全，应优先使用类型守卫；自定义类型守卫通过 `is` 关键字让 TS 在 if 分支中自动收窄。

</details>

---

**面试官：什么是可区分联合（Discriminated Unions）？**

<details>
<summary>查看回答</summary>

> 可区分联合是通过共同的可区分字段（tag/discriminant），让联合类型的每个分支可以被精确收窄的模式。
>
> **典型写法**：每个接口有一个相同名字但不同字面量值的字段（如 `kind`），在 `switch` 中根据这个字段分支，TS 自动收窄到具体类型。
>
> **优势**：
>
> - 每个分支自动收窄到具体类型，访问独有属性不报错
> - 结合 `never` 在 default 分支可检查是否处理了所有情况
> - 比可选属性 + if/else 更安全、更可维护
>
> **加分点**：Redux action、React 组件状态机、API 响应类型非常适合用可区分联合。

</details>

---

## 五、类与接口

**面试官：TypeScript 中类的访问修饰符有哪些？抽象类是什么？**

<details>
<summary>查看回答</summary>

> **访问修饰符**：
> | 修饰符 | 可访问范围 |
> |------|------|
> | `public`（默认） | 任意位置 |
> | `private` | 类内部 |
> | `protected` | 类内部 + 子类 |
> | `readonly` | 初始化后不可修改 |
>
> **参数属性简写**：`constructor(public name: string, private age: number) {}` 自动声明并赋值。
>
> **抽象类**：用 `abstract` 修饰，不能直接实例化，只能被继承。抽象方法没有实现体，子类必须实现。
>
> **加分点**：`private` 只在编译期检查，运行时仍可访问；ES 标准的 `#` 私有字段是运行时真正私有。抽象类 vs 接口：抽象类可以有实现代码和属性，接口只有结构约束。

</details>

---

**面试官：`implements` 和 `extends` 有什么区别？**

<details>
<summary>查看回答</summary>

> | 维度     | `extends`        | `implements`           |
> | -------- | ---------------- | ---------------------- |
> | 目标     | 类               | 接口/类型别名          |
> | 继承实现 | 继承父类实现代码 | 只约束结构，不继承实现 |
> | 多继承   | ❌ 单继承        | ✅ 可实现多个接口      |
> | 构造函数 | 继承父类构造     | 无                     |
>
> **一个类可以同时 extends 和 implements**：`class Eagle extends Bird implements CanFly, CanRun`
>
> **加分点**：接口是「契约」，只检查结构是否匹配；类继承是「代码复用」，继承实现和原型链。

</details>

---

## 六、类型兼容性与泛型进阶

**面试官：什么是结构化类型系统？它与名义类型系统有什么区别？**

<details>
<summary>查看回答</summary>

> **结构化类型系统**：只要结构相同，类型就兼容（鸭子类型）。TS 采用这种方式。
>
> **名义类型系统**：必须显式声明类型关系（如 Java、C#），即使结构一样也不兼容。
>
> **TS 选择结构化类型的原因**：与 JS 的鸭子类型习惯保持一致，让类型更灵活。
>
> **例外**：带有 `private`/`protected` 成员的类在兼容时要求来源相同，不能仅靠结构匹配。
>
> **加分点**：结构化类型让 TS 更灵活但也可能隐藏风险；需要名义类型效果时可用 branded types（品牌类型）模式。

</details>

---

**面试官：协变、逆变、双向协变是什么？**

<details>
<summary>查看回答</summary>

> **协变**：子类型可以赋值给父类型。数组、对象属性、函数返回值都是协变的。
>
> - `Dog[]` 可赋值给 `Animal[]`
>
> **逆变**：父类型可以赋值给子类型。函数参数是逆变的。
>
> - `(a: Animal) => void` 可赋值给 `(d: Dog) => void`
>
> **双向协变**：TS 2.x 默认函数参数是双向协变的（不安全）。
>
> **正确做法**：开启 `strictFunctionTypes: true`，让函数参数采用正确的逆变检查。
>
> **加分点**：记住口诀「返回值协变，参数逆变」是类型安全的保证。

</details>

---

## 七、模块、声明文件与工程化

**面试官：`declare` 关键字有什么用？`.d.ts` 文件是什么？**

<details>
<summary>查看回答</summary>

> **declare**：告诉 TS「这个变量/模块/类型已经存在」，只提供类型信息，不生成实际代码。用于声明全局变量、第三方模块、非 JS 资源（如图片、CSS）等。
>
> **.d.ts 文件**：只包含类型声明不包含实现的文件，为 JS 库提供类型支持。
>
> - `@types/xxx` 组织为大量 JS 库提供社区类型声明
> - 编译后不会输出为 JS
> - 可通过 `declaration: true` 自动生成
>
> **加分点**：`declare module '*.png'` 可以让 TS 识别图片等非 JS 资源的导入。

</details>

---

**面试官：`tsconfig.json` 中常用的编译选项有哪些？**

<details>
<summary>查看回答</summary>

> | 选项                | 作用                  | 推荐值     |
> | ------------------- | --------------------- | ---------- |
> | `target`            | 编译目标 JS 版本      | ES2020     |
> | `module`            | 模块系统              | ESNext     |
> | `strict`            | 开启所有严格类型检查  | true       |
> | `esModuleInterop`   | 改善 CJS 模块导入体验 | true       |
> | `skipLibCheck`      | 跳过声明文件类型检查  | true       |
> | `resolveJsonModule` | 支持导入 JSON         | true       |
> | `baseUrl` / `paths` | 路径别名              | 按项目配置 |
> | `noImplicitAny`     | 禁止隐式 any          | true       |
> | `strictNullChecks`  | 严格空值检查          | true       |
>
> **加分点**：`strict: true` 会同时开启 `noImplicitAny`、`strictNullChecks`、`strictFunctionTypes` 等多个严格选项；`skipLibCheck` 可加快编译但可能隐藏第三方库类型问题。

</details>

---

**面试官：TypeScript 中的模块解析策略是什么？**

<details>
<summary>查看回答</summary>

> 两种策略：`classic`（老旧）和 `node`（现代项目默认，模拟 Node.js 解析）。
>
> **Node 解析流程**：
>
> - 相对路径 `./bar` → 找 bar.ts → bar/package.json → bar/index.ts
> - 非相对路径 `bar` → node_modules/bar → @types/bar
>
> **路径别名**：`baseUrl` + `paths` 配置（如 `"@/*": ["src/*"]`），但编译期路径别名需要配合打包工具（Webpack/Vite）的 alias 才能在运行时生效。
>
> **加分点**：能区分编译期解析和运行时解析是两个独立环节。

</details>

---

## 八、进阶与最佳实践

**面试官：TypeScript 项目中有哪些常见最佳实践？**

<details>
<summary>查看回答</summary>

> 1. **开启 strict 模式**：一步到位开启所有严格检查
> 2. **避免滥用 any**：用 `unknown` + 类型守卫替代，或用泛型
> 3. **优先 interface，复杂类型用 type**
> 4. **合理使用泛型约束**：`T extends { length: number }` 限制参数结构
> 5. **字面量类型替代 magic string**：`type Status = 'pending' | 'success' | 'error'`
> 6. **善用工具类型减少重复**：`Partial<User>`、`Pick<User, 'id' | 'name'>`
> 7. **为第三方 JS 库补充 .d.ts 声明**
> 8. **类型与运行时校验结合**：用 Zod/Yup 等库做运行时校验 + 类型推导
> 9. **抽取公共类型到 `types/` 目录**
> 10. **泛型参数尽量可推断**，减少调用方显式传入的负担
>
> **加分点**：TS 的价值在于「约束」而非「限制」；类型安全需要编译期和运行时双重保障。

</details>

---

## 使用说明

- 点击 **「查看回答」** 即可展开答案
- 格式：面试官提问 → 候选人作答（含要点/对比/加分点）
- 适合面试前快速模拟练习
