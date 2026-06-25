# JavaScript 面试题（含隐藏答案 · 详细版）

共 **20 题**，涵盖基础类型、作用域与闭包、原型继承、异步编程、事件循环、ES6+ 与工程化。点击「查看答案」即可展开，每题都包含**原理 / 对比 / 代码示例 / 坑点 / 面试加分点**。

---

## 一、基础类型与核心概念

### 1. JavaScript 有哪些数据类型？如何准确判断类型？

<details>
<summary>查看答案</summary>

**基本数据类型（7 种）：**

```js
string、number、boolean、undefined、null、symbol、bigint
```

**引用数据类型：**

```js
Object（包括 Array、Function、Date、RegExp、Map、Set 等）
```

**类型判断方法对比：**

| 方法                        | 示例                                                    | 说明                     |
| --------------------------- | ------------------------------------------------------- | ------------------------ |
| `typeof`                    | `typeof 1` → number                                     | 适合基本类型和 function  |
| `instanceof`                | `[] instanceof Array`                                   | 判断原型链，适合引用类型 |
| `constructor`               | `[].constructor`                                        | 可被重写，不推荐         |
| `Object.prototype.toString` | `Object.prototype.toString.call([])` → `[object Array]` | 最准确                   |

**typeof 的坑：**

```js
typeof null; // "object"（历史遗留 bug）
typeof []; // "object"
typeof {}; // "object"
typeof NaN; // "number"
```

**推荐封装：**

```js
function getType(value) {
  return Object.prototype.toString.call(value).slice(8, -1).toLowerCase();
}

getType([]); // "array"
getType({}); // "object"
getType(null); // "null"
getType(/a/); // "regexp"
```

**面试加分点**：能解释 `typeof null === 'object'` 是 JS 早期实现的 bug；`instanceof` 不能判断基本类型但可以配合包装对象判断；实际项目中推荐用 `Object.prototype.toString.call`。

</details>

---

### 2. `==` 和 `===` 有什么区别？类型转换规则是什么？

<details>
<summary>查看答案</summary>

**核心区别：**

| 运算符 | 比较方式               | 推荐场景 |
| ------ | ---------------------- | -------- |
| `==`   | 宽松相等，会做类型转换 | 不推荐   |
| `===`  | 严格相等，不做类型转换 | 优先使用 |

**== 转换规则（简化）：**

1. 类型相同：比较值
2. `null == undefined` → true
3. 一方是 number，另一方是 string：string 转 number
4. 一方是 boolean：boolean 转 number（true→1, false→0）
5. 一方是 object，另一方是基本类型：object 转基本类型（valueOf/toString）

**经典坑：**

```js
[] == ![]; // true
// 解析：![] → false；[] == false → [] 转数字 0 → 0 == 0 → true

"0" == false; // true
null == undefined; // true
null == 0; // false
```

**对象转基本类型：**

```js
const obj = {
  valueOf() {
    return 1;
  },
  toString() {
    return "2";
  },
};

obj + 1; // 2（优先 valueOf）
String(obj); // "2"（显式转 string 用 toString）
```

**面试加分点**：能说出 ECMAScript 规范中 `==` 的比较算法有 12 步；推荐项目中一律使用 `===` 和 `!==`，除非明确需要处理 `null/undefined` 的宽松判断。

</details>

---

### 3. `var`、`let`、`const` 有什么区别？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 特性         | `var`                   | `let`           | `const`            |
| ------------ | ----------------------- | --------------- | ------------------ |
| 作用域       | 函数作用域              | 块级作用域      | 块级作用域         |
| 变量提升     | ✅ 提升，初始 undefined | ✅ 提升，但 TDZ | ✅ 提升，但 TDZ    |
| 重复声明     | ✅ 允许                 | ❌ 报错         | ❌ 报错            |
| 重新赋值     | ✅                      | ✅              | ❌（对象属性可改） |
| 全局对象属性 | ✅                      | ❌              | ❌                 |

**TDZ（Temporal Dead Zone，暂时性死区）：**

```js
console.log(a); // undefined（var 提升）
var a = 1;

console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 2;
```

**块级作用域：**

```js
for (var i = 0; i < 3; i++) {}
console.log(i); // 3

for (let j = 0; j < 3; j++) {}
console.log(j); // ReferenceError
```

**const 注意：**

```js
const obj = { a: 1 };
obj.a = 2; // ✅ 修改属性
obj = {}; // ❌ 重新赋值报错

const arr = [1];
arr.push(2); // ✅
arr = [2]; // ❌
```

**面试加分点**：能解释 TDZ 的本质是「变量已在作用域内声明，但在声明前不可访问」；能说明为什么 ES6 引入 let/const 是为了解决 var 的变量提升和作用域问题。

</details>

---

### 4. 什么是作用域链？什么是闭包？闭包有什么应用场景？

<details>
<summary>查看答案</summary>

**作用域链：**

变量查找时形成的链式结构。JS 采用词法作用域，函数定义时作用域链就确定了。

```js
const a = 1;
function fn() {
  const b = 2;
  function inner() {
    const c = 3;
    console.log(a, b, c); // 从内向外查找
  }
  inner();
}
```

**闭包：**

函数能访问定义时的词法作用域，即使这个作用域已经执行完毕。

```js
function createCounter() {
  let count = 0;
  return function () {
    return ++count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

**应用场景：**

| 场景       | 示例             |
| ---------- | ---------------- |
| 数据私有化 | 模块模式、计数器 |
| 函数柯里化 | `add(a)(b)`      |
| 防抖节流   | 保存定时器状态   |
| 回调函数   | 循环中的异步操作 |

**闭包的坑：**

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3 3 3
}

// 解决方案 1：let
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0 1 2
}

// 解决方案 2：IIFE
for (var i = 0; i < 3; i++) {
  (function (i) {
    setTimeout(() => console.log(i), 100);
  })(i);
}
```

**面试加分点**：能解释闭包会导致变量常驻内存，使用不当可能造成内存泄漏；能说出垃圾回收机制中闭包变量不会被回收。

</details>

---

## 二、this 与原型继承

### 5. `this` 的指向规则有哪些？箭头函数的 this 有什么特殊？

<details>
<summary>查看答案</summary>

**this 指向规则：**

| 调用方式        | this 指向                                |
| --------------- | ---------------------------------------- |
| 普通函数调用    | 严格模式 undefined，非严格 window/global |
| 对象方法调用    | 调用该方法的对象                         |
| new 构造函数    | 新创建的实例                             |
| call/apply/bind | 指定的第一个参数                         |
| DOM 事件处理    | 触发事件的 DOM 元素                      |

**示例：**

```js
function fn() {
  console.log(this);
}
fn(); // window / global

const obj = {
  name: "tom",
  say() {
    console.log(this.name);
  },
};
obj.say(); // "tom"

const say = obj.say;
say(); // undefined（严格模式）或 window.name
```

**箭头函数：**

- 没有自己的 this，继承外层作用域的 this
- 不能通过 call/apply/bind 改变 this
- 不能作为构造函数
- 没有 arguments 对象

```js
const obj = {
  name: "tom",
  say: () => {
    console.log(this.name); // this 指向 obj 外层（window）
  },
};
obj.say(); // undefined
```

**经典面试题：**

```js
const obj = {
  name: "obj",
  fn: function () {
    console.log(this.name);
  },
  arrow: () => {
    console.log(this.name);
  },
};

obj.fn(); // "obj"
obj.arrow(); // undefined（指向外层）

const fn = obj.fn;
fn(); // undefined
```

**面试加分点**：能解释箭头函数没有 prototype，不能用 new；能用「this 指向最后调用它的对象」帮助记忆普通函数规则。

</details>

---

### 6. 原型和原型链是什么？如何基于原型实现继承？

<details>
<summary>查看答案</summary>

**核心概念：**

- `prototype`：函数特有的属性，指向原型对象
- `__proto__`：对象特有的属性，指向构造该对象的构造函数的原型
- `constructor`：原型对象上的属性，指回构造函数

```js
function Person() {}
const p = new Person();

p.__proto__ === Person.prototype; // true
Person.prototype.constructor === Person; // true
```

**原型链：**

当访问对象属性时，若对象本身没有，会沿着 `__proto__` 向上查找，直到 `Object.prototype`（终点是 null）。

```js
p → Person.prototype → Object.prototype → null
```

**继承方式对比：**

| 方式         | 优点         | 缺点             |
| ------------ | ------------ | ---------------- |
| 原型链继承   | 简单         | 引用类型共享     |
| 构造函数继承 | 不共享引用   | 无法复用父类方法 |
| 组合继承     | 综合两者     | 调用两次父类构造 |
| 寄生组合继承 | 最优         | 稍复杂           |
| class 继承   | 语法糖，清晰 | 本质仍是原型继承 |

**寄生组合继承：**

```js
function Parent(name) {
  this.name = name;
  this.colors = ["red"];
}
Parent.prototype.say = function () {
  console.log(this.name);
};

function Child(name, age) {
  Parent.call(this, name); // 继承属性
  this.age = age;
}

Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
```

**class 语法糖：**

```js
class Parent {
  constructor(name) {
    this.name = name;
  }
  say() {
    console.log(this.name);
  }
}

class Child extends Parent {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
}
```

**面试加分点**：能画出原型链图；能解释 `class` 本质仍是函数，`extends` 底层是原型链继承；能说出 `instanceof` 就是基于原型链判断。

</details>

---

## 三、异步编程

### 7. Promise 是什么？它的三种状态如何转换？

<details>
<summary>查看答案</summary>

**Promise 状态：**

| 状态      | 说明   |
| --------- | ------ |
| pending   | 进行中 |
| fulfilled | 已成功 |
| rejected  | 已失败 |

状态一旦改变就不可再次改变：pending → fulfilled 或 pending → rejected。

**基本用法：**

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("success");
  }, 1000);
});

p.then((res) => console.log(res))
  .catch((err) => console.log(err))
  .finally(() => console.log("done"));
```

**链式调用：**

```js
fetchUser()
  .then((user) => fetchOrders(user.id))
  .then((orders) => console.log(orders))
  .catch((err) => console.log(err));
```

**常用静态方法：**

```js
Promise.all([p1, p2, p3]); // 全部成功才成功，一个失败就失败
Promise.allSettled([p1, p2]); // 等全部完成，返回状态数组
Promise.race([p1, p2]); // 返回最快完成的那个
Promise.any([p1, p2]); // 返回第一个成功的，全失败才失败
Promise.resolve(value); // 快速返回 fulfilled
Promise.reject(reason); // 快速返回 rejected
```

**Promise.all 与 Promise.allSettled 对比：**

| 特性     | Promise.all        | Promise.allSettled       |
| -------- | ------------------ | ------------------------ |
| 失败行为 | 一个失败即整体失败 | 不管失败，等全部完成     |
| 结果     | 成功值数组         | `{status, value/reason}` |
| 适用场景 | 强依赖全部结果     | 需要知道每个请求的状态   |

**面试加分点**：能手写 Promise.all / Promise.race；能解释 Promise 链式调用的返回值规则（then 返回新 Promise）；能说出 Promise 解决了回调地狱问题。

</details>

---

### 8. `async/await` 是什么？它和 Promise 有什么关系？

<details>
<summary>查看答案</summary>

**async/await 是 Promise 的语法糖**，让异步代码看起来像同步代码。

```js
async function getUser() {
  try {
    const user = await fetch("/api/user");
    const orders = await fetch(`/api/orders/${user.id}`);
    return orders;
  } catch (err) {
    console.log(err);
  }
}
```

**async 函数返回值：**

```js
async function fn() {
  return 1;
}
fn().then((res) => console.log(res)); // 1
// 返回值自动包装成 Promise
```

**await 后面可以跟：**

- Promise 对象
- 非 Promise 值（会包装成 resolved）

```js
async function fn() {
  const a = await 1; // 等价 await Promise.resolve(1)
  console.log(a);
}
```

**并行执行：**

```js
// ❌ 串行
const a = await fetchA();
const b = await fetchB();

// ✅ 并行
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

**异常处理：**

```js
async function fn() {
  try {
    await fetchData();
  } catch (err) {
    // 处理错误
  }
}
```

**面试加分点**：能解释 `async` 函数内部抛错会返回 rejected Promise；能说出 `await` 会阻塞当前 async 函数内后续代码，但不阻塞主线程。

</details>

---

## 四、事件循环与执行机制

### 9. JavaScript 的事件循环（Event Loop）是什么？宏任务和微任务有什么区别？

<details>
<summary>查看答案</summary>

**为什么需要事件循环？**

JS 是单线程语言，事件循环负责协调同步任务、异步任务的执行顺序。

**执行流程：**

1. 执行同步代码（调用栈）
2. 同步代码执行完，检查微任务队列，全部执行
3. 执行一个宏任务
4. 再次检查微任务队列
5. 循环往复

**任务分类：**

| 类型   | 示例                                               |
| ------ | -------------------------------------------------- |
| 宏任务 | setTimeout、setInterval、I/O、UI rendering、script |
| 微任务 | Promise.then、MutationObserver、queueMicrotask     |

**执行顺序：**

```
同步代码 → 微任务（清空） → 宏任务（一个） → 微任务（清空） → 宏任务（一个）...
```

**示例：**

```js
console.log("1");
setTimeout(() => console.log("2"), 0);
Promise.resolve().then(() => console.log("3"));
console.log("4");
// 输出：1 4 3 2
```

**更复杂的例子：**

```js
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
console.log("script start");
setTimeout(() => console.log("setTimeout"), 0);
async1();
new Promise((resolve) => {
  console.log("promise1");
  resolve();
}).then(() => console.log("promise2"));
console.log("script end");

// script start → async1 start → async2 → promise1 → script end → async1 end → promise2 → setTimeout
```

**面试加分点**：能解释 `await` 后面的代码会被推入微任务队列；能区分浏览器和 Node.js 事件循环的差异（Node 有 phases 阶段）。

</details>

---

### 10. 什么是防抖（debounce）和节流（throttle）？如何实现？

<details>
<summary>查看答案</summary>

**防抖：**

事件触发后，等待一段时间才执行；如果期间再次触发，重新计时。

**适用场景：** 搜索框输入、窗口 resize、表单校验。

```js
function debounce(fn, delay) {
  let timer = null;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}
```

**节流：**

一段时间内只执行一次，即使多次触发。

**适用场景：** 滚动加载、按钮点击、mousemove。

```js
function throttle(fn, interval) {
  let lastTime = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastTime >= interval) {
      lastTime = now;
      fn.apply(this, args);
    }
  };
}
```

**对比：**

| 特性     | 防抖           | 节流         |
| -------- | -------------- | ------------ |
| 触发频率 | 停止触发后执行 | 固定间隔执行 |
| 比喻     | 电梯关门       | 水龙头滴水   |
| 场景     | 搜索输入       | 滚动加载     |

**带立即执行的防抖：**

```js
function debounce(fn, delay, immediate = false) {
  let timer = null;
  return function (...args) {
    if (timer) clearTimeout(timer);
    if (immediate && !timer) {
      fn.apply(this, args);
    }
    timer = setTimeout(() => {
      if (!immediate) fn.apply(this, args);
      timer = null;
    }, delay);
  };
}
```

**面试加分点**：能手写两种实现；能解释 `apply(this, args)` 的重要性（保留 this 和事件对象）；能结合 `requestAnimationFrame` 实现更平滑的节流。

</details>

---

## 五、ES6+ 与工程化

### 11. ES6+ 中常用的新特性有哪些？

<details>
<summary>查看答案</summary>

**1）let / const**

块级作用域、不可重复声明、TDZ。

**2）箭头函数**

没有自己的 this、arguments，不能 new。

**3）模板字符串**

```js
const name = "tom";
const str = `hello, ${name}`;
```

**4）解构赋值**

```js
const { name, age } = user;
const [a, b] = arr;
```

**5）Promise / async await**

解决异步回调问题。

**6）Class**

语法糖，底层仍是原型继承。

**7）模块化 import/export**

```js
import { foo } from "./foo.js";
export const bar = 1;
export default bar;
```

**8）默认参数 / 剩余参数 / 展开运算符**

```js
function fn(a = 1, ...args) {}
const arr2 = [...arr1];
```

**9）Map / Set / WeakMap / WeakSet**

```js
const map = new Map();
const set = new Set([1, 2, 2, 3]); // {1, 2, 3}
```

**10）Symbol / BigInt**

```js
const s = Symbol("desc");
const big = 123456789012345678901n;
```

**11）可选链 / 空值合并**

```js
const name = user?.profile?.name;
const value = input ?? "default"; // 只有 null/undefined 才用默认值
```

**12）Proxy / Reflect**

Vue 3 响应式基础。

**面试加分点**：能说出 ES6 也叫 ES2015，之后每年发布一个新版本；能区分 `??` 和 `||`（`||` 对 0、''、false 都会用默认值）。

</details>

---

### 12. 深拷贝和浅拷贝有什么区别？如何实现深拷贝？

<details>
<summary>查看答案</summary>

**浅拷贝：**

只复制对象的第一层，深层对象共享引用。

```js
const obj = { a: 1, b: { c: 2 } };
const shallow = { ...obj };
shallow.b.c = 3;
console.log(obj.b.c); // 3
```

**深拷贝：**

完全复制所有层级，新对象和原对象互不影响。

**实现方式：**

**1）JSON 序列化（最简单，有局限）**

```js
const deep = JSON.parse(JSON.stringify(obj));
```

局限：丢失 function、undefined、Symbol、循环引用、Date 会转字符串等。

**2）手写递归**

```js
function deepClone(obj, map = new WeakMap()) {
  if (obj === null || typeof obj !== "object") return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (map.has(obj)) return map.get(obj);

  const clone = Array.isArray(obj) ? [] : {};
  map.set(obj, clone);

  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      clone[key] = deepClone(obj[key], map);
    }
  }
  return clone;
}
```

**3）使用库**

```js
import cloneDeep from "lodash/cloneDeep";
const copy = cloneDeep(obj);
```

**面试加分点**：能手写深拷贝并处理循环引用；能解释 `WeakMap` 用于解决循环引用问题，且不会阻止垃圾回收。

</details>

---

### 13. CommonJS 和 ES Module 有什么区别？

<details>
<summary>查看答案</summary>

**核心区别表：**

| 特性      | CommonJS                 | ES Module              |
| --------- | ------------------------ | ---------------------- |
| 语法      | require / module.exports | import / export        |
| 加载时机  | 运行时动态加载           | 编译时静态分析         |
| 加载方式  | 同步                     | 异步（浏览器）/ 编译时 |
| 值传递    | 拷贝                     | 引用（只读）           |
| 顶层 this | module.exports           | undefined              |
| 循环引用  | 部分导出                 | 支持（引用绑定）       |
| 使用环境  | Node.js                  | 浏览器 + Node          |

**CommonJS：**

```js
// a.js
module.exports = { foo: 1 };

// b.js
const a = require("./a");
console.log(a.foo);
```

**ES Module：**

```js
// a.js
export const foo = 1;
export default foo;

// b.js
import foo, { foo as bar } from "./a.js";
```

**动态 import：**

```js
const module = await import("./module.js");
```

**面试加分点**：能解释 CommonJS 的 `require` 是值拷贝，导出后修改内部变量不会影响引入方；ESM 是 live binding，导出值的修改会影响引入方。

</details>

---

### 14. 什么是 XSS 和 CSRF？如何防御？

<details>
<summary>查看答案</summary>

**XSS（跨站脚本攻击）：**

攻击者注入恶意脚本，窃取用户信息或执行恶意操作。

**类型：**

| 类型       | 说明                 |
| ---------- | -------------------- |
| 存储型 XSS | 恶意脚本存入数据库   |
| 反射型 XSS | 通过 URL 参数触发    |
| DOM 型 XSS | 前端 JS 处理不当导致 |

**防御：**

- 输入过滤、输出转义（如 `innerText` 替代 `innerHTML`）
- Content Security Policy（CSP）
- 对 Cookie 设置 `HttpOnly`

**CSRF（跨站请求伪造）：**

攻击者诱导用户在已登录状态下访问恶意链接，以用户身份执行操作。

**防御：**

- CSRF Token
- SameSite Cookie 属性
- 双重 Cookie 验证
- 校验 Referer/Origin

**对比：**

| 特性 | XSS                 | CSRF            |
| ---- | ------------------- | --------------- |
| 目标 | 用户浏览器          | 已认证用户      |
| 手段 | 注入脚本            | 伪造请求        |
| 防御 | 转义、CSP、HttpOnly | Token、SameSite |

**面试加分点**：能解释 XSS 是「脚本注入」，CSRF 是「借用户之手发请求」；能说出现代框架（React/Vue）默认会对插入 DOM 的内容做转义防御 XSS。

</details>

---

## 六、进阶与性能

### 15. 什么是事件委托？有什么优缺点？

<details>
<summary>查看答案</summary>

**事件委托：**

利用事件冒泡机制，将子元素的事件监听器绑定到父元素上，通过 `event.target` 判断具体触发元素。

```js
const list = document.getElementById("list");
list.addEventListener("click", (e) => {
  if (e.target.tagName === "LI") {
    console.log(e.target.textContent);
  }
});
```

**优点：**

- 减少事件监听器数量，节省内存
- 动态新增子元素无需重新绑定事件

**缺点：**

- 需要判断 target，逻辑稍复杂
- 不适用于不冒泡的事件（如 focus、blur）

**事件传播三个阶段：**

1. 捕获阶段（capture）
2. 目标阶段（target）
3. 冒泡阶段（bubble）

```js
elem.addEventListener("click", fn, true); // 捕获阶段监听
```

**面试加分点**：能解释事件委托基于事件冒泡；能说出 `stopPropagation()` 和 `preventDefault()` 的区别。

</details>

---

### 16. 什么是函数柯里化（Currying）？有什么应用场景？

<details>
<summary>查看答案</summary>

**柯里化：**

将一个接收多个参数的函数转换成一系列接收单一参数的函数。

```js
function add(a) {
  return function (b) {
    return function (c) {
      return a + b + c;
    };
  };
}

add(1)(2)(3); // 6
```

**通用柯里化函数：**

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function (...args2) {
      return curried.apply(this, args.concat(args2));
    };
  };
}

const sum = (a, b, c) => a + b + c;
const curriedSum = curry(sum);
curriedSum(1)(2)(3); // 6
curriedSum(1, 2)(3); // 6
```

**应用场景：**

- 参数复用：`const addTen = add(10)`
- 函数式编程
- 延迟计算

**面试加分点**：能解释柯里化是函数式编程的重要技巧；能说出 `fn.length` 表示函数期望的参数个数。

</details>

---

### 17. 解释 JavaScript 的垃圾回收机制。

<details>
<summary>查看答案</summary>

**垃圾回收（GC）**：自动回收不再使用的内存。

**主要算法：**

**1）引用计数（已少用）**

- 对象被引用次数为 0 时回收
- 缺点：无法处理循环引用

```js
function fn() {
  const a = {};
  const b = {};
  a.ref = b;
  b.ref = a;
}
```

**2）标记-清除（Mark-Sweep）**

- 从根对象（global、栈上变量）出发，标记可达对象
- 清除未被标记的对象
- 现代浏览器主流算法

**3）标记-整理（Mark-Compact）**

- 清除后整理内存，减少碎片

**V8 的分代回收：**

| 区域   | 对象特点           | 回收算法                  |
| ------ | ------------------ | ------------------------- |
| 新生代 | 存活时间短、小对象 | Scavenge（复制算法）      |
| 老生代 | 存活时间长、大对象 | Mark-Sweep + Mark-Compact |

**内存泄漏常见原因：**

- 全局变量
- 闭包未释放
- 定时器未清除
- DOM 引用未释放
- 事件监听未移除

**面试加分点**：能解释 V8 的「新生代」使用复制算法是因为大部分对象很快死亡；能说出 `WeakMap`/`WeakSet` 的键是弱引用，不会阻止垃圾回收。

</details>

---

### 18. `0.1 + 0.2 !== 0.3` 的原因是什么？如何解决？

<details>
<summary>查看答案</summary>

**原因：**

计算机使用二进制浮点数存储数字，0.1 和 0.2 在二进制下是无限循环小数，存储时产生精度损失。

```js
0.1 + 0.2; // 0.30000000000000004
```

**解决方案：**

**1）指定精度比较**

```js
function isEqual(a, b, epsilon = 1e-10) {
  return Math.abs(a - b) < epsilon;
}
```

**2）转整数运算**

```js
function add(a, b) {
  return Math.round((a + b) * 100) / 100;
}
```

**3）使用库**

```js
import Decimal from "decimal.js";
new Decimal(0.1).plus(0.2).toNumber(); // 0.3
```

**面试加分点**：能解释 IEEE 754 双精度浮点数标准；能说出金融计算等场景应避免直接用 JS 浮点数，改用整数分或专业库。

</details>

---

### 19. 什么是事件循环中的 requestAnimationFrame？和 setTimeout 有什么区别？

<details>
<summary>查看答案</summary>

**requestAnimationFrame（rAF）：**

- 在下一次浏览器重绘前执行回调
- 频率通常与显示器刷新率一致（60Hz 即约 16.7ms）
- 会自动暂停（页面不可见时），节省资源

```js
function animate() {
  // 执行动画
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

**与 setTimeout 对比：**

| 特性     | requestAnimationFrame | setTimeout                      |
| -------- | --------------------- | ------------------------------- |
| 触发时机 | 屏幕刷新前            | 固定延迟后                      |
| 精度     | 与刷新率同步          | 可能不准，低于 4ms 会被强制延迟 |
| 节能     | 页面隐藏时自动暂停    | 继续执行                        |
| 回调参数 | 提供时间戳            | 无                              |
| 适用场景 | 动画                  | 通用延时任务                    |

**面试加分点**：能解释 rAF 更适合动画是因为它避免了丢帧；能说出 rAF 回调执行时机在「重绘之前」，适合读取布局并修改样式。

</details>

---

### 20. JavaScript 性能优化有哪些常见手段？

<details>
<summary>查看答案</summary>

**1）减少 DOM 操作**

- 批量修改 class，使用 DocumentFragment
- 避免频繁读取引起重排的属性

**2）事件委托**

减少事件监听器数量。

**3）防抖节流**

控制高频事件触发频率。

**4）图片懒加载**

```js
<img loading="lazy" src="..." />
```

**5）代码分割与懒加载**

```js
const module = await import("./heavy.js");
```

**6）缓存**

- localStorage / sessionStorage
- 内存缓存
- HTTP 缓存策略

**7）避免内存泄漏**

- 清理定时器和事件监听
- 避免意外全局变量
- 合理使用 WeakMap/WeakSet

**8）使用 Web Worker**

将重计算放到后台线程。

**9）减少重排重绘**

- 使用 `transform` 和 `opacity` 做动画
- 批量修改样式

**10）算法优化**

- 减少嵌套循环
- 使用 Map/Set 替代数组查找

**面试加分点**：能强调「先测量再优化」，使用 Chrome DevTools Performance 面板定位瓶颈；能区分性能优化的不同层面（网络、渲染、脚本执行）。

</details>

---

## 使用说明

- 在 Cursor / VSCode / Qoder 等 Markdown 预览中，点击 **「查看答案」** 即可展开。
- 每题答案均包含：**核心要点 → 代码示例 → 对比/原理 → 坑点 → 面试加分点**
- 若预览环境不支持 `<details>`，请使用支持 HTML 的 Markdown 阅读器查看。
