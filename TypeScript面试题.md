# TypeScript 面试题精选（含详细答案）

> 涵盖基础、进阶、高级、类型体操、工程化五大模块，共 80+ 题目，从原理到实战全面覆盖。

## 目录

- [一、基础篇（30 题）](#一基础篇)
- [二、进阶篇（20 题）](#二进阶篇)
- [三、高级篇（15 题）](#三高级篇)
- [四、类型体操篇（10 题）](#四类型体操篇)
- [五、工程化与原理篇（10 题）](#五工程化与原理篇)

---

## 一、基础篇

### 1. TypeScript 是什么？相比 JavaScript 有什么优势？

**答案：**

TypeScript 是 JavaScript 的**超集**，由微软开发，添加了**可选的静态类型**和基于类的面向对象编程。最终会被编译为纯 JavaScript 运行。

**核心优势：**

| 维度 | TypeScript | JavaScript |
|------|-----------|------------|
| 类型系统 | 静态类型（编译时检查） | 动态类型（运行时） |
| 错误发现 | 编译期 | 运行期 |
| IDE 支持 | 强大的智能提示、重构 | 较弱 |
| 大型项目 | 适合 | 维护成本高 |
| 学习成本 | 较高 | 低 |

**关键点**：TS 增强但不改变 JS 的运行时行为，所有 JS 代码都是合法的 TS 代码。

---

### 2. `any`、`unknown`、`never`、`void` 有什么区别？

**答案：**

```typescript
// any：任意类型，关闭类型检查（不推荐）
let a: any = 1;
a.foo.bar();  // 编译通过，运行可能崩溃

// unknown：类型安全的 any，使用前必须收窄
let u: unknown = 1;
// u.toFixed();  // ❌ 错误
if (typeof u === "number") u.toFixed();  // ✅

// never：永不存在的值（永不返回的函数返回值、不可能存在的类型）
function throwError(): never { throw new Error(); }
type X = string & number;  // never，矛盾的类型

// void：函数无返回值（实际是 undefined）
function log(): void { console.log("hi"); }
```

**核心区别：**
- `any`：顶层类型 + 底层类型，可赋值给任何类型，也可被任何类型赋值（破坏类型系统）
- `unknown`：仅顶层类型，可被任何类型赋值，但不能赋值给除 `any/unknown` 外的类型
- `never`：底层类型，是所有类型的子类型，**不可被任何类型赋值**（除自身）
- `void`：仅用于函数返回值场景，本质是 `undefined`

**面试加分点**：在 `T extends ... ? ... : ...` 中，`never` 会触发**分发分布式条件类型**特性。

---

### 3. `interface` 和 `type` 的区别？

**答案：**

| 特性 | interface | type |
|------|-----------|------|
| 描述对象/函数 | ✅ | ✅ |
| 联合类型 | ❌ | ✅ |
| 元组 | ❌（间接支持） | ✅ |
| 声明合并 | ✅ | ❌ |
| extends 继承 | ✅ | ✅（用 &） |
| 映射类型 | ❌ | ✅ |
| 计算属性 | ❌ | ✅ |
| 性能 | 略快（缓存友好） | 复杂 type 较慢 |

```typescript
// interface 声明合并
interface Window { customProp: string; }
interface Window { anotherProp: number; }
// 自动合并

// type 不可重复定义
type Foo = string;
// type Foo = number;  // ❌ 错误

// type 独有能力
type Union = string | number;
type Tuple = [string, number];
type Mapped<T> = { [K in keyof T]: T[K] };
```

**经验法则**：定义对象结构优先 `interface`，需要联合/映射/条件类型时用 `type`。

---

### 4. 数组和元组（Tuple）有什么区别？

**答案：**

```typescript
// 数组：长度不限，类型相同
let arr: number[] = [1, 2, 3, 4];

// 元组：长度固定，每个位置类型独立
let tuple: [string, number, boolean] = ["hi", 1, true];

// 元组的进阶用法
let labeled: [name: string, age: number] = ["Alice", 25];  // 标签元组
let optional: [string, number?] = ["hi"];                   // 可选元素
let rest: [string, ...number[]] = ["a", 1, 2, 3];           // 剩余元素
let readonlyT: readonly [string, number] = ["a", 1];        // 只读元组
```

**关键点**：元组的本质是**位置敏感**的数组，常用于函数参数列表（`Parameters<T>`）、返回多值（如 React 的 `useState`）。

---

### 5. 类型断言有几种写法？什么场景使用？

**答案：**

```typescript
// 1. as 语法（推荐）
const el = document.getElementById("app") as HTMLDivElement;

// 2. 尖括号语法（JSX 中冲突，不推荐）
const el2 = <HTMLDivElement>document.getElementById("app");

// 3. 非空断言 !
const el3 = document.getElementById("app")!;

// 4. const 断言
const config = { url: "x", port: 80 } as const;
// 类型为 { readonly url: "x"; readonly port: 80 }

// 5. 双重断言（强制断言，慎用）
const num = "hello" as unknown as number;
```

**使用场景：**
- 当你比 TS 编译器更了解类型时
- DOM 操作（`getElementById` 返回 `HTMLElement | null`）
- 第三方库类型不准确时
- 字面量类型推断（`as const`）

**注意**：断言不会真的转换类型，只是欺骗编译器，错误的断言会导致运行时错误。

---

### 6. `?.` 和 `??` 在 TS 中如何使用？

**答案：**

```typescript
// 可选链 ?.：安全地访问可能为 null/undefined 的属性
const user: { name?: { first?: string } } = {};
const firstName = user?.name?.first;  // string | undefined

// 函数调用
obj.method?.();  // 如果 method 不存在则不调用

// 数组访问
arr?.[0];

// 空值合并 ??：仅 null/undefined 时使用默认值
const x = null ?? "default";       // "default"
const y = 0 ?? "default";          // 0（与 || 不同！）
const z = "" ?? "default";         // ""

// 对比 ||
const a = 0 || "default";          // "default"（0 是 falsy）
```

**关键区别**：`||` 处理所有 falsy 值（0、""、false、null、undefined），`??` 仅处理 null/undefined，更精确。

---

### 7. 什么是字面量类型？有什么用？

**答案：**

字面量类型是表示**确切值**的类型，包括字符串、数字、布尔字面量。

```typescript
// 字符串字面量
type Direction = "up" | "down" | "left" | "right";
let dir: Direction = "up";  // ✅
// dir = "back";  // ❌

// 数字字面量
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

// 布尔字面量
type Success = true;

// 与函数结合
function move(direction: "up" | "down"): void {}

// 字面量类型扩展（widening）
const x = "hello";  // 推断为 "hello"
let y = "hello";    // 推断为 string

// as const 防止扩展
const config = { mode: "dark" } as const;  // mode: "dark"
```

**用途**：限定取值范围、可辨识联合（discriminated union）、API 参数约束、状态机建模。

---

### 8. 解释 TypeScript 中的 `readonly` 修饰符

**答案：**

`readonly` 用于将属性、数组、元组标记为只读，防止意外修改。

```typescript
// 1. 接口/类属性
interface Point {
  readonly x: number;
  readonly y: number;
}
const p: Point = { x: 1, y: 2 };
// p.x = 10;  // ❌ 错误

// 2. 只读数组
const arr: readonly number[] = [1, 2, 3];
// arr.push(4);  // ❌ 错误
// arr[0] = 0;   // ❌ 错误

// 3. ReadonlyArray<T>（等价写法）
const arr2: ReadonlyArray<number> = [1, 2, 3];

// 4. 只读元组
const tuple: readonly [string, number] = ["a", 1];

// 5. 类的 readonly 字段（只能在声明或构造函数中赋值）
class User {
  readonly id: number;
  constructor(id: number) {
    this.id = id;  // ✅
  }
}
```

**注意事项：**
- `readonly` 是**编译时**约束，运行时仍可通过类型断言绕过
- 只是**浅层只读**，深层属性仍可修改
- `Readonly<T>` 工具类型可批量将属性变为只读

---

### 9. `keyof` 和 `typeof` 操作符的区别？

**答案：**

```typescript
// typeof：从值推导出类型
const user = { name: "Alice", age: 25 };
type User = typeof user;  // { name: string; age: number }

const colors = ["red", "green", "blue"] as const;
type Color = typeof colors[number];  // "red" | "green" | "blue"

// keyof：获取类型的键名联合
type UserKeys = keyof User;  // "name" | "age"

interface Person { name: string; age: number; email: string; }
type PersonKey = keyof Person;  // "name" | "age" | "email"

// 组合使用：典型的属性访问器
function get<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// 实战：从对象常量生成类型
const ROLES = { ADMIN: "admin", USER: "user" } as const;
type Role = typeof ROLES[keyof typeof ROLES];  // "admin" | "user"
```

**口诀**：`typeof` 把**值**变**类型**，`keyof` 把**类型的键**变**联合类型**。

---

### 10. 什么是可辨识联合（Discriminated Union）？

**答案：**

通过一个**共同的字面量字段**作为"判别标签"，将多个接口组合成联合类型，TS 可基于该字段进行类型收窄。

```typescript
interface Circle {
  kind: "circle";    // 判别标签
  radius: number;
}

interface Square {
  kind: "square";    // 判别标签
  side: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Square | Triangle;

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;  // 自动收窄为 Circle
    case "square":
      return shape.side ** 2;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      // 穷尽性检查（exhaustive check）
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

**优势**：编译期保证所有分支被处理，新增类型时漏掉处理会立即报错。

---

### 11. TS 中的访问修饰符有哪些？

**答案：**

```typescript
class Animal {
  public name: string;        // 公开（默认）
  private age: number;        // 仅当前类
  protected dna: string;      // 当前类及子类
  readonly id: number;        // 只读
  static count = 0;           // 静态成员
  
  // ES 私有字段（运行时强制私有）
  #realPrivate = "secret";
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
    this.id = Date.now();
    this.dna = "ATCG";
  }
}

// 参数属性（简写）
class Point {
  constructor(
    public x: number,
    private y: number,
    readonly z: number
  ) {}
  // 自动声明并赋值
}
```

**`private` vs `#` 私有字段：**
- `private`：TS 编译时检查，运行时仍可访问
- `#`：ES 标准，**运行时真正私有**，无法通过任何方式访问

---

### 12. `extends` 在 TS 中有几种用法？

**答案：**

```typescript
// 1. 类继承
class Dog extends Animal {}

// 2. 接口继承
interface Admin extends User { permissions: string[]; }

// 3. 接口多继承
interface SuperAdmin extends User, Admin {}

// 4. 泛型约束
function logLength<T extends { length: number }>(x: T): void {
  console.log(x.length);
}

// 5. 条件类型
type IsString<T> = T extends string ? true : false;
type A = IsString<"hello">;  // true

// 6. 类实现接口（注意是 implements 不是 extends）
class MyClass implements MyInterface {}

// 7. 类型继承（type 用 & 模拟）
type Extended = Base & { extra: string };
```

---

### 13. 什么是函数重载？怎么实现？

**答案：**

通过定义多个**重载签名**+ 一个**实现签名**，让同一函数支持多种参数形式。

```typescript
// 重载签名（仅类型，不可执行）
function format(value: string): string;
function format(value: number): string;
function format(value: Date): string;

// 实现签名（必须兼容所有重载，外部不可见）
function format(value: string | number | Date): string {
  if (typeof value === "string") return value.trim();
  if (typeof value === "number") return value.toFixed(2);
  return value.toISOString();
}

format("hello");        // ✅ 调用 string 重载
format(3.14);           // ✅ 调用 number 重载
format(new Date());     // ✅ 调用 Date 重载
// format(true);        // ❌ 没有匹配的重载
```

**注意：**
- 实现签名外部**不可见**
- 重载顺序：从具体到宽泛（TS 按顺序匹配）
- 现代写法可用泛型/联合类型替代大部分重载

---

### 14. `null` 和 `undefined` 在 TS 中如何处理？

**答案：**

```typescript
// 默认情况下，null/undefined 是所有类型的子类型（不安全）
let s: string = null;  // 默认通过

// 开启 strictNullChecks 后（推荐）
// "strict": true  →  null/undefined 必须显式声明
let s2: string = null;  // ❌ 错误
let s3: string | null = null;  // ✅

// 处理可空值
function process(input?: string) {  // input: string | undefined
  // 1. 类型守卫
  if (input) console.log(input.toUpperCase());
  
  // 2. 可选链
  console.log(input?.toUpperCase());
  
  // 3. 空值合并
  console.log(input ?? "default");
  
  // 4. 非空断言（确定不为空时）
  console.log(input!.toUpperCase());
}

// NonNullable 工具类型
type Safe = NonNullable<string | null | undefined>;  // string
```

---

### 15. 怎样让接口的某个属性可选？

**答案：**

```typescript
// 1. 单个属性可选：?
interface User {
  name: string;
  age?: number;  // 可选
}

// 2. 全部可选：Partial<T>
interface User {
  name: string;
  age: number;
  email: string;
}
type PartialUser = Partial<User>;  // 所有字段变为可选

// 3. 部分字段可选：自定义工具类型
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;
type UserAgeOptional = PartialBy<User, "age">;  // 仅 age 可选

// 4. 反向：让所有字段必选
type RequiredUser = Required<PartialUser>;
```

---

### 16. 什么是索引签名？

**答案：**

索引签名描述**键名未知但类型已知**的对象。

```typescript
// 字符串键
interface StringMap {
  [key: string]: string;
}
const headers: StringMap = { "Content-Type": "application/json" };

// 数字键
interface NumberMap {
  [index: number]: string;
}
const arr: NumberMap = ["a", "b", "c"];

// 限制：所有显式属性必须兼容索引签名类型
interface Mixed {
  [key: string]: number;
  count: number;     // ✅
  // name: string;   // ❌ 不兼容
}

// Record 工具类型替代写法
type StringMap2 = Record<string, string>;

// 严格的键名（用映射类型替代）
type UserKeys = "name" | "email";
type StrictUser = Record<UserKeys, string>;
```

---

### 17. 解释 `as const` 的作用

**答案：**

`as const` 是 TS 3.4 引入的**常量断言**，用于：
1. 字符串/数字字面量推断为字面量类型而非 `string/number`
2. 对象所有属性变为 `readonly`
3. 数组变为只读元组

```typescript
// 不加 as const
const config = {
  url: "http://localhost",
  port: 3000,
  methods: ["GET", "POST"]
};
// 类型：{ url: string; port: number; methods: string[] }

// 加 as const
const config2 = {
  url: "http://localhost",
  port: 3000,
  methods: ["GET", "POST"]
} as const;
// 类型：{
//   readonly url: "http://localhost";
//   readonly port: 3000;
//   readonly methods: readonly ["GET", "POST"];
// }

// 应用：从对象提取联合类型
const COLORS = ["red", "green", "blue"] as const;
type Color = typeof COLORS[number];  // "red" | "green" | "blue"
```

---

### 18. TS 中函数参数的协变与逆变？

**答案：**

简单理解：
- **协变（Covariant）**：返回值是协变的（子类型可赋值给父类型）
- **逆变（Contravariant）**：参数是逆变的（父类型可赋值给子类型，反直觉）

```typescript
class Animal {}
class Dog extends Animal { bark() {} }

// 返回值协变：返回 Dog 的函数兼容返回 Animal 的函数类型
type AnimalFactory = () => Animal;
const dogFactory: AnimalFactory = (): Dog => new Dog();  // ✅

// 参数逆变：接受 Animal 的函数兼容接受 Dog 的函数类型
type DogHandler = (d: Dog) => void;
const animalHandler: DogHandler = (a: Animal) => {};  // ✅（开启 strictFunctionTypes）
```

**为什么参数是逆变？** 因为父类型能做的事情，子类型一定能做。接受 `Animal` 的函数能处理任何 `Dog`。

**配置项**：`strictFunctionTypes` 开启后强制函数参数逆变检查。

---

### 19. TS 怎样处理 JSON 数据类型？

**答案：**

```typescript
// 1. 安全的 JSON 类型定义
type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue };

// 2. 解析 JSON 时类型断言
const data = JSON.parse(text) as User;

// 3. 推荐：使用 zod / io-ts 进行运行时校验
import { z } from "zod";
const UserSchema = z.object({
  name: z.string(),
  age: z.number(),
});
type User = z.infer<typeof UserSchema>;
const user = UserSchema.parse(JSON.parse(text));  // 运行时校验+类型推断

// 4. 导入 JSON 文件
// tsconfig: "resolveJsonModule": true
import config from "./config.json";
```

---

### 20. 联合类型和交叉类型的区别？

**答案：**

```typescript
// 联合类型 |：A 或 B
type A = string | number;
let a: A = "hi";  // ✅
a = 42;           // ✅

// 交叉类型 &：A 且 B（合并所有属性）
interface HasName { name: string; }
interface HasAge { age: number; }
type Person = HasName & HasAge;
const p: Person = { name: "Alice", age: 25 };  // 必须同时具备

// 注意：基本类型交叉得到 never
type Impossible = string & number;  // never

// 联合类型的属性访问限制
type U = { a: string } | { b: number };
// u.a;  // ❌ 不一定有
// u.b;  // ❌ 不一定有

// 交叉类型可访问所有属性
type I = { a: string } & { b: number };
const i: I = { a: "x", b: 1 };
i.a; i.b;  // ✅
```

**记忆法**：联合是「**取值**的并集，**操作**的交集」，交叉是「**取值**的交集，**操作**的并集」。

---

### 21. 什么是非空断言操作符 `!`？

**答案：**

`!` 告诉编译器某个值**一定不为 null/undefined**，绕过空值检查。

```typescript
function getEl(id: string) {
  const el = document.getElementById(id);  // HTMLElement | null
  return el!;  // 强制断言为 HTMLElement
}

// 类字段：明确赋值断言
class User {
  name!: string;  // 告诉 TS：会在某处赋值，别报错
  
  init() { this.name = "Alice"; }
}
```

**注意：**
- 仅是编译期欺骗，运行时仍可能为 null
- 滥用 `!` 是反模式，应优先用类型守卫
- 可选链 `?.` 比 `!` 更安全

---

### 22. `Object`、`object`、`{}` 的区别？

**答案：**

```typescript
// Object（大写）：所有非 null/undefined 值（不推荐使用）
let a: Object = "hi";       // ✅
let b: Object = 42;         // ✅
let c: Object = {};         // ✅
// let d: Object = null;    // ❌（strict 模式）

// object（小写）：非原始类型（对象、数组、函数）
let e: object = {};         // ✅
let f: object = [];         // ✅
// let g: object = "hi";    // ❌

// {}（空对象类型）：除 null/undefined 外的所有类型
let h: {} = "hi";           // ✅
let i: {} = 42;             // ✅
// let j: {} = null;        // ❌
```

**最佳实践：**
- 描述任意对象用 `object`
- 描述任意非空值用 `{}`（但语义不清，慎用）
- 几乎不用 `Object`
- 描述具体对象用接口或 `Record<string, unknown>`

---

### 23. TS 中如何定义函数类型？

**答案：**

```typescript
// 1. 函数声明
function add(a: number, b: number): number { return a + b; }

// 2. 函数表达式（箭头函数）
const sub: (a: number, b: number) => number = (a, b) => a - b;

// 3. type 别名
type BinaryOp = (a: number, b: number) => number;
const mul: BinaryOp = (a, b) => a * b;

// 4. interface（call signature）
interface BinaryOp2 {
  (a: number, b: number): number;
}

// 5. 带属性的函数（混合类型）
interface Counter {
  (): number;       // 调用签名
  count: number;    // 属性
  reset(): void;
}

const counter = (() => counter.count++) as Counter;
counter.count = 0;
counter.reset = () => { counter.count = 0; };
```

---

### 24. TS 中的 `enum` 有什么坑？

**答案：**

```typescript
// 1. 数字枚举有反向映射（编译产物体积变大）
enum Direction { Up, Down }
console.log(Direction.Up);     // 0
console.log(Direction[0]);     // "Up" ←— 反向映射

// 2. 数字枚举类型不安全
function move(d: Direction) {}
move(99);  // ✅（不报错！）

// 3. 字符串枚举无反向映射，更安全
enum Status { Active = "active", Inactive = "inactive" }

// 4. const enum 编译时内联，无运行时对象
const enum Size { S = "s", M = "m" }
let x = Size.S;  // 编译为 let x = "s";

// 5. 推荐替代方案：as const 对象
const STATUS = {
  ACTIVE: "active",
  INACTIVE: "inactive",
} as const;
type Status = typeof STATUS[keyof typeof STATUS];
```

**为什么很多团队禁用 enum？**
- 数字枚举类型不安全
- 普通 enum 增加产物体积
- `const enum` 在 isolatedModules 模式下不兼容
- 不利于 tree-shaking

---

### 25. `strict` 模式下都开启了哪些检查？

**答案：**

`"strict": true` 等价于开启以下所有：

| 选项 | 作用 |
|------|------|
| `noImplicitAny` | 禁止隐式 any |
| `strictNullChecks` | null/undefined 必须显式声明 |
| `strictFunctionTypes` | 函数参数逆变检查 |
| `strictBindCallApply` | bind/call/apply 类型检查 |
| `strictPropertyInitialization` | 类属性必须初始化 |
| `noImplicitThis` | 禁止隐式 this 类型 |
| `alwaysStrict` | 始终启用 ES 严格模式 |
| `useUnknownInCatchVariables` | catch 变量为 unknown |

**强烈建议新项目开启 strict 模式**，可以避免大量潜在 bug。

---

### 26. `infer` 关键字的作用？

**答案：**

`infer` 用于**条件类型**中，从某个类型中"提取"子类型。

```typescript
// 1. 提取函数返回值类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type R1 = ReturnType<() => string>;  // string

// 2. 提取数组元素类型
type ElementOf<T> = T extends (infer E)[] ? E : never;
type E1 = ElementOf<number[]>;  // number

// 3. 提取 Promise 解析值
type Awaited<T> = T extends Promise<infer U> ? U : T;
type R2 = Awaited<Promise<string>>;  // string

// 4. 提取构造函数实例类型
type Instance<T> = T extends new (...args: any[]) => infer I ? I : never;
type R3 = Instance<typeof Date>;  // Date

// 5. 提取首尾元素
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;
```

**关键点**：`infer` 只能在 `extends` 子句中使用，相当于"在结构匹配时给某个位置起个名字"。

---

### 27. 什么是泛型？为什么需要它？

**答案：**

泛型是**类型的参数化**，让代码在保持类型安全的同时具备复用性。

```typescript
// 没有泛型：失去类型信息
function identity1(arg: any): any { return arg; }
const x = identity1("hello");  // x: any（丢失类型）

// 用泛型：保留类型
function identity<T>(arg: T): T { return arg; }
const y = identity("hello");   // y: string ✅
const z = identity(42);        // z: number ✅

// 泛型类
class Stack<T> {
  private items: T[] = [];
  push(item: T) { this.items.push(item); }
  pop(): T | undefined { return this.items.pop(); }
}

// 泛型约束
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// 泛型默认值
interface Response<T = any> { data: T; }
type R = Response;  // Response<any>
```

**应用场景**：容器类（Array、Map）、函数库（lodash）、API 响应封装、组件 props。

---

### 28. TS 中的"鸭子类型"是什么？

**答案：**

TS 采用**结构化类型系统**（Structural Typing），俗称"鸭子类型"——只要结构兼容就视为同一类型，不要求名义上的关系。

```typescript
interface Point { x: number; y: number; }

class MyPoint {
  constructor(public x: number, public y: number) {}
}

// 没有 implements，但结构兼容
const p: Point = new MyPoint(1, 2);  // ✅

// 函数参数也按结构匹配
function logPoint(p: Point) { console.log(p.x, p.y); }
logPoint({ x: 1, y: 2 });               // ✅
logPoint({ x: 1, y: 2, z: 3 });         // ⚠️ 字面量直接传会报错（多余属性检查）

const obj = { x: 1, y: 2, z: 3 };
logPoint(obj);                          // ✅ 通过变量传则可
```

**对比**：Java/C# 是名义类型系统，类型名相同才兼容。

---

### 29. 多余属性检查（Excess Property Checking）是什么？

**答案：**

当**对象字面量**直接赋值给已知类型时，TS 会额外检查是否有"多余属性"。

```typescript
interface Config { url: string; }

// 字面量直接传：触发多余属性检查
function load(c: Config) {}
// load({ url: "x", debug: true });  // ❌ 多余属性 debug

// 绕过方法 1：先赋给变量
const cfg = { url: "x", debug: true };
load(cfg);  // ✅（结构化类型）

// 绕过方法 2：as 断言
load({ url: "x", debug: true } as Config);  // ✅

// 绕过方法 3：声明索引签名
interface ConfigLoose { url: string; [key: string]: unknown; }
```

**设计目的**：捕获拼写错误（如把 `color` 写成 `colour`）。

---

### 30. `void` 在函数返回值和函数类型中的差异？

**答案：**

```typescript
// 函数声明：void = 不返回值（实际是 undefined）
function fn(): void { return; }
function fn2(): void { return undefined; }
// function fn3(): void { return 1; }  // ❌

// 函数类型注解：void 表示"忽略返回值"
type Callback = () => void;
const cb: Callback = () => 42;  // ✅！返回值会被忽略

// 实战陷阱
const arr: number[] = [];
arr.forEach((x): void => arr.push(x));  // forEach 的 void 类型允许返回任何值

// 因此：
[1, 2].forEach(x => console.log(x));  // ✅
```

**关键点**：作为返回值类型时，`void` 表示**调用者不应使用返回值**，但函数实现可以返回任何值。

---

## 二、进阶篇

### 31. 解释 TS 内置工具类型 `Partial`、`Required`、`Readonly` 的实现

**答案：**

```typescript
// Partial<T>：所有属性变可选
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// Required<T>：所有属性变必选（- 移除可选标记）
type Required<T> = {
  [P in keyof T]-?: T[P];
};

// Readonly<T>：所有属性变只读
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// 反向：可变（- 移除 readonly）
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};
```

**关键点**：`+` / `-` 修饰符用于**添加/移除** `?` 和 `readonly`。

---

### 32. 解释 `Pick` 和 `Omit` 的实现

**答案：**

```typescript
// Pick<T, K>：从 T 中选取 K 指定的属性
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Omit<T, K>：从 T 中排除 K 指定的属性
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

// 使用
interface User { id: number; name: string; email: string; password: string; }
type PublicUser = Omit<User, "password">;  // { id, name, email }
type Credentials = Pick<User, "email" | "password">;  // { email, password }
```

**注意**：`Omit` 中 `K extends keyof any` 而非 `K extends keyof T`，意味着**允许传入不存在的键**（TS 4.1 之前的设计）。

---

### 33. 解释 `Exclude`、`Extract`、`NonNullable`

**答案：**

```typescript
// Exclude<T, U>：从 T 中排除可赋值给 U 的部分
type Exclude<T, U> = T extends U ? never : T;
type A = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"

// Extract<T, U>：从 T 中提取可赋值给 U 的部分
type Extract<T, U> = T extends U ? T : never;
type B = Extract<string | number | boolean, string | number>;  // string | number

// NonNullable<T>：排除 null 和 undefined
type NonNullable<T> = T extends null | undefined ? never : T;
// 或：T & {}
type C = NonNullable<string | null | undefined>;  // string
```

**底层机制**：基于**分布式条件类型**——当条件类型作用于联合类型时，会自动展开每个成员。

---

### 34. 什么是分布式条件类型？

**答案：**

当条件类型 `T extends U ? X : Y` 中的 `T` 是**裸类型参数**且为联合类型时，会自动**分发**到每个成员。

```typescript
type ToArray<T> = T extends any ? T[] : never;
type R = ToArray<string | number>;
// 等价于 ToArray<string> | ToArray<number>
// = string[] | number[]

// 关闭分布式行为：用 [T] extends [U] 包裹
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;
type R2 = ToArrayNonDist<string | number>;  // (string | number)[]

// 应用：实现 Exclude
type MyExclude<T, U> = T extends U ? never : T;
type X = MyExclude<"a" | "b" | "c", "a">;
// 展开：("a" extends "a" ? never : "a") 
//      | ("b" extends "a" ? never : "b") 
//      | ("c" extends "a" ? never : "c")
// = never | "b" | "c" = "b" | "c"
```

**触发条件**：
1. 条件类型
2. T 是裸类型参数（不嵌套在元组、函数等结构中）
3. T 是联合类型

---

### 35. 解释 `ReturnType`、`Parameters`、`ConstructorParameters`

**答案：**

```typescript
// ReturnType：提取函数返回类型
type ReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R ? R : any;

// Parameters：提取函数参数类型（元组）
type Parameters<T extends (...args: any) => any> =
  T extends (...args: infer P) => any ? P : never;

// ConstructorParameters：提取构造函数参数
type ConstructorParameters<T extends abstract new (...args: any) => any> =
  T extends abstract new (...args: infer P) => any ? P : never;

// InstanceType：提取构造函数实例类型
type InstanceType<T extends abstract new (...args: any) => any> =
  T extends abstract new (...args: any) => infer R ? R : any;

// 使用示例
function greet(name: string, age: number): string { return `Hi ${name}`; }
type GreetReturn = ReturnType<typeof greet>;     // string
type GreetParams = Parameters<typeof greet>;     // [name: string, age: number]

class Animal { constructor(public name: string) {} }
type AnimalCtorParams = ConstructorParameters<typeof Animal>;  // [name: string]
type AnimalInstance = InstanceType<typeof Animal>;             // Animal
```

---

### 36. 怎样实现深层 Partial？

**答案：**

```typescript
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;

// 测试
interface User {
  name: string;
  address: {
    city: string;
    zip: string;
  };
  hobbies: string[];
}

type DeepPartialUser = DeepPartial<User>;
// {
//   name?: string;
//   address?: { city?: string; zip?: string };
//   hobbies?: (string | undefined)[];  // 注意数组也会被深度处理
// }

// 更精确的实现（区分数组、函数等）
type DeepPartial2<T> = {
  [K in keyof T]?: T[K] extends (...args: any[]) => any
    ? T[K]
    : T[K] extends Array<infer U>
    ? Array<DeepPartial2<U>>
    : T[K] extends object
    ? DeepPartial2<T[K]>
    : T[K];
};
```

---

### 37. `Record<K, T>` 的作用和实现？

**答案：**

```typescript
// 实现
type Record<K extends keyof any, T> = {
  [P in K]: T;
};

// 使用：构造对象类型
type PageInfo = Record<"home" | "about" | "contact", { title: string; url: string }>;
// {
//   home: { title: string; url: string };
//   about: { title: string; url: string };
//   contact: { title: string; url: string };
// }

// 替代索引签名（更类型安全）
type Dict = Record<string, number>;          // { [key: string]: number }
type StrictDict = Record<"a" | "b", number>; // 仅允许 a 和 b 两个键

// 实战：枚举值映射
const STATUS_LABELS: Record<Status, string> = {
  pending: "待处理",
  active: "进行中",
  done: "已完成",
};
```

---

### 38. 什么是映射类型？怎么自定义？

**答案：**

映射类型基于**已有类型**，遍历键名生成新类型。

```typescript
// 基本语法
type MyMap<T> = { [K in keyof T]: T[K] };

// 修饰符：?、readonly、+、-
type MyPartial<T> = { [K in keyof T]?: T[K] };
type MyRequired<T> = { [K in keyof T]-?: T[K] };
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };
type Mutable<T> = { -readonly [K in keyof T]: T[K] };

// 键名重映射（TS 4.1+）：as 子句
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User { name: string; age: number; }
type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number; }

// 过滤属性：键重映射 + never
type PickByValueType<T, V> = {
  [K in keyof T as T[K] extends V ? K : never]: T[K];
};
type StringProps = PickByValueType<{ a: string; b: number; c: string }, string>;
// { a: string; c: string }
```

---

### 39. 模板字面量类型有哪些应用？

**答案：**

```typescript
// 1. 拼接类型
type Greeting = `Hello, ${string}`;
let g: Greeting = "Hello, World";  // ✅

// 2. 联合类型组合
type Method = "GET" | "POST";
type Path = "/users" | "/posts";
type Endpoint = `${Method} ${Path}`;
// "GET /users" | "GET /posts" | "POST /users" | "POST /posts"

// 3. 内置工具：Uppercase、Lowercase、Capitalize、Uncapitalize
type Upper = Uppercase<"hello">;          // "HELLO"
type Cap = Capitalize<"hello world">;     // "Hello world"

// 4. 提取/解析字符串
type ExtractParam<S extends string> =
  S extends `${string}{${infer P}}${string}` ? P : never;
type P = ExtractParam<"/users/{id}/posts">;  // "id"

// 5. 路由参数推断
type RouteParams<S extends string> =
  S extends `${string}:${infer P}/${infer R}`
    ? { [K in P | keyof RouteParams<`/${R}`>]: string }
    : S extends `${string}:${infer P}`
    ? { [K in P]: string }
    : {};

type R = RouteParams<"/users/:id/posts/:postId">;
// { id: string; postId: string }
```

---

### 40. 怎样让一个类型与另一个类型互斥（XOR）？

**答案：**

```typescript
// 工具类型：XOR
type Without<T, U> = { [P in Exclude<keyof T, keyof U>]?: never };
type XOR<T, U> = (Without<T, U> & U) | (Without<U, T> & T);

// 应用：API 返回值要么 data 成功，要么 error 失败
type Result<T> = XOR<{ data: T }, { error: string }>;

const ok: Result<number> = { data: 42 };          // ✅
const err: Result<number> = { error: "fail" };   // ✅
// const both: Result<number> = { data: 42, error: "fail" };  // ❌
// const none: Result<number> = {};                            // ❌
```

---

### 41. 怎样在 TS 中实现"链式调用"的类型推断？

**答案：**

```typescript
class QueryBuilder<T = {}> {
  private data: T;
  
  constructor(data: T = {} as T) { this.data = data; }
  
  // 返回新的泛型类型，累加字段
  set<K extends string, V>(key: K, value: V): QueryBuilder<T & Record<K, V>> {
    return new QueryBuilder({ ...this.data, [key]: value } as T & Record<K, V>);
  }
  
  build(): T { return this.data; }
}

const result = new QueryBuilder()
  .set("name", "Alice")
  .set("age", 25)
  .set("active", true)
  .build();
// result 类型：{ name: string; age: number; active: boolean }
```

---

### 42. 什么是"协变位置"和"逆变位置"？

**答案：**

```typescript
// 函数类型 (T) => U 中：
// - T 是逆变位置（参数）
// - U 是协变位置（返回值）

// 提取协变位置类型 → 联合
type Cov<T> = T extends () => infer R ? R : never;
type C = Cov<(() => string) | (() => number)>;  // string | number

// 提取逆变位置类型 → 交叉（特殊行为！）
type Contra<T> = T extends (arg: infer A) => any ? A : never;
type X = Contra<((a: string) => void) | ((a: number) => void)>;
// string & number = never！

// 利用此特性实现「联合转交叉」
type UnionToIntersection<U> = 
  (U extends any ? (x: U) => void : never) extends (x: infer I) => void
    ? I
    : never;

type R = UnionToIntersection<{ a: 1 } | { b: 2 }>;
// { a: 1 } & { b: 2 }
```

---

### 43. `&` 和 `extends` 实现继承的区别？

**答案：**

```typescript
// interface extends：严格检查冲突
interface A { x: string; }
interface B extends A { x: number; }  // ❌ 冲突报错

// type &：会合并，冲突属性变为 never
type A2 = { x: string };
type B2 = A2 & { x: number };  // x: never（string & number = never）

// 函数重载行为
type Fn = ((x: string) => void) & ((x: number) => void);
const f: Fn = (x: string | number) => {};
f("a");  // ✅
f(1);    // ✅

// 实战：选择哪个？
// - 对象继承：interface extends（错误更早暴露）
// - 类型组合（如 Mixin）：type &
// - 联合类型：必须用 type
```

---

### 44. 如何处理动态键名的对象类型？

**答案：**

```typescript
// 1. 索引签名
interface DynamicMap { [key: string]: unknown; }

// 2. Record（推荐）
type Map1 = Record<string, number>;

// 3. 已知键的子集
type Keys = "id" | "name" | "email";
type StrictMap = Record<Keys, string>;

// 4. 模板字面量限定键名格式
type EnvKey = `REACT_APP_${string}`;
type EnvVars = Record<EnvKey, string>;

// 5. 严格但可扩展
interface BaseProps {
  id: string;
  name: string;
  [key: `data-${string}`]: string;  // 仅 data-* 前缀的额外属性
}
```

---

### 45. 怎样区分泛型参数中的可选与必选？

**答案：**

```typescript
// 提取必选键
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

// 提取可选键
type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

interface User {
  id: number;
  name: string;
  email?: string;
  age?: number;
}

type R = RequiredKeys<User>;  // "id" | "name"
type O = OptionalKeys<User>;  // "email" | "age"

// 原理：
// {} extends Pick<T, K>
// - K 必选时，Pick<T, K> = { K: type }，{} 不可赋值给它，结果 false
// - K 可选时，Pick<T, K> = { K?: type }，{} 可赋值，结果 true
```

---

### 46. TS 中的「协变返回类型」是什么？

**答案：**

子类覆写父类方法时，**返回值可以更具体**（协变），但**参数不能更具体**（必须逆变或不变）。

```typescript
class Animal {
  clone(): Animal { return new Animal(); }
}

class Dog extends Animal {
  override clone(): Dog { return new Dog(); }  // ✅ 返回值协变
}

// 参数逆变示例
class A {
  method(x: Dog): void {}
}
class B extends A {
  override method(x: Animal): void {}  // ✅ 参数变宽（逆变）
  // method(x: Bulldog): void  // ❌ 参数变窄（不安全）
}
```

---

### 47. 什么是抽象类？跟接口的区别？

**答案：**

```typescript
// 抽象类
abstract class Shape {
  abstract area(): number;  // 抽象方法（无实现）
  
  describe() { return `Area: ${this.area()}`; }  // 具体方法
  
  constructor(public name: string) {}  // 可有构造函数
}

// 接口
interface Shape2 {
  name: string;
  area(): number;
  // describe(): string;  // 只能定义签名
}
```

| 特性 | 抽象类 | 接口 |
|------|--------|------|
| 实现 | 可有具体方法 | 仅签名 |
| 字段 | 可有 | 只能是签名 |
| 构造函数 | 可有 | 不可有 |
| 多继承 | 单继承 | 多继承 |
| 访问修饰符 | 可有 | 默认 public |
| 编译产物 | 有运行时代码 | 无（被擦除） |

**选择**：需要复用代码逻辑时用抽象类，仅约束结构时用接口。

---

### 48. TS 怎么实现 Mixin？

**答案：**

```typescript
// 构造器类型
type Constructor<T = {}> = new (...args: any[]) => T;

// Mixin 函数：接收基类，返回扩展后的类
function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}

function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActive = false;
    activate() { this.isActive = true; }
  };
}

// 使用
class User {
  constructor(public name: string) {}
}

const TimestampedActivatableUser = Activatable(Timestamped(User));
const u = new TimestampedActivatableUser("Alice");
console.log(u.name, u.timestamp, u.isActive);
u.activate();
```

---

### 49. `unique symbol` 的用法？

**答案：**

```typescript
// 普通 symbol：所有 symbol 类型相同
const s1: symbol = Symbol("a");
const s2: symbol = Symbol("a");
// s1 !== s2 但类型相同

// unique symbol：每个值有独一无二的类型
const SECRET: unique symbol = Symbol("secret");
type SecretType = typeof SECRET;  // 独一无二

// 应用：作为对象键
const KEY: unique symbol = Symbol();
interface Container {
  [KEY]: string;
}
const c: Container = { [KEY]: "hello" };

// 限制：只能用于 const 或 readonly static 属性
class C {
  static readonly id: unique symbol = Symbol();
}
```

---

### 50. `never` 类型的应用场景？

**答案：**

```typescript
// 1. 永不返回的函数
function fail(): never { throw new Error(); }
function loop(): never { while (true) {} }

// 2. 穷尽性检查
type Shape = "circle" | "square";
function area(s: Shape) {
  switch (s) {
    case "circle": return 1;
    case "square": return 2;
    default:
      const _: never = s;  // 新增类型未处理时，此处会报错
      return _;
  }
}

// 3. 过滤联合类型
type NonString<T> = T extends string ? never : T;
type R = NonString<string | number | boolean>;  // number | boolean

// 4. 表示"不可能"
type Impossible = string & number;  // never

// 5. 类型层面的"删除"
type Without<T, K> = { [P in keyof T as P extends K ? never : P]: T[P] };
```

---

## 三、高级篇

### 51. TS 怎么定义递归类型？

**答案：**

```typescript
// 1. 树形结构
interface TreeNode<T> {
  value: T;
  children?: TreeNode<T>[];
}

// 2. JSON 类型
type JSONValue =
  | string | number | boolean | null
  | JSONValue[]
  | { [key: string]: JSONValue };

// 3. 链表
interface LinkedList<T> {
  value: T;
  next: LinkedList<T> | null;
}

// 4. 递归类型工具：DeepReadonly
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

// 5. 递归条件类型（注意深度限制）
type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;
type R = Flatten<number[][][]>;  // number
```

**注意**：TS 对递归深度有限制（通常 50 层），过深会报错。

---

### 52. 怎样实现一个类型安全的事件系统？

**答案：**

```typescript
// 事件映射
interface EventMap {
  click: { x: number; y: number };
  scroll: { offset: number };
  load: void;
}

class TypedEmitter<E> {
  private listeners: { [K in keyof E]?: Array<(payload: E[K]) => void> } = {};
  
  on<K extends keyof E>(event: K, fn: (payload: E[K]) => void) {
    (this.listeners[event] ??= []).push(fn);
  }
  
  emit<K extends keyof E>(event: K, payload: E[K]) {
    this.listeners[event]?.forEach(fn => fn(payload));
  }
}

const emitter = new TypedEmitter<EventMap>();
emitter.on("click", e => console.log(e.x, e.y));  // ✅ e 自动推断
emitter.emit("click", { x: 1, y: 2 });            // ✅
// emitter.emit("click", { x: 1 });                // ❌ 缺少 y
// emitter.emit("unknown", {});                    // ❌ 未知事件
```

---

### 53. 怎样实现 `OmitByValue`（按值类型过滤）？

**答案：**

```typescript
type OmitByValue<T, V> = {
  [K in keyof T as T[K] extends V ? never : K]: T[K];
};

interface User {
  id: number;
  name: string;
  age: number;
  email: string;
}

type StringProps = OmitByValue<User, number>;
// { name: string; email: string }
```

**原理**：键名重映射 `as` + `never` 实现过滤。

---

### 54. 怎样实现「函数柯里化」的类型？

**答案：**

```typescript
type Curry<F> = F extends (...args: infer A) => infer R
  ? A extends [infer First, ...infer Rest]
    ? Rest extends []
      ? (arg: First) => R
      : (arg: First) => Curry<(...args: Rest) => R>
    : R
  : never;

declare function curry<F extends (...args: any) => any>(fn: F): Curry<F>;

const add = (a: number, b: string, c: boolean) => "result";
const curried = curry(add);
const r = curried(1)("hello")(true);  // 类型推断完整
```

---

### 55. 怎样将联合类型转为元组？

**答案：**

```typescript
// 利用函数重载交叉的特性
type UnionToIntersection<U> =
  (U extends any ? (k: U) => void : never) extends (k: infer I) => void
    ? I
    : never;

type LastOfUnion<U> =
  UnionToIntersection<U extends any ? () => U : never> extends () => infer R
    ? R
    : never;

type UnionToTuple<U, L = LastOfUnion<U>> =
  [U] extends [never]
    ? []
    : [...UnionToTuple<Exclude<U, L>>, L];

type R = UnionToTuple<"a" | "b" | "c">;  // ["a", "b", "c"]
```

**警告**：联合类型本身无序，转换结果顺序依赖编译器实现，不要依赖此特性。

---

### 56. 实现 `IsEqual<A, B>` 判断两个类型是否完全相等

**答案：**

```typescript
// 利用条件类型对函数类型分配的特殊行为
type IsEqual<A, B> =
  (<T>() => T extends A ? 1 : 2) extends
  (<T>() => T extends B ? 1 : 2) ? true : false;

type T1 = IsEqual<string, string>;       // true
type T2 = IsEqual<string, number>;       // false
type T3 = IsEqual<{ a: 1 }, { a: 1 }>;   // true
type T4 = IsEqual<any, unknown>;         // false
type T5 = IsEqual<any, any>;             // true
```

**原理**：TS 比较函数类型时是结构相等，仅当 A 和 B 完全相同时，两个 `<T>() => ...` 才相同。

---

### 57. 实现一个简单的 `If` 类型

**答案：**

```typescript
type If<C extends boolean, T, F> = C extends true ? T : F;

type A = If<true, "yes", "no">;   // "yes"
type B = If<false, "yes", "no">;  // "no"

// 升级版：支持 truthy/falsy
type IfTruthy<C, T, F> = 
  [C] extends [never] ? F :
  C extends 0 | "" | false | null | undefined ? F :
  T;
```

---

### 58. 实现 `Trim` 字符串类型

**答案：**

```typescript
type Whitespace = " " | "\n" | "\t";

type TrimLeft<S extends string> =
  S extends `${Whitespace}${infer R}` ? TrimLeft<R> : S;

type TrimRight<S extends string> =
  S extends `${infer R}${Whitespace}` ? TrimRight<R> : S;

type Trim<S extends string> = TrimLeft<TrimRight<S>>;

type R1 = Trim<"  hello  ">;  // "hello"
type R2 = Trim<"\n\thi\n">;   // "hi"
```

---

### 59. 实现 `Replace`、`ReplaceAll`

**答案：**

```typescript
// 替换首次出现
type Replace<S extends string, From extends string, To extends string> =
  From extends ""
    ? S
    : S extends `${infer L}${From}${infer R}`
    ? `${L}${To}${R}`
    : S;

// 替换全部
type ReplaceAll<S extends string, From extends string, To extends string> =
  From extends ""
    ? S
    : S extends `${infer L}${From}${infer R}`
    ? `${L}${To}${ReplaceAll<R, From, To>}`
    : S;

type R1 = Replace<"foo bar foo", "foo", "baz">;     // "baz bar foo"
type R2 = ReplaceAll<"foo bar foo", "foo", "baz">;  // "baz bar baz"
```

---

### 60. 实现 `Split` 字符串拆分

**答案：**

```typescript
type Split<S extends string, D extends string> =
  string extends S
    ? string[]
    : S extends ""
    ? []
    : S extends `${infer L}${D}${infer R}`
    ? [L, ...Split<R, D>]
    : [S];

type R1 = Split<"a,b,c", ",">;       // ["a", "b", "c"]
type R2 = Split<"hello world", " ">; // ["hello", "world"]
type R3 = Split<"abc", "">;          // ["abc"]
```

---

### 61. 实现 `DeepRequired`

**答案：**

```typescript
type DeepRequired<T> = {
  [K in keyof T]-?: T[K] extends object
    ? T[K] extends Function
      ? T[K]
      : DeepRequired<T[K]>
    : T[K];
};

interface User {
  name?: string;
  address?: {
    city?: string;
    zip?: string;
  };
}

type R = DeepRequired<User>;
// {
//   name: string;
//   address: { city: string; zip: string };
// }
```

---

### 62. 实现 `Flatten` 数组扁平化

**答案：**

```typescript
// 一维扁平化
type Flat<T> = T extends Array<infer U>
  ? U extends Array<any>
    ? U[number]
    : U
  : T;

// 完全扁平化（递归）
type DeepFlat<T> = T extends Array<infer U>
  ? DeepFlat<U>
  : T;

type R1 = Flat<[1, [2, 3], [4, [5]]]>;     // 1 | 2 | 3 | 4 | [5]
type R2 = DeepFlat<[1, [2, [3, [4]]]]>;    // 1 | 2 | 3 | 4
```

---

### 63. TS 4.9 引入的 `satisfies` 操作符是什么？

**答案：**

`satisfies` 在**保留具体类型推断**的同时**校验类型约束**。

```typescript
type Colors = "red" | "green" | "blue";
type RGB = [number, number, number];

// 1. 用类型注解：丢失具体类型
const palette1: Record<Colors, string | RGB> = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
};
palette1.green.toUpperCase();  // ❌ green 类型为 string | RGB

// 2. 用 satisfies：保留具体推断
const palette2 = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
} satisfies Record<Colors, string | RGB>;

palette2.green.toUpperCase();  // ✅ green 推断为 string
palette2.red[0];                // ✅ red 推断为 number[]
// palette2.purple              // ❌ 类型外的 key 报错
```

**对比：**
- `as`：单向断言，可能错误
- `:` 类型注解：失去具体类型
- `satisfies`：双向校验 + 保留推断 ✨

---

### 64. 怎样精确控制函数 `this` 的类型？

**答案：**

```typescript
// 1. this 参数（伪参数，编译时擦除）
interface Button {
  text: string;
  onClick(this: Button): void;  // 限定 this 类型
}

const btn: Button = {
  text: "Submit",
  onClick() { console.log(this.text); }
};

// 2. ThisType<T>：上下文 this 类型
interface Methods {
  greet(): string;
}
interface Context {
  name: string;
}

type Bound = Methods & ThisType<Context>;

const obj: Bound = {
  greet() { return `Hi ${this.name}`; }  // this 推断为 Context
};
```

---

### 65. 模块声明合并的应用？

**答案：**

```typescript
// 1. 扩展全局对象
declare global {
  interface Window {
    APP: { version: string };
  }
}
window.APP = { version: "1.0" };

// 2. 扩展第三方库类型
import "express";
declare module "express" {
  interface Request {
    user?: { id: string };
  }
}

// 3. 扩展 React JSX
declare namespace JSX {
  interface IntrinsicElements {
    "my-button": { label: string };
  }
}
```

---

## 四、类型体操篇

### 66. 实现 `MyPick<T, K>`

```typescript
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

---

### 67. 实现 `MyReadonly2<T, K>`（仅指定属性变只读）

```typescript
type MyReadonly2<T, K extends keyof T = keyof T> = 
  Omit<T, K> & { readonly [P in K]: T[P] };

interface Todo { title: string; done: boolean; description: string; }
type R = MyReadonly2<Todo, "title" | "description">;
// { done: boolean; readonly title: string; readonly description: string }
```

---

### 68. 实现 `TupleToObject<T>`

```typescript
type TupleToObject<T extends readonly (string | number | symbol)[]> = {
  [K in T[number]]: K;
};

const tuple = ["tesla", "model 3", "model x"] as const;
type R = TupleToObject<typeof tuple>;
// { tesla: "tesla"; "model 3": "model 3"; "model x": "model x" }
```

---

### 69. 实现 `First<T>` 获取数组首元素类型

```typescript
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;
// 或 type First<T extends any[]> = T["length"] extends 0 ? never : T[0];

type R1 = First<[1, 2, 3]>;  // 1
type R2 = First<[]>;         // never
```

---

### 70. 实现 `Length<T>` 获取元组长度

```typescript
type Length<T extends readonly any[]> = T["length"];

type R = Length<[1, 2, 3, 4]>;  // 4
```

---

### 71. 实现 `MyAwaited<T>`

```typescript
type MyAwaited<T> = T extends Promise<infer U>
  ? U extends Promise<any>
    ? MyAwaited<U>  // 递归处理嵌套 Promise
    : U
  : never;

type R = MyAwaited<Promise<Promise<Promise<string>>>>;  // string
```

---

### 72. 实现 `If<C, T, F>`

```typescript
type If<C extends boolean, T, F> = C extends true ? T : F;
```

---

### 73. 实现 `Concat<T, U>` 元组拼接

```typescript
type Concat<T extends any[], U extends any[]> = [...T, ...U];

type R = Concat<[1, 2], [3, 4]>;  // [1, 2, 3, 4]
```

---

### 74. 实现 `Includes<T, U>`

```typescript
type IsEqual<A, B> =
  (<T>() => T extends A ? 1 : 2) extends 
  (<T>() => T extends B ? 1 : 2) ? true : false;

type Includes<T extends any[], U> = T extends [infer F, ...infer R]
  ? IsEqual<F, U> extends true
    ? true
    : Includes<R, U>
  : false;

type R1 = Includes<[1, 2, 3], 2>;  // true
type R2 = Includes<[1, 2, 3], 4>;  // false
```

---

### 75. 实现 `Push<T, V>` 和 `Pop<T>`

```typescript
type Push<T extends any[], V> = [...T, V];
type Pop<T extends any[]> = T extends [...infer R, any] ? R : never;
type Shift<T extends any[]> = T extends [any, ...infer R] ? R : never;
type Unshift<T extends any[], V> = [V, ...T];

type R1 = Push<[1, 2], 3>;     // [1, 2, 3]
type R2 = Pop<[1, 2, 3]>;       // [1, 2]
type R3 = Shift<[1, 2, 3]>;     // [2, 3]
type R4 = Unshift<[2, 3], 1>;   // [1, 2, 3]
```

---

## 五、工程化与原理篇

### 76. `tsconfig.json` 中常用配置项有哪些？

**答案：**

```jsonc
{
  "compilerOptions": {
    /* 基础 */
    "target": "ES2020",              // 编译目标版本
    "module": "ESNext",              // 模块系统
    "lib": ["ES2020", "DOM"],        // 内置类型库
    "outDir": "./dist",              // 输出目录
    "rootDir": "./src",              // 源码目录
    
    /* 严格性 */
    "strict": true,                  // 开启所有严格检查
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true,          // 未使用的局部变量
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    
    /* 模块解析 */
    "moduleResolution": "node",      // 或 "bundler"（TS 5.0+）
    "esModuleInterop": true,         // import 与 CommonJS 互操作
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,       // 允许 import json
    "baseUrl": "./",
    "paths": { "@/*": ["src/*"] },   // 路径别名
    
    /* 输出控制 */
    "declaration": true,             // 生成 .d.ts
    "declarationMap": true,          // 类型声明的 source map
    "sourceMap": true,
    "removeComments": true,
    "isolatedModules": true,         // 每个文件作为单独模块（Babel 兼容）
    
    /* JSX */
    "jsx": "react-jsx",              // React 17+
    
    /* 性能 */
    "skipLibCheck": true,            // 跳过 .d.ts 检查
    "incremental": true              // 增量编译
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

---

### 77. `.d.ts` 文件的作用？怎么编写？

**答案：**

`.d.ts` 是**声明文件**，仅包含类型信息，不含实现，用于：
1. 为 JS 库提供类型声明
2. 全局类型扩展
3. 模块类型声明

```typescript
// global.d.ts - 全局声明
declare const VERSION: string;
declare function gtag(...args: any[]): void;

// 模块声明
declare module "lodash-es" {
  export function debounce<T>(fn: T, wait: number): T;
}

// 通配符模块（资源文件）
declare module "*.png" {
  const src: string;
  export default src;
}

declare module "*.css" {
  const classes: Record<string, string>;
  export default classes;
}

// 扩展全局接口
declare global {
  interface Window {
    __APP_CONFIG__: { env: string };
  }
}
export {};  // 让此文件成为模块
```

**查找规则**：
- 同名 `.d.ts` 自动关联（如 `foo.js` + `foo.d.ts`）
- `node_modules/@types/<pkg>` 自动加载
- `tsconfig.types` 字段指定

---

### 78. 怎样为没有类型的第三方库添加类型？

**答案：**

```typescript
// 方案 1：安装社区维护的类型
// npm install -D @types/lodash

// 方案 2：自己写声明文件
// types/some-lib/index.d.ts
declare module "some-lib" {
  export function doSomething(input: string): number;
  export interface Config { url: string; }
  const _default: { version: string };
  export default _default;
}

// tsconfig.json 中包含
{
  "compilerOptions": {
    "typeRoots": ["./types", "./node_modules/@types"]
  }
}

// 方案 3：临时方案（不推荐）
declare module "some-lib";  // 整个模块为 any
```

---

### 79. TS 编译过程是怎样的？

**答案：**

```
源码 → 词法分析（Scanner）→ Token 流
   ↓
语法分析（Parser）→ AST
   ↓
绑定（Binder）→ Symbol 表（建立作用域）
   ↓
类型检查（Checker）→ 类型推断与校验
   ↓
转换（Transformer）→ 目标 ES 版本 AST
   ↓
发射（Emitter）→ JavaScript 代码 + .d.ts
```

**关键点：**
- 类型检查与代码生成**解耦**（即使有类型错误，也能输出 JS）
- TS 编译器**不做** Polyfill（需 Babel/core-js）
- 类型信息在编译时被完全擦除

---

### 80. TypeScript 与 Babel 的差异？为什么常常一起使用？

**答案：**

| 维度 | TypeScript（tsc） | Babel |
|------|-------------------|-------|
| 类型检查 | ✅ | ❌ |
| 代码降级 | ✅ | ✅ |
| Polyfill | ❌ | ✅（core-js） |
| 编译速度 | 较慢 | 快 |
| 单文件编译 | ❌（需项目级） | ✅ |
| 实验语法 | 较保守 | 激进 |

**常见组合**：
- `Babel`（@babel/preset-typescript）负责**编译**（速度快、支持 polyfill）
- `tsc --noEmit` 单独负责**类型检查**

```jsonc
// 仅类型检查不输出
{ "compilerOptions": { "noEmit": true, "isolatedModules": true } }
```

---

### 81. 为什么开启 `isolatedModules`？

**答案：**

`isolatedModules: true` 限制每个文件能被**独立编译**（如 Babel、esbuild 这种单文件编译器），强制：

```typescript
// ❌ 必须用 export type 
import { SomeType } from "./types";  // 若 SomeType 仅是类型则报错
import type { SomeType } from "./types";  // ✅

// ❌ 不允许跨模块的 const enum
import { MyEnum } from "./enums";  // const enum 不可被孤立编译

// ❌ 必须有显式 export
namespace Foo { export const x = 1; }  // 仅命名空间的文件需 export {}
```

**好处**：与 Babel/esbuild/swc 兼容，编译速度快。

---

### 82. 怎样实现项目引用（Project References）？

**答案：**

用于 monorepo 或大型项目，将代码拆为多个子项目，独立编译。

```jsonc
// 根 tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./packages/core" },
    { "path": "./packages/utils" }
  ]
}

// packages/utils/tsconfig.json
{
  "compilerOptions": {
    "composite": true,           // 启用项目引用模式（必须）
    "declaration": true,
    "outDir": "./dist"
  }
}

// packages/core/tsconfig.json
{
  "compilerOptions": { "composite": true },
  "references": [{ "path": "../utils" }]
}
```

```bash
# 构建命令（增量构建）
tsc --build
tsc -b --watch
tsc -b --clean
```

**优势**：增量编译、并行构建、明确依赖关系。

---

### 83. TypeScript 中 `import type` 与 `import` 的区别？

**答案：**

```typescript
// 普通 import：值 + 类型
import { Foo } from "./foo";

// import type：仅类型，编译时被完全擦除
import type { Foo } from "./foo";

// 混合写法（TS 4.5+）
import { type SomeType, someValue } from "./foo";
```

**作用：**
- 明确区分类型与值的导入
- 避免不必要的运行时引入（防止 tree-shaking 失效）
- `isolatedModules` 模式下必须使用

**配置**：`"verbatimModuleSyntax": true` 强制使用显式类型导入。

---

### 84. 为什么 TS 不支持运行时类型检查？

**答案：**

TypeScript 的核心设计原则：**类型在编译后被完全擦除**，不引入运行时开销。

**原因：**
1. 性能：运行时类型检查会拖慢应用
2. 兼容：编译输出是纯 JS
3. 哲学：TS 是 JS 的"类型层"，不改变运行行为

**解决方案：运行时校验库**
```typescript
// zod
import { z } from "zod";
const UserSchema = z.object({
  name: z.string(),
  age: z.number().min(0),
});
type User = z.infer<typeof UserSchema>;
const user = UserSchema.parse(data);  // 运行时校验

// 其他选择：io-ts、yup、joi、ajv（JSON Schema）、class-validator
```

---

### 85. 在 React/Vue 中使用 TS 的常见 Tips？

**答案：**

**React：**
```typescript
// 函数组件
interface Props { name: string; }
const Hello: React.FC<Props> = ({ name }) => <div>{name}</div>;
// 推荐：直接函数式（避免 React.FC 隐式 children）
const Hello2 = ({ name }: Props) => <div>{name}</div>;

// 事件类型
const onClick = (e: React.MouseEvent<HTMLButtonElement>) => {};
const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {};

// useState 泛型
const [user, setUser] = useState<User | null>(null);

// useRef
const inputRef = useRef<HTMLInputElement>(null);

// 自定义 hook 返回元组
function useToggle(): [boolean, () => void] {
  const [v, setV] = useState(false);
  return [v, () => setV(p => !p)] as const;
}
```

**Vue 3：**
```typescript
// defineProps + defineEmits
const props = defineProps<{
  msg: string;
  count?: number;
}>();

const emit = defineEmits<{
  (e: "update", value: string): void;
  (e: "close"): void;
}>();

// ref 类型
const count = ref<number>(0);
const list = ref<User[]>([]);

// 组件类型
import type { Component } from "vue";
const MyComp: Component = ...;
```

---

## 学习建议

1. **优先级**：基础类型 > interface/type > 泛型 > 工具类型 > 高级技巧
2. **实战驱动**：在项目中遇到类型问题，再针对性学习
3. **常练习**：[type-challenges](https://github.com/type-challenges/type-challenges) 类型体操
4. **看源码**：阅读 React、Vue、Express 等知名库的类型声明
5. **保持更新**：TS 每季度发版，新版本常带来革命性特性（如 `satisfies`、`const T extends`）

---

> 💡 **面试 Tips**：
> - 基础题（1-30）必须答得流利，是底线
> - 进阶题（31-50）能答出原理（如分布式条件类型、infer）会加分
> - 类型体操不必死记，理解思路即可
> - 工程化部分（76-85）大型团队/高级岗位重点考察
> - 主动结合自己项目讲述「踩坑经历」是最佳加分项
