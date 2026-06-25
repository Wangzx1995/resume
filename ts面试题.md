# TypeScript 面试题（含隐藏答案 · 详细版）

共 **20 题**，涵盖基础类型、接口与类、泛型、高级类型、类型推断与守卫、编译配置与工程化。点击「查看答案」即可展开，每题都包含**原理 / 对比 / 代码示例 / 坑点 / 面试加分点**。

---

## 一、基础类型与核心概念

### 1. TypeScript 相比 JavaScript 有什么核心优势？

<details>
<summary>查看答案</summary>

**1）静态类型检查**

- 在编译期发现类型错误，避免运行时崩溃
- IDE 支持更强大的自动补全、跳转、重构

```ts
function add(a: number, b: number) {
  return a + b;
}
add(1, "2"); // 编译报错：Argument of type 'string' is not assignable to 'number'
```

**2）更好的可读性与可维护性**

- 类型即文档，函数签名直接说明输入输出
- 大型项目下重构更安全

**3）先进的语言特性**

- 泛型、枚举、装饰器、接口、抽象类、命名空间
- 这些特性在 ES 标准中尚未完全落地或表达力不足

**4）工程化支持**

- `tsconfig.json` 统一项目配置
- 与 ESLint、Babel、Vite、Webpack 深度集成

**5）团队协作**

- 接口契约明确，前后端联调更顺畅
- 通过类型约束减少沟通成本

**TS 不是 JS 的替代品，而是超集**

- 所有 JS 代码都是合法 TS（默认配置下）
- TS 最终编译为 JS 运行

**面试加分点**：能说出 TS 的「类型只在编译期存在」，运行时是擦除的（Type Erasure）；能区分静态类型与运行时校验的边界。

</details>

---

### 2. `any`、`unknown`、`never`、`void` 有什么区别？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 类型      | 含义                   | 可赋值给任意类型？ | 可被任意类型赋值？ | 常见场景                    |
| --------- | ---------------------- | ------------------ | ------------------ | --------------------------- |
| `any`     | 任意类型，关闭类型检查 | ✅                 | ✅                 | 兼容老代码、临时绕过        |
| `unknown` | 未知类型，安全的 any   | ❌                 | ✅                 | 不确定类型的值              |
| `never`   | 永不发生的类型         | ✅                 | ❌                 | 抛错函数、 exhaustive check |
| `void`    | 无返回值               | ❌                 | ❌                 | 函数没有 return             |

**1）any —— 关闭类型检查**

```ts
let a: any = 1;
a = "hello";
a.toFixed(); // 不报错，但运行可能崩溃
```

**2）unknown —— 安全的 any**

```ts
let u: unknown = 1;

u.toFixed(); // ❌ 报错：Object is of type 'unknown'

// ✅ 必须先做类型收窄
if (typeof u === "number") {
  u.toFixed();
}
```

**3）never —— 永不存在的值**

```ts
function throwError(msg: string): never {
  throw new Error(msg);
}

function exhaustiveCheck(x: "a" | "b") {
  switch (x) {
    case "a":
      return 1;
    case "b":
      return 2;
    default:
      const _exhaustive: never = x; // 若有新增分支未处理，会报错
      return _exhaustive;
  }
}
```

**4）void —— 无显式返回值**

```ts
function log(msg: string): void {
  console.log(msg);
}
```

**面试加分点**：能解释 `unknown` 是 TS 3.0 引入的，设计目标就是替代不安全的 `any`；`never` 是所有类型的子类型，可用于完整性检查。

</details>

---

### 3. `interface` 和 `type` 有什么区别？什么时候用哪个？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 特性               | `interface`             | `type`          |
| ------------------ | ----------------------- | --------------- |
| 扩展方式           | `extends`               | `&` 交叉类型    |
| 同名声明           | ✅ 自动合并（声明合并） | ❌ 重复声明报错 |
| 描述对象形状       | 最自然                  | 也可以          |
| 联合/元组/基础类型 | ❌ 不行                 | ✅ 可以         |
| 性能（大量时）     | 稍好                    | 稍差            |
| 实现类             | ✅ `implements`         | ✅ 也可以       |

**interface 示例：**

```ts
interface User {
  name: string;
}
interface User {
  age: number;
}
// 合并后：{ name: string; age: number }

interface Admin extends User {
  role: string;
}
```

**type 示例：**

```ts
type User = {
  name: string;
};

type ID = string | number;
type Point = [number, number];

type Admin = User & { role: string };
```

**选择建议：**

| 场景                     | 推荐        |
| ------------------------ | ----------- |
| 定义对象/类的结构        | `interface` |
| 需要声明合并             | `interface` |
| 联合类型、元组、映射类型 | `type`      |
| 复杂类型运算             | `type`      |

**常见坑点：**

```ts
// ❌ type 同名重复声明报错
type A = { x: number };
type A = { y: number }; // Error: Duplicate identifier 'A'

// ✅ interface 同名自动合并
interface A {
  x: number;
}
interface A {
  y: number;
}
```

**面试加分点**：能说出官方推荐优先用 `interface`，复杂类型再用 `type`；能解释声明合并对扩展第三方库类型非常有用。

</details>

---

### 4. TypeScript 中的基本类型有哪些？`null` 和 `undefined` 有什么区别？

<details>
<summary>查看答案</summary>

**基本类型：**

```ts
let str: string = "hello";
let num: number = 100;
let bool: boolean = true;
let n: null = null;
let u: undefined = undefined;
let sym: symbol = Symbol();
let big: bigint = 100n;
```

**null 与 undefined 区别：**

| 维度     | `null`           | `undefined`              |
| -------- | ---------------- | ------------------------ |
| 语义     | 显式表示「无值」 | 表示「未定义」           |
| 默认值   | 不会自动赋值     | 变量未初始化时的默认值   |
| 可选参数 | 较少直接使用     | 函数可选参数默认值场景多 |

```ts
let a; // a 为 undefined
let b = null; // b 显式为空

function greet(name?: string) {
  console.log(name); // 不传时为 undefined
}
```

**严格空值检查：**

开启 `strictNullChecks` 后，`null` 和 `undefined` 不能随意赋值给其他类型。

```ts
let name: string = null; // ❌ strictNullChecks 下报错
let name2: string | null = null; // ✅
```

**面试加分点**：能解释 `strictNullChecks` 是 TS 最重要的严格模式选项之一，开启后可大幅减少空指针错误。

</details>

---

### 5. 什么是类型推断？TypeScript 在什么情况下会推断类型？

<details>
<summary>查看答案</summary>

**类型推断**：TS 在没有显式注解时，根据上下文自动推导出类型。

**1）变量初始化推断**

```ts
let a = 1; // 推断为 number
let b = "hello"; // 推断为 string
let c = [1, 2, 3]; // 推断为 number[]
```

**2）函数返回值推断**

```ts
function add(a: number, b: number) {
  return a + b; // 推断返回 number
}
```

**3）上下文类型推断（Contextual Typing）**

```ts
window.onkeydown = (event) => {
  // event 被推断为 KeyboardEvent
  console.log(event.key);
};
```

**4）类型参数推断**

```ts
function identity<T>(arg: T): T {
  return arg;
}
identity("hello"); // T 被推断为 "hello"（字面量类型）
identity<string>("hello"); // 显式指定
```

**何时显式注解？**

```ts
// 需要更宽类型时
let status: "active" | "inactive" = "active";

// 函数参数必须注解
function greet(name: string) {}

// 复杂对象提高可读性
interface User {
  name: string;
}
const user: User = { name: "tom" };
```

**面试加分点**：能区分「最佳公共类型推断」和「上下文类型推断」；知道 `const` 声明时字面量推断更窄。

</details>

---

## 二、泛型

### 6. 什么是泛型？有什么实际应用场景？

<details>
<summary>查看答案</summary>

**泛型**：在定义函数、接口、类时不指定具体类型，而是使用类型参数，使用时再传入具体类型。

```ts
function identity<T>(arg: T): T {
  return arg;
}

identity<number>(100);
identity<string>("hello");
identity(100); // 自动推断
```

**实际应用场景：**

**1）通用 API 请求函数**

```ts
async function request<T>(url: string): Promise<T> {
  const res = await fetch(url);
  return res.json() as Promise<T>;
}

const user = await request<User>("/api/user");
```

**2）通用工具函数**

```ts
function pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
  const result = {} as Pick<T, K>;
  keys.forEach((k) => (result[k] = obj[k]));
  return result;
}

pick({ a: 1, b: 2, c: 3 }, ["a", "b"]); // { a: number; b: number }
```

**3）React 组件 props**

```ts
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return <>{items.map(renderItem)}</>;
}
```

**4）数据结构**

```ts
class Stack<T> {
  private items: T[] = [];
  push(item: T) {
    this.items.push(item);
  }
  pop(): T | undefined {
    return this.items.pop();
  }
}
```

**面试加分点**：能解释泛型实现了「类型级别的参数化」，让代码既灵活又安全；能说出泛型约束 `extends` 的用途。

</details>

---

### 7. 泛型约束 `extends` 和默认类型 `=` 怎么用？

<details>
<summary>查看答案</summary>

**1）泛型约束**

限制泛型参数必须满足某种结构。

```ts
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength("hello"); // ✅ string 有 length
logLength([1, 2]); // ✅ 数组有 length
logLength(100); // ❌ number 没有 length
```

**2）多类型参数约束**

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

getProperty({ a: 1, b: "x" }, "a"); // number
```

**3）默认类型**

```ts
interface ResponseData<T = any> {
  code: number;
  data: T;
  msg: string;
}

const res: ResponseData = { code: 0, data: {}, msg: "" }; // T 默认为 any
const res2: ResponseData<User> = { code: 0, data: user, msg: "" };
```

**4）约束 + 默认组合**

```ts
function createArray<T = string>(length: number, value: T): T[] {
  return Array(length).fill(value);
}
```

**面试加分点**：能解释 `keyof T` 拿到对象类型的键联合类型，`T[K]` 是索引访问类型；能说明泛型约束让泛型不再「为所欲为」。

</details>

---

## 三、高级类型

### 8. 常用内置工具类型有哪些？它们分别做了什么？

<details>
<summary>查看答案</summary>

**常用工具类型表：**

| 工具类型         | 作用                          | 示例结果             |
| ---------------- | ----------------------------- | -------------------- |
| `Partial<T>`     | 所有属性变为可选              | `{ a?: number }`     |
| `Required<T>`    | 所有属性变为必选              | `{ a: number }`      |
| `Readonly<T>`    | 所有属性变为只读              | `readonly a: number` |
| `Pick<T, K>`     | 从 T 中选取 K 属性            | 子集类型             |
| `Omit<T, K>`     | 从 T 中排除 K 属性            | 剩余类型             |
| `Record<K, T>`   | 构造键为 K、值为 T 的对象类型 | `{ [k: string]: T }` |
| `Exclude<T, U>`  | 从 T 中排除可赋值给 U 的类型  | 差集                 |
| `Extract<T, U>`  | 从 T 中提取可赋值给 U 的类型  | 交集                 |
| `NonNullable<T>` | 排除 null/undefined           | 非空类型             |
| `ReturnType<T>`  | 获取函数返回类型              | 函数的 return 类型   |
| `Parameters<T>`  | 获取函数参数元组类型          | `[arg1: A, arg2: B]` |

**代码示例：**

```ts
interface User {
  id: number;
  name: string;
  age: number;
}

type UserUpdate = Partial<User>; // { id?: number; name?: string; age?: number; }
type UserPreview = Pick<User, "id" | "name">; // { id: number; name: string; }
type UserWithoutAge = Omit<User, "age">; // { id: number; name: string; }

type StatusMap = Record<string, string>; // { [k: string]: string }

type T1 = Exclude<"a" | "b" | "c", "a">; // "b" | "c"
type T2 = Extract<"a" | "b" | "c", "a" | "d">; // "a"
```

**面试加分点**：能手写实现 `Pick`、`Omit`、`Partial`、`Readonly` 等工具类型；理解它们都是基于映射类型实现的。

</details>

---

### 9. 什么是映射类型（Mapped Types）？如何手写实现 Partial？

<details>
<summary>查看答案</summary>

**映射类型**：遍历已有类型的每个属性，生成新类型。

**手写 `Partial<T>`：**

```ts
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

interface User {
  name: string;
  age: number;
}

type PartialUser = MyPartial<User>;
// { name?: string; age?: number; }
```

**关键语法：**

- `keyof T`：获取 T 的所有键
- `in`：遍历键联合类型
- `T[P]`：索引访问类型
- `?`：将属性变为可选

**手写 `Readonly<T>`：**

```ts
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};
```

**手写 `Pick<T, K>`：**

```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};
```

**添加修饰符：**

```ts
// 移除可选
[P in keyof T]-?: T[P]

// 移除 readonly
-readonly [P in keyof T]: T[P]
```

**面试加分点**：能解释映射类型是 TS 类型系统的核心能力之一；`keyof` + `in` + `T[P]` 的组合能实现大量工具类型。

</details>

---

### 10. 条件类型、infer 和分布式条件类型是什么？

<details>
<summary>查看答案</summary>

**1）条件类型**

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<"hello">; // true
type B = IsString<123>; // false
```

**2）infer 推断**

```ts
// 提取数组元素类型
type ElementType<T> = T extends (infer E)[] ? E : never;

type T1 = ElementType<number[]>; // number
type T2 = ElementType<string>; // never

// 提取函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

**3）分布式条件类型**

当条件类型中的泛型参数是联合类型时，会分发到每个成员上。

```ts
type ToArray<T> = T extends any ? T[] : never;

type A = ToArray<string | number>;
// string[] | number[]
// 不是 (string | number)[]
```

**阻止分布：**

```ts
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type A = ToArrayNonDist<string | number>;
// (string | number)[]
```

**面试加分点**：能解释 `infer` 只能在 `extends` 条件类型的 true 分支中使用；分布式条件类型是 TS 实现很多高级工具类型的基础。

</details>

---

## 四、类型推断、断言与守卫

### 11. 类型断言（Type Assertion）和类型守卫（Type Guard）有什么区别？

<details>
<summary>查看答案</summary>

**类型断言**：告诉编译器「我知道这个值是什么类型」，没有运行时检查。

```ts
const el = document.getElementById("app") as HTMLDivElement;
const input = <HTMLInputElement>document.getElementById("name"); // JSX 中不推荐
```

**类型守卫**：在运行时检查类型，帮助 TS 收窄类型。

```ts
function process(value: string | number) {
  if (typeof value === "string") {
    value.toUpperCase(); // ✅ 在此分支 value 为 string
  } else {
    value.toFixed(); // ✅ 在此分支 value 为 number
  }
}
```

**常见类型守卫方式：**

| 方式         | 示例                        |
| ------------ | --------------------------- |
| `typeof`     | `typeof x === 'string'`     |
| `instanceof` | `x instanceof Date`         |
| `in`         | `'name' in x`               |
| 字面量判断   | `x.type === 'success'`      |
| 自定义守卫   | `isFish(pet): pet is Fish`  |
| 可区分联合   | `interface A { kind: 'a' }` |

**自定义类型守卫：**

```ts
interface Fish {
  swim: () => void;
}
interface Bird {
  fly: () => void;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim();
  } else {
    pet.fly();
  }
}
```

**面试加分点**：能强调类型断言是「编译期欺骗」，滥用会失去类型安全；类型守卫是更安全的做法，应优先使用。

</details>

---

### 12. 什么是可区分联合（Discriminated Unions）？

<details>
<summary>查看答案</summary>

**可区分联合**：通过共同的可区分字段（tag），让联合类型的分支可以被类型守卫精确收窄。

```ts
interface Square {
  kind: "square";
  size: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Circle {
  kind: "circle";
  radius: number;
}

type Shape = Square | Rectangle | Circle;

function area(shape: Shape): number {
  switch (shape.kind) {
    case "square":
      return shape.size ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "circle":
      return Math.PI * shape.radius ** 2;
    default:
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

**优势：**

- 每个分支自动收窄到具体类型
- 结合 `never` 可检查是否处理了所有分支
- 比可选属性 + `if/else` 更安全

**面试加分点**：能说出 Redux action、React 组件状态机等场景非常适合用可区分联合；能提到 `kind` 字段就是 discriminant。

</details>

---

## 五、类与接口

### 13. TypeScript 中类的访问修饰符有哪些？抽象类是什么？

<details>
<summary>查看答案</summary>

**访问修饰符：**

| 修饰符      | 含义         | 可访问范围       |
| ----------- | ------------ | ---------------- |
| `public`    | 公开（默认） | 任意位置         |
| `private`   | 私有         | 类内部           |
| `protected` | 受保护       | 类内部 + 子类    |
| `readonly`  | 只读         | 初始化后不可修改 |

```ts
class Animal {
  public name: string;
  private age: number;
  protected species: string;

  constructor(name: string, age: number, species: string) {
    this.name = name;
    this.age = age;
    this.species = species;
  }
}

class Dog extends Animal {
  bark() {
    console.log(this.species); // ✅ protected 子类可访问
    // console.log(this.age);    // ❌ private 子类不可访问
  }
}
```

**参数属性简写：**

```ts
class User {
  constructor(
    public name: string,
    private age: number,
  ) {}
}
```

**抽象类：**

```ts
abstract class Shape {
  abstract area(): number; // 子类必须实现

  printArea() {
    console.log(this.area());
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }
  area() {
    return Math.PI * this.radius ** 2;
  }
}

// const s = new Shape(); // ❌ 不能实例化抽象类
```

**面试加分点**：能区分 `private` 和 `#` 私有字段（`#` 是 ES 标准，运行时真正私有，`private` 只在编译期检查）；能解释抽象类与接口的区别。

</details>

---

### 14. `implements` 和 `extends` 有什么区别？

<details>
<summary>查看答案</summary>

**extends**：继承另一个类，获得其属性和方法。

```ts
class Animal {
  move() {}
}

class Dog extends Animal {
  bark() {}
}
```

**implements**：实现一个接口，必须满足接口的结构约束。

```ts
interface CanFly {
  fly(): void;
}

class Bird implements CanFly {
  fly() {
    console.log("flying");
  }
}
```

**区别：**

| 维度     | `extends`    | `implements`           |
| -------- | ------------ | ---------------------- |
| 目标     | 类           | 接口/类型别名          |
| 继承实现 | 继承父类实现 | 只约束结构，不继承实现 |
| 多继承   | ❌ 单继承    | ✅ 可实现多个接口      |
| 构造函数 | 继承父类构造 | 无                     |

**一个类可以同时 extends 和 implements：**

```ts
class Eagle extends Bird implements CanFly, CanRun {
  fly() {}
  run() {}
}
```

**面试加分点**：能解释接口是「契约」，只检查结构；类继承是「代码复用」，会继承实现和原型链。

</details>

---

## 六、类型兼容性与泛型进阶

### 15. 什么是结构化类型系统？它与名义类型系统有什么区别？

<details>
<summary>查看答案</summary>

**结构化类型系统（Structural Typing）**：只要结构相同，类型就兼容。

```ts
interface Point {
  x: number;
  y: number;
}

class Location {
  x: number;
  y: number;
  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

const p: Point = new Location(1, 2); // ✅ 结构相同即可兼容
```

**名义类型系统（Nominal Typing）**：必须显式声明类型关系（如 Java、C#）。

```java
// Java 中以下代码不成立，除非显式 implements
class Location { int x, y; }
Point p = new Location(1, 2); // ❌
```

**TS 是结构化类型：**

```ts
interface Named {
  name: string;
}

function greet(n: Named) {
  console.log(n.name);
}

greet({ name: "tom", age: 20 }); // ✅ 鸭子类型
```

**例外：**

私有成员和 protected 成员会让类在兼容时要求来源相同。

```ts
class Animal {
  private name: string;
}
class Dog {
  private name: string;
}

let a: Animal = new Dog(); // ❌ 私有成员来源不同
```

**面试加分点**：能解释 TS 选择结构化类型是为了与 JS 的鸭子类型习惯保持一致；能说出这种设计让类型更灵活但也可能隐藏风险。

</details>

---

### 16. 协变（Covariance）、逆变（Contravariance）、双向协变（Bivariance）是什么？

<details>
<summary>查看答案</summary>

**子类型关系：**

```ts
interface Animal {
  name: string;
}
interface Dog extends Animal {
  bark(): void;
}

let animal: Animal = { name: "a" };
let dog: Dog = { name: "d", bark: () => {} };

animal = dog; // ✅ Dog 是 Animal 的子类型
// dog = animal; // ❌
```

**协变**：子类型可以赋值给父类型（数组、对象属性、返回值）。

```ts
let animals: Animal[] = [];
let dogs: Dog[] = [];
animals = dogs; // ✅ 数组是协变的
```

**逆变**：父类型可以赋值给子类型（函数参数）。

```ts
type AnimalFn = (a: Animal) => void;
type DogFn = (d: Dog) => void;

let animalFn: AnimalFn = (a: Animal) => {};
let dogFn: DogFn = (d: Dog) => {};

dogFn = animalFn; // ✅ 参数是逆变的
// animalFn = dogFn; // ❌
```

**双向协变**：TS 2.x 默认函数参数是双向协变的（strictFunctionTypes 关闭时）。

```ts
// strictFunctionTypes: false 时可能成立，但不安全
animalFn = dogFn; // 可能不报错
```

**开启 strictFunctionTypes：**

```ts
// tsconfig.json
{
  "compilerOptions": {
    "strictFunctionTypes": true
  }
}
```

**面试加分点**：能解释「返回值是协变的，参数是逆变的」是类型安全的；能说出开启 `strictFunctionTypes` 后 TS 对函数参数采用正确的逆变。

</details>

---

## 七、模块、声明文件与工程化

### 17. TypeScript 中 `declare` 关键字有什么用？`.d.ts` 文件是什么？

<details>
<summary>查看答案</summary>

**declare**：告诉 TS 「这个变量/模块/类型已经存在」，只提供类型信息，不生成实际代码。

```ts
// 声明全局变量
declare const VERSION: string;

// 声明全局函数
declare function $(selector: string): HTMLElement;

// 声明模块
declare module "my-lib" {
  export function foo(): void;
}

// 声明图片等非 JS 资源
declare module "*.png" {
  const src: string;
  export default src;
}
```

**.d.ts 文件：**

- 只包含类型声明，不包含实现
- 用于为 JS 库提供类型支持
- 常见：`node_modules/@types/xxx/index.d.ts`

**自动生成声明文件：**

```json
{
  "compilerOptions": {
    "declaration": true,
    "declarationDir": "./types"
  }
}
```

**面试加分点**：能解释 `.d.ts` 在编译后不会被输出为 JS；能说明 `@types` 组织为大量 JS 库提供了社区类型声明。

</details>

---

### 18. `tsconfig.json` 中常用的编译选项有哪些？

<details>
<summary>查看答案</summary>

**常用选项表：**

| 选项                               | 作用                       | 推荐值     |
| ---------------------------------- | -------------------------- | ---------- |
| `target`                           | 编译目标 JS 版本           | `ES2020`   |
| `module`                           | 模块系统                   | `ESNext`   |
| `strict`                           | 开启所有严格类型检查       | `true`     |
| `esModuleInterop`                  | 改善 CommonJS 模块导入体验 | `true`     |
| `skipLibCheck`                     | 跳过声明文件类型检查       | `true`     |
| `forceConsistentCasingInFileNames` | 强制文件名大小写一致       | `true`     |
| `resolveJsonModule`                | 支持导入 JSON 文件         | `true`     |
| `moduleResolution`                 | 模块解析策略               | `node`     |
| `baseUrl` / `paths`                | 路径别名                   | 按项目配置 |
| `outDir`                           | 输出目录                   | `./dist`   |
| `sourceMap`                        | 生成 source map            | `true`     |
| `noImplicitAny`                    | 禁止隐式 any               | `true`     |
| `strictNullChecks`                 | 严格 null/undefined 检查   | `true`     |

**示例配置：**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "outDir": "./dist",
    "sourceMap": true
  },
  "include": ["src/**/*"]
}
```

**面试加分点**：能解释 `strict: true` 会同时开启 `noImplicitAny`、`strictNullChecks`、`strictFunctionTypes` 等；能说明 `skipLibCheck` 可以加快编译速度但可能隐藏第三方库类型问题。

</details>

---

### 19. TypeScript 中的模块解析策略是什么？

<details>
<summary>查看答案</summary>

**两种模块解析策略：**

| 策略      | 特点                  | 适用场景     |
| --------- | --------------------- | ------------ |
| `classic` | 相对路径解析，较简单  | TS 1.6 之前  |
| `node`    | 模拟 Node.js 模块解析 | 现代项目默认 |

**Node 模块解析流程（简化）：**

```
import { foo } from "./bar"
→ 先找 bar.ts / bar.tsx / bar.d.ts
→ 再找 bar/package.json 的 types/main
→ 再找 bar/index.ts / index.tsx / index.d.ts

import { foo } from "bar"
→ node_modules/bar
→ node_modules/bar/package.json
→ node_modules/@types/bar
```

**路径别名：**

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  }
}
```

```ts
import Button from "@components/Button";
```

**面试加分点**：能解释 TS 编译时路径别名需要配合打包工具（Webpack/Vite）的 alias 配置才能在运行时生效；能区分编译期解析和运行时解析。

</details>

---

## 八、进阶与最佳实践

### 20. TypeScript 项目中有哪些常见最佳实践？

<details>
<summary>查看答案</summary>

**1）开启严格模式**

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

**2）避免滥用 any**

```ts
// ❌
function process(data: any) {}

// ✅
function process<T>(data: T): T {
  return data;
}
// 或 unknown + 类型守卫
function process(data: unknown) {}
```

**3）优先使用 interface/type 而不是内联类型**

```ts
// ❌
function fn(user: { name: string; age: number }) {}

// ✅
interface User {
  name: string;
  age: number;
}
function fn(user: User) {}
```

**4）合理使用泛型约束**

```ts
function getLength<T extends { length: number }>(x: T) {
  return x.length;
}
```

**5）使用字面量类型和联合类型替代 magic string**

```ts
type Status = "pending" | "success" | "error";
```

**6）泛型组件/Hook 的类型参数尽量可推断**

```ts
function useArray<T>(initial: T[]) { ... }
useArray([1, 2, 3]); // 无需显式 <number>
```

**7）为第三方 JS 库补充类型声明**

```ts
// types/xxx.d.ts
declare module "legacy-lib" {
  export const version: string;
}
```

**8）善用工具类型减少重复**

```ts
type UserUpdate = Partial<User>;
type UserPublic = Pick<User, "id" | "name">;
```

**9）类型与运行时校验结合**

```ts
import { z } from "zod";

const UserSchema = z.object({
  name: z.string(),
  age: z.number(),
});

type User = z.infer<typeof UserSchema>;

const user = UserSchema.parse(data); // 运行时校验 + 类型推导
```

**10）持续重构类型**

- 不要一次写完美类型，随业务发展迭代
- 抽取公共类型到 `types/` 或 `shared/` 目录

**面试加分点**：能强调 TS 的价值在于「约束」而非「限制」；能结合 Zod/Yup 等库说明类型安全需要编译期和运行时双重保障。

</details>

---

## 使用说明

- 在 Cursor / VSCode / Qoder 等 Markdown 预览中，点击 **「查看答案」** 即可展开。
- 每题答案均包含：**核心要点 → 代码示例 → 对比/原理 → 坑点 → 面试加分点**
- 若预览环境不支持 `<details>`，请使用支持 HTML 的 Markdown 阅读器查看。
