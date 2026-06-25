# JavaScript 面试题（含隐藏答案 · 详细版）

共 **30 题**，涵盖基础类型、作用域与闭包、原型继承、异步编程、事件循环、ES6+ 与工程化、高频手写与浏览器原理。点击「查看答案」即可展开，答案以**口述要点 / 对比 / 核心原理 / 坑点 / 面试加分点**为主，代码已精简。

---

## 一、基础类型与核心概念

### 1. JavaScript 有哪些数据类型？如何准确判断类型？

<details>
<summary>查看答案</summary>

**基本数据类型（7 种）**：`string`、`number`、`boolean`、`undefined`、`null`、`symbol`、`bigint`

**引用数据类型**：`Object`（包括 `Array`、`Function`、`Date`、`RegExp`、`Map`、`Set` 等）

**类型判断方法对比：**

| 方法                        | 示例                                                    | 说明                     |
| --------------------------- | ------------------------------------------------------- | ------------------------ |
| `typeof`                    | `typeof 1` → number                                     | 适合基本类型和 function  |
| `instanceof`                | `[] instanceof Array`                                   | 判断原型链，适合引用类型 |
| `constructor`               | `[].constructor`                                        | 可被重写，不推荐         |
| `Object.prototype.toString` | `Object.prototype.toString.call([])` → `[object Array]` | 最准确                   |

**typeof 的坑**：`typeof null` 是 `"object"`（历史遗留 bug）；`typeof []`、`typeof {}` 都是 `"object"`；`typeof NaN` 是 `"number"`。

**推荐判断方式**：`Object.prototype.toString.call(value).slice(8, -1).toLowerCase()` 最准确，能区分 `array`、`null`、`regexp` 等。

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

**经典坑**：`[] == ![]` 为 `true`（`![]` 先转 `false`，`[]` 再转数字 `0`）；`"0" == false`、`null == undefined` 为 `true`；`null == 0` 为 `false`。

**对象转基本类型**：对象参与运算时优先调 `valueOf`，`valueOf` 返回非原始值再调 `toString`；显式 `String(obj)` 直接走 `toString`。

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

**TDZ（暂时性死区）**：`var` 声明会提升并初始化为 `undefined`；`let/const` 虽然也会提升，但在声明前访问会抛 `ReferenceError`。

**块级作用域**：`let/const` 只在 `{}` 内有效；典型例子是 `for (var i ...)` 循环后 `i` 仍可在全局访问，而 `let` 不会。

**const 注意**：`const` 保证的是引用不变，对象/数组可以修改内部属性或元素，但不能重新赋值。

**面试加分点**：能解释 TDZ 的本质是「变量已在作用域内声明，但在声明前不可访问」；能说明为什么 ES6 引入 let/const 是为了解决 var 的变量提升和作用域问题。

</details>

---

### 4. 什么是作用域链？什么是闭包？闭包有什么应用场景？

<details>
<summary>查看答案</summary>

**作用域链**：变量查找时形成的链式结构。JS 采用词法作用域，函数定义时作用域链就确定了，访问变量从内层作用域逐层向外查找。

**闭包**：函数能访问定义时的词法作用域，即使这个作用域已经执行完毕。典型例子是返回的函数内部引用了外部函数的局部变量。

**应用场景：**

| 场景       | 示例             |
| ---------- | ---------------- |
| 数据私有化 | 模块模式、计数器 |
| 函数柯里化 | `add(a)(b)`      |
| 防抖节流   | 保存定时器状态   |
| 回调函数   | 循环中的异步操作 |

**闭包的坑**：`for (var i = 0; i < 3; i++) { setTimeout(() => console.log(i)) }` 会输出 3 个 3，因为 `var` 没有块级作用域，三个回调共享同一个 `i`。改用 `let` 或 IIFE 可解决。

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

**this 指向口诀**：普通函数看调用，`obj.fn()` 中 `this` 是 `obj`；直接调用 `fn()` 是 `undefined`（严格模式）或全局对象；`call/apply/bind` 可显式指定；箭头函数没有自己的 `this`，继承外层词法作用域。

**箭头函数特点**：没有自己的 `this` 和 `arguments`、不能作为构造函数、不能用 `call/apply/bind` 改变 `this`。

**经典面试题要点**：对象方法里的普通函数，`this` 指向调用该方法的对象；提取出来再调用，`this` 会丢失；对象里的箭头函数，`this` 指向定义时外层作用域。

**面试加分点**：能解释箭头函数没有 prototype，不能用 new；能用「this 指向最后调用它的对象」帮助记忆普通函数规则。

</details>

---

### 6. 原型和原型链是什么？如何基于原型实现继承？

<details>
<summary>查看答案</summary>

**核心概念**：

- `prototype`：函数特有的属性，指向原型对象
- `__proto__`：对象特有的属性，指向构造该对象的构造函数的原型
- `constructor`：原型对象上的属性，指回构造函数

**原型链**：访问对象属性时，若本身没有，沿 `__proto__` 向上查找，链路是 `实例 → 构造函数.prototype → Object.prototype → null`。

**继承方式对比：**

| 方式         | 优点         | 缺点             |
| ------------ | ------------ | ---------------- |
| 原型链继承   | 简单         | 引用类型共享     |
| 构造函数继承 | 不共享引用   | 无法复用父类方法 |
| 组合继承     | 综合两者     | 调用两次父类构造 |
| 寄生组合继承 | 最优         | 稍复杂           |
| class 继承   | 语法糖，清晰 | 本质仍是原型继承 |

**寄生组合继承**：子类构造里用 `Parent.call(this, ...)` 继承实例属性，再用 `Child.prototype = Object.create(Parent.prototype)` 继承原型方法，最后修复 `constructor`。

**class 语法糖**：`class extends super` 本质仍是原型继承，只是写法更清晰；`class` 本质是函数，`extends` 底层还是原型链。

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

**基本用法**：`new Promise((resolve, reject) => { ... })`， then 处理成功、catch 处理失败、finally 无论成败都执行。

**链式调用**：每个 `then` 返回新 Promise，可继续 `.then()`，错误可被末尾 `catch` 捕获。

**常用静态方法**：

- `Promise.all`：全部成功才成功，一个失败即整体失败
- `Promise.allSettled`：等全部完成，返回状态数组
- `Promise.race`：返回最快完成的那个
- `Promise.any`：返回第一个成功的，全失败才失败
- `Promise.resolve/reject`：快速创建已落定 Promise

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

**async 函数返回值**：`async` 函数默认返回 Promise，内部 `return` 的值会被包装成 `resolved`，抛错则变成 `rejected`。

**await 后面可以跟**：Promise 对象，或非 Promise 值（会包装成 resolved）。

**并行执行**：多个独立请求不要顺序 `await`，应使用 `Promise.all([fetchA(), fetchB()])` 并行。

**异常处理**：用 `try/catch` 捕获 `await` 抛出的错误。

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

**执行顺序**：同步代码 → 清空微任务 → 执行一个宏任务 → 再清空微任务 → 下一个宏任务，循环往复。

**示例要点**：`console.log` 同步执行；`Promise.then` 进微任务；`setTimeout` 进宏任务，因此即使延迟为 0 也在微任务之后执行。

**async/await 要点**：`await` 后面的代码会被推入微任务队列，等当前同步代码和已有微任务执行完再继续。

**面试加分点**：能解释 `await` 后面的代码会被推入微任务队列；能区分浏览器和 Node.js 事件循环的差异（Node 有 phases 阶段）。

</details>

---

### 10. 什么是防抖（debounce）和节流（throttle）？如何实现？

<details>
<summary>查看答案</summary>

**防抖**：事件触发后等待一段时间才执行，期间再次触发则重新计时。适用搜索框输入、窗口 resize、表单校验。

**实现思路**：闭包保存一个 `timer`，每次触发先 `clearTimeout` 再重新 `setTimeout`，执行时用 `apply(this, args)` 保留 `this` 和参数。

**节流**：一段时间内只执行一次，即使多次触发。适用滚动加载、按钮点击、mousemove。

**实现思路**：记录上次执行时间 `lastTime`，当前时间与上次差值达到间隔才执行。

**对比：**

| 特性     | 防抖           | 节流         |
| -------- | -------------- | ------------ |
| 触发频率 | 停止触发后执行 | 固定间隔执行 |
| 比喻     | 电梯关门       | 水龙头滴水   |
| 场景     | 搜索输入       | 滚动加载     |

**带立即执行的防抖**：增加 `immediate` 参数，首次触发立即执行，后续在延迟结束后执行；实现时用 `timer` 是否为 null 判断是否是首次。

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

**3）模板字符串**：用反引号 `` `hello, ${name}` `` 拼接字符串，支持多行。

**4）解构赋值**：对象解构 `const { name, age } = user`、数组解构 `const [a, b] = arr`。

**5）Promise / async await**

解决异步回调问题。

**6）Class**

语法糖，底层仍是原型继承。

**7）模块化 import/export**：`import { foo } from './foo'`、`export const bar = 1`、`export default`。

**8）默认参数 / 剩余参数 / 展开运算符**：`function fn(a = 1, ...args)`、`const arr2 = [...arr1]`。

**9）Map / Set / WeakMap / WeakSet**：`Map` 可做复杂键值缓存，`Set` 可去重，`WeakMap/WeakSet` 键是弱引用，不阻止垃圾回收。

**10）Symbol / BigInt**：`Symbol` 创建唯一标识；`BigInt` 表示超大整数，尾部加 `n`。

**11）可选链 / 空值合并**：`user?.profile?.name` 避免深层访问报错；`input ?? 'default'` 只在 `null/undefined` 时取默认值（与 `||` 不同，`||` 对 `0`、`''`、`false` 也会生效）。

**12）Proxy / Reflect**

Vue 3 响应式基础。

**面试加分点**：能说出 ES6 也叫 ES2015，之后每年发布一个新版本；能区分 `??` 和 `||`（`||` 对 0、''、false 都会用默认值）。

</details>

---

### 12. 深拷贝和浅拷贝有什么区别？如何实现深拷贝？

<details>
<summary>查看答案</summary>

**浅拷贝**：只复制对象的第一层，深层对象共享引用。如 `{ ...obj }`、`Object.assign`，修改嵌套对象会影响原对象。

**深拷贝**：完全复制所有层级，新对象和原对象互不影响。

**实现方式**：

1. **JSON 序列化**：`JSON.parse(JSON.stringify(obj))` 最简单，但会丢失 function、undefined、Symbol，无法处理循环引用和 Date。
2. **手写递归**：递归复制每个属性，用 `WeakMap` 记录已拷贝对象解决循环引用，特殊处理 Date、RegExp、数组等。
3. **使用库**：`lodash/cloneDeep`、`structuredClone`（现代浏览器/Node 原生支持）。

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

**CommonJS**：`module.exports = {...}` 导出，`const a = require('./a')` 引入；运行时同步加载，导出的是值的拷贝。

**ES Module**：`export/export default` 导出，`import ... from '...'` 引入；编译时静态分析，导出的是只读引用（live binding）。

**动态 import**：`const module = await import('./module.js')` 可在运行时异步加载模块，常用于代码分割和懒加载。

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

**事件委托**：利用事件冒泡，把子元素事件监听器绑定到父元素上，通过 `event.target` 判断具体触发元素。

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

捕获阶段监听：`addEventListener('click', fn, true)`，第三个参数为 `true` 表示捕获阶段触发。

**面试加分点**：能解释事件委托基于事件冒泡；能说出 `stopPropagation()` 和 `preventDefault()` 的区别。

</details>

---

### 16. 什么是函数柯里化（Currying）？有什么应用场景？

<details>
<summary>查看答案</summary>

**柯里化**：将一个接收多个参数的函数转换成一系列接收单一参数的函数。例如 `add(1)(2)(3)`。

**通用柯里化实现思路**：用闭包收集参数，参数个数达到原函数期望长度 `fn.length` 时再调用原函数；支持 `curried(1, 2)(3)` 这种分批传参。

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

例如两个对象互相引用：`a.ref = b; b.ref = a`，即使函数结束，二者引用次数仍不为 0，导致无法回收。

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

`0.1 + 0.2` 实际为 `0.30000000000000004`。

**解决方案**：

1. **指定精度比较**：`Math.abs(a - b) < 1e-10`
2. **转整数运算**：先乘 10 的幂次再运算，如 `Math.round((0.1 + 0.2) * 100) / 100`
3. **使用库**：`decimal.js`、`big.js` 等专业精度库

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

用法：`requestAnimationFrame(animate)` 递归调用，在下次重绘前执行动画逻辑。

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

**4）图片懒加载**：使用 `<img loading="lazy">` 或 IntersectionObserver 实现可视区外图片延迟加载。

**5）代码分割与懒加载**：`await import('./heavy.js')` 动态导入，按需加载 chunk。

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

## 七、高频手写与浏览器原理

### 21. 从输入 URL 到页面显示，浏览器发生了什么？

<details>
<summary>查看答案</summary>

**完整流程**：URL 解析 → DNS 解析 → 建立 TCP 连接（HTTPS 还需 TLS 握手）→ 发送 HTTP 请求 → 服务器响应 → 浏览器解析 HTML → 构建 DOM/CSSOM → 合并渲染树 → Layout → Paint → Composite。

**关键步骤说明：**

| 阶段         | 说明                                                     |
| ------------ | -------------------------------------------------------- |
| DNS 解析     | 域名 → IP 地址，可能经过 DNS 缓存、递归查询              |
| TCP 三次握手 | 建立可靠连接，HTTPS 还需 TLS/SSL 握手                    |
| HTTP 请求    | 发送请求行、请求头，携带 Cookie 等                       |
| 服务器响应   | 返回状态码、响应头、HTML 文档                            |
| 解析 HTML    | 遇到 `<link>` 异步加载 CSS，遇到 `<script>` 默认阻塞解析 |
| 构建渲染树   | DOM + CSSOM，只包含可见节点                              |
| 布局         | 计算每个节点的几何位置                                   |
| 绘制         | 将像素渲染到屏幕上                                       |

**面试加分点**：能解释 `script` 标签默认阻塞 HTML 解析，可用 `defer`/`async` 优化；能说出重定向、HSTS、HTTP/2 多路复用等细节会让流程更复杂。

</details>

---

### 22. 浏览器的缓存机制是怎样的？强缓存和协商缓存有什么区别？

<details>
<summary>查看答案</summary>

**缓存决策流程**：浏览器请求资源 → 检查是否有本地缓存 → 先判断强缓存（`Cache-Control` / `Expires`）是否命中，命中则直接用（200 from cache）→ 未命中则带协商缓存字段发请求 → 服务器判断未变化返回 304，变化则返回 200 + 新资源。

**强缓存：**

| 字段            | 说明                                   |
| --------------- | -------------------------------------- |
| `Expires`       | HTTP/1.0，绝对过期时间，受本地时间影响 |
| `Cache-Control` | HTTP/1.1，相对时间，如 `max-age=3600`  |

示例：`Cache-Control: max-age=31536000, immutable`

**协商缓存：**

| 字段                                  | 说明                           |
| ------------------------------------- | ------------------------------ |
| `Last-Modified` / `If-Modified-Since` | 基于文件最后修改时间，精度秒级 |
| `ETag` / `If-None-Match`              | 基于文件内容哈希，更精确       |

**对比：**

| 特性       | 强缓存         | 协商缓存         |
| ---------- | -------------- | ---------------- |
| 是否发请求 | 不发           | 发请求           |
| 状态码     | 200 from cache | 304              |
| 优先级     | 优先判断       | 强缓存失效后判断 |

**面试加分点**：能说出 `Cache-Control: no-cache` 不是不缓存，而是强制协商缓存；`no-store` 才是真正不缓存。

</details>

---

### 23. 什么是跨域？如何解决跨域问题？

<details>
<summary>查看答案</summary>

**同源策略：**

协议、域名、端口三者都相同才叫同源。任一不同即产生跨域。

例如 `http://a.com:80/api` 与 `http://a.com:8080/page` 因端口不同而跨域。

**解决方案：**

| 方案               | 适用场景                                                  |
| ------------------ | --------------------------------------------------------- |
| **CORS**           | 服务端设置 `Access-Control-Allow-Origin` 等响应头，最常用 |
| **Nginx 反向代理** | 开发环境或线上统一入口，把接口代理到同源路径下            |
| **JSONP**          | 只支持 GET，已逐渐淘汰                                    |
| **postMessage**    | 不同窗口/iframe 间通信                                    |
| **WebSocket**      | 不受同源策略限制                                          |

**CORS 简单请求与预检请求：**

非简单请求会先发 `OPTIONS` 预检，携带 `Origin`、`Access-Control-Request-Method` 等头，服务端返回允许后方可发真实请求。

满足以下条件是简单请求：

- 方法为 GET/HEAD/POST
- Content-Type 为 `application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`
- 无自定义请求头

否则浏览器会自动发送 OPTIONS 预检请求。

**面试加分点**：能解释 CORS 是浏览器的安全策略，服务端不校验时 Postman/ curl 能调通但浏览器会拦截；能说出 `withCredentials` 用于携带 Cookie。

</details>

---

### 24. 什么是重排（Reflow）和重绘（Repaint）？如何减少？

<details>
<summary>查看答案</summary>

**重排（Reflow / Layout）：**

当元素的几何属性（尺寸、位置）发生变化，浏览器需要重新计算布局。

触发属性示例：`width`、`height`、`margin`、`padding`、`top`、`left`、`offsetHeight` 等。

**重绘（Repaint）：**

当元素外观发生变化但不影响布局时，浏览器重新绘制像素。

触发属性示例：`color`、`background-color`、`box-shadow`、`border-radius`、`visibility` 等。

**关系**：重排一定会引起重绘，重绘不一定会引起重排。

**减少重排重绘的方法：**

1. **批量修改样式**：一次性修改 className，而不是逐条改 style
2. **使用 `transform` / `opacity`**：触发 GPU 加速，避开重排
3. **避免频繁读取布局属性**：如 `offsetHeight`、`scrollTop` 会强制同步布局
4. **使用 `DocumentFragment`**：离线操作 DOM 后再一次性插入
5. **使用 `will-change`**：提前告知浏览器哪些属性将变化
6. **虚拟滚动**：只渲染可视区域 DOM

**示例要点**：避免在循环里交替读取 `offsetWidth` 和写入 `style.width`（会强制同步布局），应先把最终值算好再一次性写入。

**面试加分点**：能解释 `offsetHeight`、`getBoundingClientRect()` 等会触发强制同步布局（Forced Synchronous Layout），导致性能急剧下降。

</details>

---

### 25. 手写一个 Promise.all 实现

<details>
<summary>查看答案</summary>

**核心要求：**

- 接收一个可迭代对象（如数组）
- 所有 Promise 都成功时返回结果数组
- 任意一个失败时立即 reject
- 空数组直接返回空数组的 resolved Promise

**实现思路**：

1. 返回一个新 Promise
2. 用 `Array.from` 把可迭代对象转成数组，空数组直接 `resolve([])`
3. 遍历每个元素，用 `Promise.resolve(p)` 包装（支持非 Promise 值）
4. 成功时按 `index` 写入结果，计数 `count++`，全部完成再 `resolve(result)`
5. 任一失败立即 `reject(reason)`

**关键点**：用 `result[index]` 保证顺序；用 `count` 计数而不是 `result.length`，避免稀疏数组问题；一个失败即整体失败但其他 Promise 仍会执行。

**关键点：**

- 用 `Promise.resolve(p)` 包装非 Promise 值
- 用 `result[index]` 保证结果顺序与输入一致
- `count` 计数而不是用 `result.length` 判断，避免稀疏数组问题

**面试加分点**：能手写 `Promise.race` / `Promise.allSettled`；能解释 Promise.all 中一个失败就短路，其他 Promise 仍会执行但结果不再关心。

</details>

---

### 26. 手写一个 new 操作符

<details>
<summary>查看答案</summary>

**new 操作符做的事：**

1. 创建一个空对象
2. 空对象的原型指向构造函数的 `prototype`
3. 执行构造函数，将 this 绑定到这个新对象
4. 如果构造函数返回对象，则返回该对象；否则返回新对象

**实现思路**：

1. 创建空对象：`Object.create(Constructor.prototype)`，让其原型指向构造函数原型
2. 执行构造函数并绑定 this：`Constructor.apply(obj, args)`
3. 判断返回值：若构造函数返回对象/函数则返回该值，否则返回新对象

**面试加分点**：能解释 `Object.create()` 的作用；能说出构造函数返回基本类型时会被忽略，返回对象时才会替代默认实例。

</details>

---

### 27. 手写 call、apply、bind

<details>
<summary>查看答案</summary>

**myCall / myApply 思路**：将目标函数作为 `context` 的临时方法调用，用 `Symbol` 做键避免覆盖原属性，`call` 传展开参数，`apply` 传数组，执行后删除临时属性并返回结果。

**myBind 思路**：返回一个新函数，新函数内部用 `apply` 把原函数绑定到指定 `context`，并合并预先传入的参数和调用时传入的参数。

**bind 作为构造函数**：更完整的实现需判断调用方式，若用 `new` 调用则 `this` 指向新实例，否则指向绑定的 `context`；并让 `bound.prototype` 继承原函数原型。

**面试加分点**：能解释 `bind` 返回的新函数作为构造函数时，this 应指向新实例；能说出 Symbol 键避免覆盖对象原有属性。

</details>

---

### 28. 实现数组扁平化（flat）

<details>
<summary>查看答案</summary>

**递归实现思路**：遍历数组，若元素是数组且 `depth > 0`，则递归展开后推入结果；否则直接推入。

**reduce 实现思路**：`depth > 0` 时用 `reduce` 拼接，`Array.isArray(cur)` 则递归，否则直接 `concat`。

**完全拍平**：深度为 Infinity 时持续递归，也可用栈实现——把数组元素入栈，遇到数组再展开，最终得到扁平结果。

**面试加分点**：能手写指定深度的扁平化；能解释 `Array.prototype.flat` 的默认深度是 1；栈实现适合面试展示对数据结构的掌握。

</details>

---

### 29. 实现发布订阅模式（EventEmitter）

<details>
<summary>查看答案</summary>

**核心 API：**

- `on(event, listener)`：订阅事件
- `emit(event, ...args)`：发布事件
- `off(event, listener)`：取消订阅
- `once(event, listener)`：只订阅一次

**实现思路**：内部用一个对象 `this.events` 存储事件到监听器数组的映射。

- `on`：不存在该事件则创建数组，把监听器 push 进去，可返回取消订阅函数
- `emit`：取出对应监听器数组依次执行
- `off`：过滤掉指定监听器
- `once`：用包装函数包裹原监听器，执行一次后自动 `off`

**与观察者模式的区别：**

| 特性     | 发布订阅                 | 观察者模式                         |
| -------- | ------------------------ | ---------------------------------- |
| 耦合度   | 低，通过事件中心通信     | 高，Subject 直接通知 Observer      |
| 通信方式 | 发布者和订阅者不直接认识 | 观察者直接注册到被观察者上         |
| 典型应用 | EventBus、消息队列       | Vue 2 响应式、DOM MutationObserver |

**面试加分点**：能说出 `once` 用包装函数实现；能解释发布订阅比观察者模式多了一层事件中心，解耦更彻底。

</details>

---

### 30. 前端路由的原理是什么？hash 和 history 模式有什么区别？

<details>
<summary>查看答案</summary>

**前端路由本质：**

监听 URL 变化，根据 URL 匹配对应的组件或页面，而不刷新整个页面。

**hash 模式：**

- URL 中 `#` 后面的部分变化不会触发页面刷新
- 通过 `hashchange` 事件监听
- 兼容性好，包括 IE8+

监听 `window.addEventListener('hashchange', ...)`，通过 `location.hash.slice(1)` 获取路径并渲染。

**history 模式：**

- 使用 HTML5 History API：`pushState`、`replaceState`、`popstate`
- URL 更美观，没有 `#`
- 需要服务端配置，避免刷新 404

使用 `history.pushState/replaceState` 改变 URL，监听 `popstate` 事件获取 `location.pathname` 并渲染。注意手动 `pushState` 不会触发 `popstate`。

**对比：**

| 特性       | hash 模式    | history 模式      |
| ---------- | ------------ | ----------------- |
| URL 形式   | `/#/about`   | `/about`          |
| 兼容性     | 好，IE8+     | IE10+             |
| 服务端支持 | 不需要       | 需要配置 fallback |
| SEO        | 一般         | 更好              |
| 监听事件   | `hashchange` | `popstate`        |

**面试加分点**：能解释 `pushState` 不会触发 `popstate`，只有浏览器前进后退/手动改 URL 才会触发；能说出 Nginx/Node 服务端需要返回 `index.html` 来支持 history 模式刷新。

</details>

---

## 使用说明

- 在 Cursor / VSCode / Qoder 等 Markdown 预览中，点击 **「查看答案」** 即可展开。
- 每题答案均包含：**核心要点 → 对比/原理 → 坑点 → 面试加分点**（代码已大幅精简，以口述为主）
- 若预览环境不支持 `<details>`，请使用支持 HTML 的 Markdown 阅读器查看。
