# TypeScript 面试题（口述版）

共 **20 题**，涵盖基础类型、接口与类、泛型、高级类型、类型推断与守卫、编译配置与工程化。所有答案已精简为口述要点，可直接用于面试口头表达。

---

## 一、基础类型与核心概念

### 1. TypeScript 相比 JavaScript 有什么核心优势？

<details>
<summary>查看答案</summary>

**五大核心优势：**

1. **静态类型检查** — 编译期发现错误，避免运行时崩溃；IDE 自动补全和跳转更强
2. **可读性与可维护性** — 类型即文档，函数签名说明输入输出，大型项目重构更安全
3. **先进语言特性** — 泛型、枚举、装饰器、接口、抽象类等 ES 标准尚未完全落地的能力
4. **工程化支持** — tsconfig 统一配置，与 ESLint、Vite、Webpack 深度集成
5. **团队协作** — 接口契约明确，减少前后端联调和沟通成本

**关键补充：**

- TS 是 JS 的超集，不是替代品，所有 JS 代码都是合法 TS
- TS 最终编译为 JS 运行，类型只在编译期存在（Type Erasure）

**面试加分点**：强调「类型只在编译期存在，运行时被擦除」；能区分静态类型检查与运行时校验的边界。

</details>

---

### 2. `any`、`unknown`、`never`、`void` 有什么区别？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 类型      | 含义                   | 可赋值给任意类型？ | 可被任意类型赋值？ | 常见场景             |
| --------- | ---------------------- | ------------------ | ------------------ | -------------------- |
| `any`     | 任意类型，关闭类型检查 | 是                 | 是                 | 兼容老代码、临时绕过 |
| `unknown` | 未知类型，安全的 any   | 否                 | 是                 | 不确定类型的外部数据 |
| `never`   | 永不发生的类型         | 是                 | 否                 | 抛错函数、穷尽检查   |
| `void`    | 无返回值               | 否                 | 否                 | 函数没有 return      |

**口述要点：**

- **any**：关闭所有检查，可直接访问属性和调用方法，相当于回到 JS，不安全
- **unknown**：必须先用 `typeof` / `instanceof` / 类型断言收窄后才能操作，安全版的 any
- **never**：两个典型场景 — ①抛异常的函数返回类型（永远不会正常返回）；②穷尽检查（switch 的 default 分支赋值给 never，遗漏分支会报错）
- **void**：函数没有显式 return 时标注的返回类型

**面试加分点**：`unknown` 是 TS 3.0 引入的，设计目标就是替代不安全的 `any`；`never` 是所有类型的子类型（bottom type）。

</details>

---

### 3. `interface` 和 `type` 有什么区别？什么时候用哪个？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 特性                | `interface`          | `type`       |
| ------------------- | -------------------- | ------------ |
| 扩展方式            | `extends`            | `&` 交叉类型 |
| 同名声明            | 自动合并（声明合并） | 重复声明报错 |
| 描述对象形状        | 最自然               | 也可以       |
| 联合/元组/基础类型  | 不行                 | 可以         |
| 性能（大量使用时）  | 稍好                 | 稍差         |
| 实现类 `implements` | 支持                 | 也支持       |

**选择建议：**

- 定义对象/类结构、需要声明合并 → 用 `interface`
- 联合类型、元组、映射类型、复杂类型运算 → 用 `type`
- 官方推荐：优先 `interface`，复杂类型再用 `type`

**关键点：** interface 的声明合并对扩展第三方库类型非常有用 — 同名 interface 会自动合并属性，而 type 同名直接报错。

**面试加分点**：能说出声明合并的实际用途（如给 Window 对象扩展属性、给第三方库补充类型）。

</details>

---

### 4. TypeScript 中的基本类型有哪些？`null` 和 `undefined` 有什么区别？

<details>
<summary>查看答案</summary>

**7 种基本类型：** `string`、`number`、`boolean`、`null`、`undefined`、`symbol`、`bigint`

**扩展类型：**

| 类型      | 说明                         | 典型场景           |
| --------- | ---------------------------- | ------------------ |
| `any`     | 关闭类型检查                 | 旧代码迁移         |
| `unknown` | 安全顶层类型，使用前必须收窄 | API 返回值         |
| `void`    | 无返回值                     | 函数标注           |
| `never`   | 永不到达的类型               | 抛异常、穷尽检查   |
| `tuple`   | 固定长度和类型的数组         | 坐标、键值对       |
| `enum`    | 枚举，一组命名常量           | 状态码、方向       |
| `object`  | 非原始类型的统称             | 约束参数为引用类型 |
| `literal` | 字面量类型，精确到具体值     | 有限选项集合       |

**null 与 undefined 区别：**

| 维度     | `null`           | `undefined`          |
| -------- | ---------------- | -------------------- |
| 语义     | 显式表示「无值」 | 表示「未定义」       |
| 默认值   | 不会自动赋值     | 变量未初始化的默认值 |
| 可选参数 | 较少直接使用     | 函数可选参数默认值   |

**关键补充：** 开启 `strictNullChecks` 后，null 和 undefined 不能随意赋给其他类型，必须显式用联合类型声明（如 `string | null`）。这是 TS 最重要的严格选项之一，可大幅减少空指针错误。

</details>

---

### 5. 什么是类型推断？TypeScript 在什么情况下会推断类型？

<details>
<summary>查看答案</summary>

**定义：** TS 在没有显式注解时，根据上下文自动推导出类型。

**四种推断场景：**

1. **变量初始化推断** — `let a = 1` 自动推断为 number
2. **函数返回值推断** — 根据 return 语句推断返回类型
3. **上下文类型推断（Contextual Typing）** — 如事件回调的参数会被推断为对应 Event 类型
4. **类型参数推断** — 泛型函数调用时根据传入参数推断 T

**何时需要显式注解：**

- 需要更宽的类型时（如字面量需要联合类型）
- 函数参数必须注解（TS 不会推断入参）
- 复杂对象为提高可读性

**面试加分点**：能区分「最佳公共类型推断」和「上下文类型推断」；知道 `const` 声明时推断更窄（字面量类型），`let` 推断更宽（基础类型）。

</details>

---

## 二、泛型

### 6. 什么是泛型？有什么实际应用场景？

<details>
<summary>查看答案</summary>

**定义：** 在定义函数、接口、类时不指定具体类型，使用类型参数（如 `<T>`），使用时再传入具体类型。本质是「类型级别的参数化」。

**四大应用场景：**

1. **通用 API 请求函数** — `request<T>(url): Promise<T>`，调用时传入期望的响应类型
2. **通用工具函数** — 如 `pick<T, K>(obj, keys)` 从对象中选取指定属性
3. **React 组件 props** — `ListProps<T>` 让列表组件可渲染任意类型的数据
4. **数据结构** — 如 `Stack<T>` 泛型栈，push/pop 保持类型一致

**核心价值：** 让代码既灵活（不限定具体类型）又安全（类型约束仍然存在）。

**面试加分点**：能说出泛型约束 `extends` 的用途 — 限制 T 必须满足某种结构，避免「为所欲为」。

</details>

---

### 7. 泛型约束 `extends` 和默认类型 `=` 怎么用？

<details>
<summary>查看答案</summary>

**泛型约束：** 用 `T extends SomeType` 限制泛型参数必须满足某种结构。

- 例：`T extends { length: number }` 表示 T 必须有 length 属性
- `K extends keyof T` 表示 K 只能是 T 的键名之一

**默认类型：** 用 `T = DefaultType` 为泛型提供默认值，调用时可不传。

- 例：`interface ResponseData<T = any>` 不传 T 时默认为 any

**组合使用：** 约束和默认可以同时存在，如 `<T extends object = {}>`

**keyof + T[K] 组合：**

- `keyof T` — 获取对象类型的所有键组成的联合类型
- `T[K]` — 索引访问类型，获取 T 中键 K 对应的值类型
- 经典模式：`getProperty<T, K extends keyof T>(obj: T, key: K): T[K]`

**面试加分点**：能解释泛型约束让泛型「不再为所欲为」，保持灵活性的同时增加安全约束。

</details>

---

## 三、高级类型

### 8. 常用内置工具类型有哪些？它们分别做了什么？

<details>
<summary>查看答案</summary>

**核心工具类型表：**

| 工具类型         | 作用                 | 一句话说明                 |
| ---------------- | -------------------- | -------------------------- |
| `Partial<T>`     | 所有属性变可选       | 用于更新场景，只传部分字段 |
| `Required<T>`    | 所有属性变必选       | Partial 的反操作           |
| `Readonly<T>`    | 所有属性变只读       | 防止意外修改               |
| `Pick<T, K>`     | 选取指定属性         | 从大类型中取子集           |
| `Omit<T, K>`     | 排除指定属性         | 从大类型中去掉某些字段     |
| `Record<K, T>`   | 构造键值对类型       | 快速定义字典/映射结构      |
| `Exclude<T, U>`  | 差集                 | 从联合类型中排除某些成员   |
| `Extract<T, U>`  | 交集                 | 从联合类型中提取某些成员   |
| `NonNullable<T>` | 排除 null/undefined  | 得到非空类型               |
| `ReturnType<T>`  | 获取函数返回类型     | 不想手动写返回类型时用     |
| `Parameters<T>`  | 获取函数参数元组类型 | 获取函数签名信息           |

**面试加分点**：这些工具类型都是基于映射类型实现的；能手写 Partial、Pick、Omit 的实现。

</details>

---

### 9. 什么是映射类型（Mapped Types）？如何手写实现 Partial？

<details>
<summary>查看答案</summary>

**定义：** 遍历已有类型的每个属性，按照某种规则生成新类型。

**核心语法：** `{ [P in keyof T]: T[P] }` — 遍历 T 的所有键，保持值类型不变

**手写实现思路：**

- **Partial**：`[P in keyof T]?: T[P]` — 在键后面加 `?` 使属性变可选
- **Readonly**：`readonly [P in keyof T]: T[P]` — 在键前面加 `readonly`
- **Pick**：`[P in K]: T[P]`，其中 `K extends keyof T` — 只遍历指定的键

**修饰符操作：**

- 加可选：`?`
- 移除可选：`-?`（变为必选）
- 移除只读：`-readonly`

**关键语法组件：**

- `keyof T` — 获取所有键的联合类型
- `in` — 遍历联合类型的每个成员
- `T[P]` — 索引访问，获取属性值类型

**面试加分点**：映射类型是 TS 类型系统的核心能力，`keyof` + `in` + `T[P]` 三者组合能实现绝大多数工具类型。

</details>

---

### 10. 条件类型、infer 和分布式条件类型是什么？

<details>
<summary>查看答案</summary>

**1）条件类型：** `T extends U ? X : Y` — 类似三元表达式，根据类型关系选择结果类型

**2）infer 推断：** 在条件类型的 `extends` 子句中用 `infer` 声明待推断的类型变量

- 提取数组元素类型：`T extends (infer E)[] ? E : never`
- 提取函数返回类型：`T extends (...args: any[]) => infer R ? R : never`
- infer 只能在 extends 条件的 true 分支中使用

**3）分布式条件类型：** 当泛型参数是联合类型时，条件类型会自动分发到每个成员

- `ToArray<string | number>` 结果是 `string[] | number[]`，不是 `(string | number)[]`
- 阻止分布：用元组包裹 `[T] extends [any]`

**面试加分点**：分布式条件类型是 TS 实现很多高级工具类型的基础（如 Exclude、Extract）；infer 是类型层面的「模式匹配」。

</details>

---

## 四、类型推断、断言与守卫

### 11. 类型断言和类型守卫有什么区别？

<details>
<summary>查看答案</summary>

**类型断言：** 告诉编译器「我知道这个值是什么类型」，用 `as` 语法，**没有运行时检查**，是编译期的「信任声明」。

**类型守卫：** 在运行时检查类型，帮助 TS 自动收窄类型范围，**有运行时检查**，更安全。

**类型守卫的六种方式：**

| 方式         | 说明                                  |
| ------------ | ------------------------------------- |
| `typeof`     | 判断基础类型（string/number/boolean） |
| `instanceof` | 判断实例所属类                        |
| `in`         | 判断对象是否有某个属性                |
| 字面量判断   | `x.type === 'success'` 可区分联合     |
| 自定义守卫   | `function isFish(pet): pet is Fish`   |
| 可区分联合   | 通过共同 tag 字段收窄                 |

**自定义类型守卫：** 函数返回值类型写成 `paramName is Type` 的形式，返回 true 时 TS 将参数收窄为该类型。

**面试加分点**：类型断言是「编译期欺骗」，滥用会失去类型安全；类型守卫是更安全的做法，应优先使用。

</details>

---

### 12. 什么是可区分联合（Discriminated Unions）？

<details>
<summary>查看答案</summary>

**定义：** 联合类型中每个成员都有一个共同的「标签字段」（discriminant），通过判断这个字段可以精确收窄到具体类型。

**核心模式：**

- 每个接口定义一个相同的字面量类型字段（如 `kind: 'circle'`）
- 用 `switch(shape.kind)` 分支处理，每个 case 中自动收窄为具体类型
- 在 default 分支赋值给 `never` 可做穷尽检查

**优势：**

- 每个分支自动收窄到具体类型，无需手动断言
- 结合 `never` 检查所有分支是否处理完毕
- 比可选属性 + if/else 更安全、更清晰

**典型应用：** Redux action、React 组件状态机、API 响应（成功/失败/加载中）

**面试加分点**：能说出 `kind` 字段就是 discriminant（区分标记）；这个模式在状态管理中非常常见。

</details>

---

## 五、类与接口

### 13. TypeScript 中类的访问修饰符有哪些？抽象类是什么？

<details>
<summary>查看答案</summary>

**四种修饰符：**

| 修饰符      | 含义         | 可访问范围       |
| ----------- | ------------ | ---------------- |
| `public`    | 公开（默认） | 任意位置         |
| `private`   | 私有         | 仅类内部         |
| `protected` | 受保护       | 类内部 + 子类    |
| `readonly`  | 只读         | 初始化后不可修改 |

**参数属性简写：** 在构造函数参数前加修饰符，自动声明并赋值为类属性，省去手动声明和赋值。

**抽象类（abstract class）：**

- 不能被实例化，只能被继承
- 可以包含抽象方法（子类必须实现）和具体方法（子类可直接使用）
- 相比接口：抽象类可以有实现代码，接口只有结构约束

**private vs # 私有字段：**

- `private` — 编译期检查，运行时实际上还是能访问
- `#` — ES 标准的真正运行时私有，外部完全无法访问

**面试加分点**：能区分 private（编译期私有）和 # 字段（运行时真正私有）；能解释抽象类与接口的区别（抽象类可以有实现）。

</details>

---

### 14. `implements` 和 `extends` 有什么区别？

<details>
<summary>查看答案</summary>

**核心区别：**

| 维度     | `extends`                     | `implements`           |
| -------- | ----------------------------- | ---------------------- |
| 目标     | 继承类                        | 实现接口               |
| 继承实现 | 继承父类的属性和方法实现      | 只约束结构，不继承实现 |
| 多继承   | 单继承（只能 extends 一个类） | 可同时实现多个接口     |
| 构造函数 | 继承父类构造函数              | 无                     |

**口述要点：**

- `extends` 是「代码复用」— 子类获得父类的实现和原型链
- `implements` 是「契约约束」— 只检查类是否满足接口结构，不提供任何实现
- 一个类可以同时 extends 一个父类 + implements 多个接口

**面试加分点**：接口是契约，只检查结构；类继承是代码复用，会继承实现和原型链。

</details>

---

## 六、类型兼容性与泛型进阶

### 15. 什么是结构化类型系统？它与名义类型系统有什么区别？

<details>
<summary>查看答案</summary>

**结构化类型系统（Structural Typing）：** 只要结构相同，类型就兼容。不需要显式声明继承关系。

**名义类型系统（Nominal Typing）：** 必须显式声明类型关系才能兼容（如 Java、C# 的 implements）。

**TS 选择结构化类型的原因：** 与 JS 的鸭子类型习惯保持一致 —「如果它看起来像鸭子、叫起来像鸭子，那它就是鸭子」。

**实际表现：**

- 一个对象只要具备接口要求的所有属性，就可以赋值给该接口类型
- 即使没有 implements 声明，结构满足就兼容

**例外情况：** 类的 `private` 和 `protected` 成员会要求来源相同 — 两个类有相同结构但各自声明了 private 字段，它们不兼容。

**面试加分点**：结构化类型让 TS 更灵活，但也可能隐藏风险（额外属性不报错）；对象字面量直接赋值时有「多余属性检查」作为安全网。

</details>

---

### 16. 协变、逆变、双向协变是什么？

<details>
<summary>查看答案</summary>

**前提：** Dog extends Animal，Dog 是 Animal 的子类型。

**协变（Covariance）：** 子类型方向保持一致 — Dog[] 可赋值给 Animal[]

- 适用于：数组、对象属性、函数返回值
- 直觉：「能用动物的地方，给一只狗也可以」

**逆变（Contravariance）：** 子类型方向反转 — 处理 Animal 的函数可赋值给处理 Dog 的函数

- 适用于：函数参数位置
- 直觉：「你需要一个能处理狗的函数，给你一个能处理所有动物的函数当然没问题」

**双向协变：** TS 2.x 默认函数参数是双向协变的（不安全）

- 开启 `strictFunctionTypes: true` 后，函数参数采用正确的逆变检查

**口诀：** 返回值协变，参数逆变 — 这是类型安全的。

**面试加分点**：能说出开启 `strictFunctionTypes` 后 TS 对函数参数采用正确的逆变；这个选项包含在 `strict: true` 中。

</details>

---

## 七、模块、声明文件与工程化

### 17. `declare` 关键字有什么用？`.d.ts` 文件是什么？

<details>
<summary>查看答案</summary>

**declare 的作用：** 告诉 TS「这个变量/模块/类型已经存在」，只提供类型信息，不生成实际 JS 代码。

**常见用法：**

- 声明全局变量：`declare const VERSION: string`
- 声明全局函数：`declare function $(selector: string): HTMLElement`
- 声明模块：`declare module 'my-lib' { ... }`
- 声明非 JS 资源：`declare module '*.png'`（让 TS 认识图片导入）

**.d.ts 文件：**

- 只包含类型声明，不包含实现代码
- 编译后不会输出为 JS
- 用于为 JS 库提供类型支持
- `@types` 组织为大量 JS 库提供了社区类型声明（如 `@types/react`）

**自动生成：** tsconfig 中设置 `declaration: true` 可自动从源码生成 .d.ts 文件。

**面试加分点**：`.d.ts` 在编译后不产出 JS；`@types` 组织维护了数千个 JS 库的类型声明。

</details>

---

### 18. `tsconfig.json` 中常用的编译选项有哪些？

<details>
<summary>查看答案</summary>

**核心选项表：**

| 选项                | 作用                  | 推荐值     |
| ------------------- | --------------------- | ---------- |
| `target`            | 编译目标 JS 版本      | ES2020     |
| `module`            | 模块系统              | ESNext     |
| `strict`            | 开启所有严格检查      | true       |
| `esModuleInterop`   | 改善 CJS 模块导入体验 | true       |
| `skipLibCheck`      | 跳过声明文件类型检查  | true       |
| `resolveJsonModule` | 支持导入 JSON         | true       |
| `moduleResolution`  | 模块解析策略          | node       |
| `baseUrl` / `paths` | 路径别名              | 按项目配置 |
| `outDir`            | 输出目录              | ./dist     |
| `sourceMap`         | 生成 source map       | true       |
| `noImplicitAny`     | 禁止隐式 any          | true       |
| `strictNullChecks`  | 严格空值检查          | true       |

**关键知识点：**

- `strict: true` 会同时开启 noImplicitAny、strictNullChecks、strictFunctionTypes 等多个选项
- `skipLibCheck` 能加快编译速度，但可能隐藏第三方库的类型问题
- `paths` 只解决编译期路径，运行时还需要打包工具配合 alias

**面试加分点**：能解释 strict 是一组选项的集合；paths 需要配合 Vite/Webpack 的 resolve.alias 才能运行时生效。

</details>

---

### 19. TypeScript 中的模块解析策略是什么？

<details>
<summary>查看答案</summary>

**两种策略：**

| 策略      | 特点                  | 适用场景     |
| --------- | --------------------- | ------------ |
| `classic` | 简单的相对路径查找    | TS 1.6 之前  |
| `node`    | 模拟 Node.js 模块解析 | 现代项目默认 |

**Node 策略的解析流程：**

- 相对路径 `./bar`：先找 bar.ts → bar/package.json 的 types → bar/index.ts
- 非相对路径 `bar`：逐级向上查找 node_modules/bar → node_modules/@types/bar

**路径别名：** 在 tsconfig 中配置 `baseUrl` + `paths` 可实现路径映射（如 `@/` 映射到 `src/`）

**关键点：** TS 编译时的路径别名只是告诉编译器去哪找类型，运行时还需要打包工具（Webpack 的 resolve.alias、Vite 的 resolve.alias）或 Node.js 的 tsconfig-paths 配合。

**面试加分点**：能区分编译期解析和运行时解析；能说明 paths 配置需要「双端配合」。

</details>

---

## 八、进阶与最佳实践

### 20. TypeScript 项目中有哪些常见最佳实践？

<details>
<summary>查看答案</summary>

**十条核心实践：**

1. **开启 strict 模式** — 一劳永逸开启所有严格检查
2. **避免滥用 any** — 用 `unknown` + 类型守卫替代，或用泛型保持灵活性
3. **优先 interface/type 而非内联类型** — 提高可读性和复用性
4. **合理使用泛型约束** — `T extends { length: number }` 保持灵活又安全
5. **字面量联合类型替代 magic string** — `type Status = 'pending' | 'success' | 'error'`
6. **泛型参数尽量可推断** — 让调用方无需显式传入类型参数
7. **为第三方 JS 库补充类型声明** — 在 `types/xxx.d.ts` 中用 declare module
8. **善用工具类型减少重复** — Partial、Pick、Omit 等避免手写重复类型
9. **类型与运行时校验结合** — 用 Zod/Yup 实现编译期 + 运行时双重保障
10. **持续重构类型** — 随业务发展迭代，抽取公共类型到 types/ 目录

**面试加分点**：TS 的价值在于「约束」而非「限制」；类型安全需要编译期和运行时双重保障（如 Zod 的 `z.infer` 同时得到类型和校验）。

</details>

---

## 使用说明

- 本文档为**口述版**，删除了所有多行代码块，以要点、表格和一句话思路替代
- 适用于面试前快速复习和口头表达练习
- 详细代码示例版请参考 `ts面试题.md`
