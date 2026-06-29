# Vite 面试题（含隐藏答案 · 详细版）

共 **18 题**，涵盖核心原理、与 Webpack 对比、开发服务器、构建机制、插件系统、HMR、配置实践与性能优化。点击「查看答案」即可展开，答案以**口述要点 / 对比 / 核心原理 / 坑点 / 面试加分点**为主，适合面试直接回答。

---

## 一、核心概念与 Webpack 对比

### 1. Vite 是什么？为什么比 Webpack 快？

<details>
<summary>查看答案</summary>

**一句话概括：** Vite 是基于原生 ES Modules 的下一代前端构建工具，开发环境利用浏览器原生 ESM 实现按需编译，生产环境用 Rollup 打包。

**为什么快（核心原因）：**

| 维度     | Webpack                           | Vite                                          |
| -------- | --------------------------------- | --------------------------------------------- |
| 启动方式 | 先打包整个应用，再启动 dev server | 先启动 dev server，按需编译请求的模块         |
| 模块处理 | 所有文件编译打包成 bundle         | 利用浏览器原生 ESM，源码不打包                |
| 依赖处理 | 一起打包                          | esbuild 预构建（Go 语言，比 JS 快 10-100 倍） |
| HMR      | 重新构建受影响的 chunk            | 只需让浏览器重新请求变更模块，与项目大小无关  |
| 冷启动   | O(n)，项目越大越慢                | O(1)，几乎恒定                                |

**两个关键技术：**

1. **原生 ESM**：浏览器直接发 HTTP 请求加载每个模块，dev server 只需按需编译单个文件
2. **esbuild 预构建**：将 node_modules 中的依赖（CommonJS/UMD）转为 ESM 并合并小模块，用 Go 实现，速度极快

**面试加分点**：Webpack 是「先打包后启动」，Vite 是「先启动后按需编译」——这个根本性的架构差异决定了启动速度的量级差距。项目越大，差距越明显。

</details>

---

### 2. Vite 和 Webpack 的核心区别有哪些？各自适合什么场景？

<details>
<summary>查看答案</summary>

**全方位对比表：**

| 维度       | Webpack                        | Vite                                  |
| ---------- | ------------------------------ | ------------------------------------- |
| 定位       | 通用打包器（Bundler）          | 开发构建工具（Dev + Build）           |
| 开发服务器 | 基于打包结果的 dev server      | 基于原生 ESM 的 dev server            |
| 构建工具   | 自身（JS）                     | 生产用 Rollup，预构建用 esbuild       |
| 冷启动速度 | 慢（项目大时秒级到分钟级）     | 快（通常 <1s）                        |
| HMR 速度   | 受 bundle 大小影响             | 恒定快（只替换单个模块）              |
| 配置复杂度 | 复杂（loader/plugin 体系庞大） | 简单（约定优于配置）                  |
| 生态       | 最丰富，几乎覆盖所有场景       | 快速增长，主流场景已覆盖              |
| 代码分割   | 成熟灵活                       | Rollup 支持，够用但定制稍弱           |
| 兼容性     | 可支持到 IE                    | 默认仅支持现代浏览器（ESM）           |
| CSS 处理   | 需 loader 配置                 | 内置支持 PostCSS/CSS Modules/预处理器 |
| TypeScript | 需 ts-loader/babel             | 内置 esbuild 转译（只转译不检查）     |
| 学习曲线   | 陡峭                           | 平缓                                  |

**Webpack 仍有优势的场景：**

- 需要支持老浏览器（IE11）
- 超复杂的定制打包需求（微前端、Module Federation）
- 历史大型项目迁移成本高
- 需要极其精细的 chunk 策略

**Vite 更适合的场景：**

- 新项目首选
- Vue/React/Svelte 等现代框架项目
- 追求开发体验（快启动、快 HMR）
- 库开发（Library Mode）
- SSR 项目（Nuxt 3、Astro 底层都用 Vite）

**面试加分点**：Vite 不是 Webpack 的替代品，而是针对现代浏览器环境的新方案。Webpack 5 也在改进（持久缓存、Module Federation），但架构限制了根本提速。

</details>

---

### 3. Vite 为什么开发用 ESM，生产却用 Rollup 打包？

<details>
<summary>查看答案</summary>

**开发环境用原生 ESM 的原因：**

- 不需要打包，浏览器直接按需加载
- 修改单文件只需让浏览器重新请求该模块
- 启动快、HMR 快

**生产环境不能直接用 ESM 的原因：**

1. **网络请求过多**：一个页面可能几百个模块文件，HTTP/2 下仍有性能瓶颈
2. **Tree-shaking**：需要打包工具分析依赖图才能删除死代码
3. **代码压缩**：需要统一 minify
4. **代码分割**：需要智能地拆分 chunk，实现按需加载
5. **兼容性**：部分浏览器/CDN 对 ESM 支持不完善
6. **资源处理**：图片/字体/CSS 需要统一 hash、base64 等

**为什么选 Rollup 而不是 Webpack/esbuild？**

| 选项    | 优劣                                                      |
| ------- | --------------------------------------------------------- |
| Rollup  | 输出更干净、Tree-shaking 更好、插件生态丰富、ESM 优先设计 |
| Webpack | 输出冗余大、配置复杂、生态太重                            |
| esbuild | 速度极快但代码分割和插件能力不够成熟，CSS 处理弱          |

**Vite 的分工：**

- **esbuild**：依赖预构建 + TS/JSX 转译（速度）
- **Rollup**：生产打包 + 插件体系（质量和灵活性）

**面试加分点**：Vite 未来可能用 Rolldown（Rust 版 Rollup，兼容 Rollup 插件 API）统一开发和生产构建，解决开发/生产行为不一致的问题。

</details>

---

## 二、开发服务器与预构建

### 4. Vite 的依赖预构建（Pre-Bundling）是什么？为什么需要？

<details>
<summary>查看答案</summary>

**依赖预构建**：Vite 在 dev server 启动前，用 **esbuild** 将 `node_modules` 中的第三方依赖统一转换为 ESM 格式并合并。

**为什么需要：**

1. **格式转换**：很多 npm 包只提供 CommonJS（如 lodash），浏览器不认识 `require`，必须转为 ESM
2. **合并请求**：一个库可能内部有几百个小模块文件（如 lodash-es 有 600+ 文件），不合并会导致浏览器发几百个请求，严重卡顿
3. **性能**：预构建后缓存到 `node_modules/.vite`，后续启动直接复用

**工作流程：**

1. Vite 启动时扫描源码中的 `import` 语句，找出第三方依赖
2. 用 esbuild 将这些依赖打包为单个 ESM 文件
3. 结果缓存到 `node_modules/.vite/deps`
4. 浏览器请求依赖时直接返回缓存文件
5. `package.json` 或 `lockfile` 变化时自动重新预构建

**配置项：**

```js
// vite.config.ts
export default {
  optimizeDeps: {
    include: ["lodash-es"], // 强制预构建
    exclude: ["my-linked-lib"], // 排除
    force: true, // 强制重新预构建
  },
};
```

**坑点：**

- 动态 import 的依赖首次可能未被扫描到，导致页面刷新（Vite 会自动发现并重新预构建）
- monorepo 中 linked 的本地包默认不预构建，可能需要手动 include

**面试加分点**：预构建用的是 esbuild（Go 语言），速度是 Webpack/Rollup 的 10-100 倍；预构建结果带有强缓存 HTTP 头（max-age），二次加载零请求。

</details>

---

### 5. Vite 的 HMR（热模块替换）原理是什么？和 Webpack HMR 有什么区别？

<details>
<summary>查看答案</summary>

**Vite HMR 原理：**

1. 文件修改 → Vite dev server 检测到变化
2. 分析模块依赖图，确定受影响的模块边界
3. 通过 WebSocket 通知浏览器
4. 浏览器重新请求变更的模块（带时间戳避免缓存）
5. 框架的 HMR 运行时（如 Vue/React）执行热替换

**与 Webpack HMR 的对比：**

| 维度         | Webpack HMR             | Vite HMR                              |
| ------------ | ----------------------- | ------------------------------------- |
| 更新粒度     | 重新构建受影响的 chunk  | 只让浏览器重新请求变更模块            |
| 速度影响因素 | 与 chunk 大小正相关     | 与项目大小无关，恒定快                |
| 传输内容     | 增量 patch（JSON + JS） | 完整模块文件                          |
| 实现机制     | module.hot.accept API   | import.meta.hot API                   |
| 框架集成     | 需要 loader 配合        | 框架插件内置（如 @vitejs/plugin-vue） |

**Vite HMR 快的根本原因：**

- 不需要重新打包任何 bundle
- 修改一个 `.vue` 文件，只需重新编译这一个文件
- 通过模块图精确失效，不会扩散到无关模块

**HMR 边界：**

- Vue SFC：`<script>`、`<template>`、`<style>` 可以独立热更新
- CSS：直接替换 `<style>` 标签
- 如果修改的模块没有 HMR 处理，会向上冒泡直到找到能处理的边界，最坏情况全页刷新

**面试加分点**：Webpack 的 HMR 速度随项目增大而变慢，因为要重新构建 chunk；Vite 的 HMR 速度几乎恒定，因为只处理单个模块。

</details>

---

### 6. Vite dev server 处理请求的完整流程是什么？

<details>
<summary>查看答案</summary>

**浏览器请求一个模块时的处理流程：**

1. **浏览器发起请求**：如 `GET /src/App.vue`
2. **路径解析**：Vite 将裸模块（如 `import { ref } from 'vue'`）重写为预构建路径 `/node_modules/.vite/deps/vue.js`
3. **文件转换**：
   - `.vue` → 通过 `@vitejs/plugin-vue` 编译为 JS
   - `.ts/.tsx` → 通过 esbuild 转译为 JS
   - `.css` → 转为 JS 模块（注入 `<style>` 标签）
   - `.json` → 转为 `export default {...}`
4. **注入 HMR 代码**：给模块添加 `import.meta.hot` 相关热更新逻辑
5. **返回响应**：设置正确的 `Content-Type` 和缓存头
6. **浏览器执行**：遇到 `import` 语句再发起新请求（按需加载）

**源码 vs 依赖的不同处理：**

| 类型                 | 处理方式         | 缓存策略                    |
| -------------------- | ---------------- | --------------------------- |
| 源码（src/下）       | 每次请求实时编译 | 304 协商缓存                |
| 依赖（node_modules） | 预构建后缓存     | 强缓存（max-age: 31536000） |

**面试加分点**：Vite 的核心思路是把模块转换推迟到「浏览器请求时」，而不是启动时全部编译；依赖用强缓存、源码用协商缓存，这是性能的关键。

</details>

---

## 三、构建与配置

### 7. Vite 的生产构建有哪些重要配置？

<details>
<summary>查看答案</summary>

**常用构建配置：**

```js
// vite.config.ts
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    target: 'es2015',           // 浏览器兼容目标
    outDir: 'dist',             // 输出目录
    assetsDir: 'assets',        // 静态资源目录
    sourcemap: true,            // 是否生成 sourcemap
    minify: 'terser',           // 压缩方式：'esbuild'(默认) | 'terser' | false
    cssCodeSplit: true,         // CSS 代码分割
    rollupOptions: {            // 自定义 Rollup 配置
      input: { ... },           // 多入口
      output: {
        manualChunks: { ... },  // 手动分包
        chunkFileNames: 'js/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]',
      },
    },
    lib: { ... },               // 库模式
  }
})
```

**代码分割策略：**

```js
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        'vendor': ['vue', 'vue-router', 'pinia'],
        'ui': ['element-plus'],
        'utils': ['lodash-es', 'dayjs'],
      }
    }
  }
}
```

**或用函数形式动态分包：**

```js
manualChunks(id) {
  if (id.includes('node_modules')) {
    return 'vendor'
  }
}
```

**面试加分点**：Vite 默认用 esbuild 压缩（比 terser 快 20-40 倍），需要极致压缩率时才换 terser；`cssCodeSplit: true` 让每个异步 chunk 的 CSS 独立加载，避免全量引入。

</details>

---

### 8. Vite 如何处理 CSS？支持哪些特性？

<details>
<summary>查看答案</summary>

**Vite 内置 CSS 支持（零配置）：**

| 特性         | 说明                                            |
| ------------ | ----------------------------------------------- |
| CSS 导入     | `import './style.css'` 自动注入页面             |
| CSS Modules  | 文件名 `.module.css` 自动启用，导出类名映射对象 |
| 预处理器     | 安装 sass/less/stylus 即可用，无需配置 loader   |
| PostCSS      | 项目有 `postcss.config.js` 自动应用             |
| @import 解析 | 支持别名 `@import '@/styles/vars.css'`          |
| url() 资源   | 自动解析为正确路径                              |
| CSS 代码分割 | 异步 chunk 的 CSS 自动分离                      |

**CSS Modules 用法：**

```vue
<template>
  <div :class="$style.container">...</div>
</template>

<style module>
.container {
  color: red;
}
</style>
```

或 JS 中：

```js
import styles from "./App.module.css";
// styles.container → 'App_container_1a2b3c'
```

**预处理器全局变量注入：**

```js
// vite.config.ts
css: {
  preprocessorOptions: {
    scss: {
      additionalData: `@use "@/styles/variables" as *;`;
    }
  }
}
```

**与 Webpack 的区别：**

| 维度        | Webpack                    | Vite               |
| ----------- | -------------------------- | ------------------ |
| 预处理器    | 需 sass-loader + node-sass | 只需安装 sass      |
| CSS Modules | 需配置 css-loader          | 文件名约定即可     |
| PostCSS     | 需 postcss-loader          | 放配置文件自动生效 |
| 代码分割    | MiniCssExtractPlugin       | 内置               |

**面试加分点**：Vite 对 CSS 的处理体现了「约定优于配置」的理念；开发模式下 CSS 通过 JS 注入 `<style>` 标签（实现 HMR），生产模式才抽取为独立 .css 文件。

</details>

---

### 9. Vite 的环境变量和模式（Mode）怎么用？

<details>
<summary>查看答案</summary>

**环境变量文件：**

```
.env                # 所有模式都加载
.env.local          # 所有模式，被 git 忽略
.env.development    # development 模式
.env.production     # production 模式
.env.[mode]         # 自定义模式
```

**变量命名规则：**

- 只有 `VITE_` 前缀的变量才会暴露给客户端代码
- 通过 `import.meta.env.VITE_API_URL` 访问
- `import.meta.env.MODE` / `DEV` / `PROD` / `SSR` 是内置变量

**TypeScript 类型声明：**

```ts
// env.d.ts
interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
}
```

**自定义模式：**

```bash
vite build --mode staging
# 加载 .env.staging
```

**与 Webpack 的区别：**

| 维度     | Webpack                   | Vite                       |
| -------- | ------------------------- | -------------------------- |
| 访问方式 | `process.env.VUE_APP_XXX` | `import.meta.env.VITE_XXX` |
| 前缀     | `VUE_APP_` / `REACT_APP_` | `VITE_`                    |
| 注入原理 | DefinePlugin 字符串替换   | 同（构建时静态替换）       |
| 运行时   | Node 环境变量             | 浏览器 ESM 环境            |

**坑点：**

- `.env` 文件修改后需要重启 dev server
- 不要在 `VITE_` 变量中放敏感信息（会被打包到客户端代码中）
- 服务端代码（SSR）可访问所有 `process.env`，不受 `VITE_` 前缀限制

**面试加分点**：`import.meta.env` 是 ESM 标准的元数据机制；Vite 在构建时通过静态替换将环境变量内联到代码中，不是运行时读取。

</details>

---

## 四、插件系统

### 10. Vite 的插件系统是怎样的？和 Webpack 有什么区别？

<details>
<summary>查看答案</summary>

**Vite 插件基于 Rollup 插件接口扩展：**

- 兼容大部分 Rollup 插件（生产构建时直接使用）
- 额外扩展了 Vite 特有的钩子（开发服务器相关）

**Vite 特有钩子：**

| 钩子                 | 作用                            |
| -------------------- | ------------------------------- |
| `config`             | 修改 Vite 配置                  |
| `configResolved`     | 配置解析完毕，可读取最终配置    |
| `configureServer`    | 自定义 dev server（添加中间件） |
| `transformIndexHtml` | 转换 index.html                 |
| `handleHotUpdate`    | 自定义 HMR 处理                 |

**Rollup 通用钩子（Vite 中也可用）：**

| 钩子                      | 作用               |
| ------------------------- | ------------------ |
| `resolveId`               | 自定义模块路径解析 |
| `load`                    | 自定义模块加载     |
| `transform`               | 转换模块内容       |
| `buildStart` / `buildEnd` | 构建生命周期       |

**简单插件示例：**

```js
function myPlugin() {
  return {
    name: "my-plugin",
    transform(code, id) {
      if (id.endsWith(".md")) {
        return `export default ${JSON.stringify(marked(code))}`;
      }
    },
  };
}
```

**与 Webpack 的对比：**

| 维度        | Webpack               | Vite                        |
| ----------- | --------------------- | --------------------------- |
| 插件接口    | 自有 Tapable 事件系统 | Rollup 插件接口 + 扩展      |
| 复杂度      | 高（hooks 几十个）    | 低（核心 hooks 10 个左右）  |
| Loader 概念 | 有（文件转换专用）    | 无（统一用 transform 钩子） |
| 插件数量    | 数万                  | 数千（快速增长中）          |

**面试加分点**：Vite 没有 loader 概念，文件转换统一走插件的 `transform` 钩子；插件可通过 `enforce: 'pre' | 'post'` 控制执行顺序；`apply: 'serve' | 'build'` 限定只在开发或生产生效。

</details>

---

### 11. 常用的 Vite 插件有哪些？

<details>
<summary>查看答案</summary>

**官方插件：**

| 插件                     | 作用                                       |
| ------------------------ | ------------------------------------------ |
| `@vitejs/plugin-vue`     | Vue 3 SFC 支持                             |
| `@vitejs/plugin-vue-jsx` | Vue JSX/TSX 支持                           |
| `@vitejs/plugin-react`   | React Fast Refresh + JSX                   |
| `@vitejs/plugin-legacy`  | 旧浏览器兼容（生成 ESM + legacy 双份代码） |

**常用社区插件：**

| 插件                       | 作用                                       |
| -------------------------- | ------------------------------------------ |
| `unplugin-auto-import`     | API 自动导入（ref/reactive 等无需 import） |
| `unplugin-vue-components`  | 组件自动注册（按需导入 UI 库组件）         |
| `vite-plugin-svg-icons`    | SVG 雪碧图                                 |
| `vite-plugin-compression`  | gzip/brotli 压缩                           |
| `vite-plugin-pwa`          | PWA 支持                                   |
| `vite-plugin-mock`         | Mock 数据                                  |
| `rollup-plugin-visualizer` | 打包分析可视化                             |
| `vite-plugin-pages`        | 文件路由                                   |
| `vite-plugin-inspect`      | 调试插件转换过程                           |

**unplugin 系列的优势**：一套代码同时支持 Vite、Webpack、Rollup、esbuild，跨构建工具通用。

**面试加分点**：能说出 `unplugin-auto-import` + `unplugin-vue-components` 是 Vue 3 项目的标配，大幅减少样板 import 代码。

</details>

---

## 五、高级特性

### 12. Vite 如何配置代理解决跨域？

<details>
<summary>查看答案</summary>

**代理配置：**

```js
// vite.config.ts
export default defineConfig({
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:3000",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ""),
      },
      // WebSocket 代理
      "/socket.io": {
        target: "ws://localhost:3000",
        ws: true,
      },
    },
  },
});
```

**原理**：Vite dev server 底层用 `http-proxy`，在开发服务器层面做请求转发，浏览器不感知跨域。

**与 Webpack devServer.proxy 的区别：**

| 维度      | Webpack                 | Vite                     |
| --------- | ----------------------- | ------------------------ |
| 底层库    | `http-proxy-middleware` | `http-proxy`（更轻量）   |
| 配置方式  | 几乎一致                | 几乎一致                 |
| 路径重写  | `pathRewrite` 对象      | `rewrite` 函数（更灵活） |
| WebSocket | 需单独配                | `ws: true`               |

**坑点：**

- 代理只在开发环境生效，生产环境需 Nginx 或后端 CORS
- `changeOrigin: true` 修改请求头中的 host，某些后端会校验
- 多个代理规则按配置顺序匹配，先匹配到的先处理

**面试加分点**：代理是 dev server 的功能，和 Vite/Webpack 本身的打包无关；生产环境跨域解决方案是 CORS 或 Nginx 反向代理。

</details>

---

### 13. Vite 的 Library Mode（库模式）怎么用？

<details>
<summary>查看答案</summary>

**库模式**：用 Vite 打包组件库/工具库，输出 ESM/UMD/CJS 等格式。

**配置：**

```js
// vite.config.ts
import { resolve } from "path";
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, "src/index.ts"),
      name: "MyLib", // UMD 全局变量名
      formats: ["es", "umd"], // 输出格式
      fileName: (format) => `my-lib.${format}.js`,
    },
    rollupOptions: {
      external: ["vue"], // 排除 peer dependencies
      output: {
        globals: { vue: "Vue" }, // UMD 中外部依赖的全局变量
      },
    },
  },
});
```

**输出结果：**

- `my-lib.es.js` — ES Module 格式
- `my-lib.umd.js` — UMD 格式（支持 `<script>` 引入）

**package.json 配置：**

```json
{
  "main": "./dist/my-lib.umd.js",
  "module": "./dist/my-lib.es.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.es.js",
      "require": "./dist/my-lib.umd.js"
    }
  }
}
```

**面试加分点**：库模式下 CSS 默认注入 JS；如需独立 CSS 文件可用 `vite-plugin-css-injected-by-js` 或手动处理；`external` 必须排除 peer deps 避免打包进去。

</details>

---

### 14. Vite 如何支持旧浏览器？`@vitejs/plugin-legacy` 是什么？

<details>
<summary>查看答案</summary>

**Vite 默认只支持现代浏览器**：原生支持 ESM 的浏览器（Chrome 87+、Firefox 78+、Safari 14+、Edge 88+）。

**需要兼容旧浏览器时：**

```js
import legacy from "@vitejs/plugin-legacy";

export default defineConfig({
  plugins: [
    legacy({
      targets: ["defaults", "not IE 11"], // browserslist
      additionalLegacyPolyfills: ["regenerator-runtime/runtime"],
    }),
  ],
});
```

**工作原理：**

1. 正常打包输出现代 ESM 代码
2. 额外用 `@babel/preset-env` 转译一份 legacy 代码
3. 在 HTML 中同时注入两份：
   - `<script type="module">` — 现代浏览器加载
   - `<script nomodule>` — 旧浏览器加载
4. 自动注入必要的 polyfill（core-js）

**优势（相比 Webpack 的 babel-loader）：**

- 现代浏览器不加载 polyfill，代码更小更快
- 旧浏览器才降级，互不影响
- 按需 polyfill，不一刀切

**面试加分点**：`<script type="module">` + `<script nomodule>` 是浏览器原生的差异化加载机制；现代浏览器忽略 nomodule，旧浏览器忽略 type=module。

</details>

---

## 六、性能优化与实践

### 15. Vite 项目有哪些常见的性能优化手段？

<details>
<summary>查看答案</summary>

**开发环境优化：**

| 手段         | 说明                                                    |
| ------------ | ------------------------------------------------------- |
| 预构建优化   | `optimizeDeps.include` 手动包含动态依赖，减少二次预构建 |
| 减少文件监听 | `server.watch.ignored` 忽略不必要的目录                 |
| 按需引入     | 配合 `unplugin-vue-components` 按需加载 UI 组件         |
| 减少插件     | 开发环境排除不必要插件（如压缩分析）                    |

**生产构建优化：**

| 手段         | 说明                                            |
| ------------ | ----------------------------------------------- |
| 代码分割     | `manualChunks` 合理拆分 vendor/ui/utils         |
| Tree-shaking | 确保用 ESM 格式的库（如 lodash-es 而非 lodash） |
| CSS 分割     | `cssCodeSplit: true`（默认开启）                |
| 压缩         | 默认 esbuild 压缩；极致需求用 terser            |
| 图片优化     | `vite-plugin-imagemin` 压缩图片                 |
| gzip/brotli  | `vite-plugin-compression` 预压缩                |
| 分析包体     | `rollup-plugin-visualizer` 可视化分析           |
| 动态导入     | 路由懒加载 `() => import('./Page.vue')`         |

**通用优化：**

| 手段            | 说明                                                |
| --------------- | --------------------------------------------------- |
| 路径别名        | `resolve.alias` 避免深层相对路径                    |
| TS 类型检查分离 | Vite 只转译不检查类型，检查交给 `vue-tsc` 或 IDE    |
| 缓存利用        | `build.rollupOptions.output.chunkFileNames` 含 hash |
| CDN 外置        | 大库走 CDN，`build.rollupOptions.external`          |
| 预加载          | `<link rel="modulepreload">` Vite 自动注入          |

**面试加分点**：Vite 生产打包默认自动做了很多优化（CSS 分割、JS 分块、预加载提示、资源 hash）；开发者只需关注手动分包策略和第三方库的引入方式。

</details>

---

### 16. 从 Webpack 迁移到 Vite 需要注意什么？

<details>
<summary>查看答案</summary>

**主要迁移步骤：**

1. **入口变化**：`index.html` 移到项目根目录，`<script type="module" src="/src/main.ts">`
2. **配置迁移**：`webpack.config.js` → `vite.config.ts`
3. **路径别名**：`resolve.alias` 配置方式类似但语法不同
4. **环境变量**：`process.env.VUE_APP_XXX` → `import.meta.env.VITE_XXX`
5. **全局变量**：`DefinePlugin` → `define` 配置项
6. **静态资源**：`require()` → `import` 或 `new URL('./img.png', import.meta.url)`
7. **CSS**：移除 loader 配置，Vite 内置处理
8. **插件替换**：找对应 Vite 插件或 unplugin

**常见踩坑：**

| 问题                 | 解决                                               |
| -------------------- | -------------------------------------------------- |
| `require()` 不可用   | 改为 `import` 或 `import.meta.glob`                |
| CommonJS 模块报错    | 添加到 `optimizeDeps.include`                      |
| `process.env` 不存在 | 改为 `import.meta.env`                             |
| 全局 SCSS 变量       | `css.preprocessorOptions.scss.additionalData`      |
| require.context      | 改为 `import.meta.glob` / `import.meta.globEager`  |
| HTML 模板变量        | 用 `vite-plugin-html` 或 `transformIndexHtml` 钩子 |
| Node.js 内置模块     | 浏览器环境无 `fs`/`path`，需 polyfill 或移除       |

**`import.meta.glob` 替代 `require.context`：**

```js
// Webpack
const modules = require.context("./modules", true, /\.ts$/);

// Vite
const modules = import.meta.glob("./modules/**/*.ts");
// 返回 { './modules/a.ts': () => import('./modules/a.ts') }

// 同步（eagerly）
const modules = import.meta.glob("./modules/**/*.ts", { eager: true });
```

**面试加分点**：迁移的核心难点不在配置，而在 CommonJS → ESM 的转换（require → import），以及动态加载方式的变化。

</details>

---

## 七、进阶原理

### 17. Vite 中的 esbuild 和 Rollup 各负责什么？

<details>
<summary>查看答案</summary>

**Vite 内部的分工：**

| 工具        | 负责环节                              | 优势                                      |
| ----------- | ------------------------------------- | ----------------------------------------- |
| **esbuild** | 依赖预构建、TS/JSX 转译、生产 JS 压缩 | Go 语言，极快（10-100x）                  |
| **Rollup**  | 生产打包、代码分割、插件系统          | 输出质量高、Tree-shaking 好、插件生态丰富 |

**为什么不全用 esbuild？**

| 能力         | esbuild  | Rollup               |
| ------------ | -------- | -------------------- |
| 代码分割     | 基础支持 | 成熟灵活             |
| CSS 处理     | 基础     | 插件丰富             |
| 插件生态     | 有限     | 丰富                 |
| 输出格式     | ESM/CJS  | ESM/CJS/UMD/IIFE     |
| Tree-shaking | 较好     | 更好（ESM 优先设计） |
| 自定义能力   | 弱       | 强                   |

**为什么不全用 Rollup？**

- Rollup 速度不够快（JS 写的），预构建和转译环节需要 esbuild 的极速
- 开发环境只需要转译单文件，esbuild 最合适

**未来方向：Rolldown**

Vite 团队正在开发 Rolldown（Rust 实现的 Rollup 兼容替代），目标是统一开发和生产的打包工具，解决行为不一致问题。

**面试加分点**：能说出 Vite 的工具选择策略是「让最合适的工具做最擅长的事」——esbuild 负责速度敏感环节，Rollup 负责质量和灵活性。

</details>

---

### 18. Vite 与其他新兴构建工具（Turbopack、Rspack、Farm）相比如何？

<details>
<summary>查看答案</summary>

**新兴构建工具对比：**

| 工具          | 语言                          | 定位             | 特点                                          |
| ------------- | ----------------------------- | ---------------- | --------------------------------------------- |
| **Vite**      | JS + Go(esbuild) + Rust(未来) | 开发构建工具     | ESM 开发 + Rollup 生产，生态最好              |
| **Turbopack** | Rust                          | Webpack 继任者   | Vercel 开发，Turbo 引擎增量计算，Next.js 专用 |
| **Rspack**    | Rust                          | Webpack 兼容替代 | 字节跳动开发，兼容 Webpack 生态，迁移成本低   |
| **Farm**      | Rust                          | 全 Rust 构建工具 | 极快，兼容 Vite 插件                          |
| **Rolldown**  | Rust                          | Rollup 兼容替代  | Vite 团队开发，未来替换 Vite 中的 Rollup      |

**Vite 的优势：**

1. **生态最成熟**：插件数量多、框架支持广（Vue/React/Svelte/Solid）
2. **社区最大**：文档完善、问题解决方案多
3. **上层框架采用**：Nuxt 3、Astro、SvelteKit、Remix 都基于 Vite
4. **稳定可靠**：经过大量生产验证

**Vite 的不足：**

1. 开发和生产行为不完全一致（ESM dev vs Rollup build）
2. 大型 monorepo 中首次加载可能慢（太多模块请求）
3. 预构建有时需要手动调整

**面试加分点**：工具选择看团队和场景——新项目首选 Vite（生态好）；Webpack 历史项目想提速可选 Rspack（兼容迁移）；Next.js 项目看 Turbopack 发展。Vite 的 Rolldown 计划是为了统一开发/生产，解决当前最大痛点。

</details>

---

## 使用说明

- 在 Cursor / VSCode / Qoder 等 Markdown 预览中，点击 **「查看答案」** 即可展开。
- 每题答案均包含：**核心要点 → 对比/原理 → 坑点 → 面试加分点**（代码已精简，以口述为主）
- 若预览环境不支持 `<details>`，请使用支持 HTML 的 Markdown 阅读器查看。
