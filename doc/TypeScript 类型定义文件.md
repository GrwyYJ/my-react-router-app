# 全面详解 TypeScript 类型定义文件（.d.ts）

`.d.ts`（Type Definition File，简称“类型定义文件”）是 TypeScript 生态的核心组成部分，本质是**仅包含类型声明、不包含任何运行时代码的“类型描述文件”**。它的核心作用是为 TypeScript 编译器提供“类型信息”，解决 TS 对非 TS 代码（如 JavaScript 库、Node.js 环境、浏览器 API）的类型识别问题，同时实现项目内类型共享、智能提示和编译时类型校验。

## 一、.d.ts 的核心定位与价值

### 1. 核心定位

- 不包含任何可执行逻辑（无函数体、无变量赋值、无副作用代码），仅描述“代码的类型结构”；
- 是 TypeScript 与 JavaScript 生态的“桥梁”——让 TS 能理解 JS 代码的类型，也让 TS 项目的类型可被外部复用；
- 编译时生效（仅用于 TS 类型校验），不会被转译为 JavaScript 代码（最终产物中无 `.d.ts` 相关内容）。

### 2. 核心价值

| 价值点   | 具体说明                                                                   |
| -------- | -------------------------------------------------------------------------- |
| 类型识别 | 让 TS 识别 JS 库、Node.js 内置模块、浏览器 API 等无类型信息的代码的类型；  |
| 类型校验 | 编译时检查类型错误（如参数类型不匹配、访问不存在的属性），避免运行时错误； |
| 智能提示 | 编辑器（VS Code 等）基于 `.d.ts` 提供代码补全、类型提示、文档说明；      |
| 类型共享 | 抽取项目公共类型（接口、类型别名），供多个文件复用，避免重复定义；         |
| 生态兼容 | 让 TS 项目能无缝使用海量 JavaScript 第三方库（无需等待库作者迁移到 TS）。  |

## 二、.d.ts 的适用场景

### 1. 为 JavaScript 库补充类型（最常见）

绝大多数第三方库（如 `lodash`、`axios`、`jquery`）是用 JavaScript 编写的，本身没有类型信息。通过 `.d.ts` 文件，TS 能识别这些库的导出内容（函数、变量、类型）：

- 若库自带类型：现代库（如 `vite`、`react`、`vue`）会在 `node_modules/[库名]/` 目录下内置 `.d.ts` 文件（如 `vite/client.d.ts`）；
- 若库无自带类型：TypeScript 社区通过 `@types/[库名]` 包提供类型定义（托管在 [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) 仓库），如 `@types/lodash`、`@types/node`。

### 2. 声明全局类型/变量（项目内全局复用）

在 TS 项目中，部分类型或变量需要全局可用（无需 `import` 导入），可通过 `.d.ts` 声明：

- 全局变量：如 Node.js 的 `process`、浏览器的 `window` 扩展、项目全局配置；
- 全局类型：如 `Nullable<T>`、`ApiResponse<T>` 等通用类型别名/接口；
- 全局函数：如项目全局工具函数（`formatDate`、`deepClone`）。

### 3. 项目内类型共享（模块化复用）

将项目中重复使用的类型（如接口、类型别名、枚举）抽取到 `.d.ts` 文件中，供多个模块导入复用，保持类型一致性：

- 示例：`src/types/api.d.ts` 声明所有接口请求/响应类型，`src/types/common.d.ts` 声明通用工具类型。

### 4. 类型扩展与模块增强

对已有库或内置类型进行“扩展”（不修改原代码），比如：

- 扩展 `Express` 的 `Request` 对象，添加 `user` 属性；
- 扩展 `Array` 原型，添加自定义方法的类型；
- 扩展 `Vite` 的 `ImportMetaEnv`，添加自定义环境变量类型。

## 三、.d.ts 的来源（3 种核心方式）

### 1. 库自带（推荐，零配置）

现代主流库（`vite`、`react`、`vue`、`axios@1.x+`）会在发布时自带 `.d.ts` 文件，安装后无需额外配置，TS 自动识别：

- 示例：安装 `vite` 后，`node_modules/vite/dist/client.d.ts` 提供 `vite/client` 相关类型（如 `import.meta.env`）。

### 2. 社区提供（`@types/[库名]`，最常用）

对于无自带类型的老库（如 `lodash`、`jquery`、`node` 环境），TypeScript 社区维护的 `@types/[库名]` 包提供类型定义：

- 安装方式：作为开发依赖安装（仅编译时用）：
  ```bash
  pnpm add -D @types/node  # Node.js 环境类型
  pnpm add -D @types/lodash # lodash 库类型
  pnpm add -D @types/express # Express 框架类型
  ```
- 特点：由 DefinitelyTyped 社区统一维护，版本与原库同步，覆盖度极高。

### 3. 手动编写（项目自定义，灵活可控）

根据项目需求手动创建 `.d.ts` 文件，声明项目专属类型。适用于：

- 项目内全局类型共享；
- 为自定义 JS 模块补充类型；
- 扩展已有类型（如 `window`、`ImportMetaEnv`）。

## 四、.d.ts 的核心语法（带实际示例）

`.d.ts` 的语法核心是“类型声明”，不包含任何实现逻辑。以下是最常用的语法场景，结合项目实际需求说明：

### 1. 声明全局变量/常量（`declare var/let/const`）

用于声明全局可用的变量/常量（如 Node.js 的 `process`、浏览器的 `window` 上的自定义属性）：

```typescript
// src/types/global.d.ts
declare const NODE_ENV: 'development' | 'production'; // 全局常量（只读）
declare let appVersion: string; // 全局变量（可修改）
declare var window: Window & { appConfig: { baseUrl: string } }; // 扩展 window 对象
```

- 说明：`declare` 告诉 TS“该变量已存在于运行环境中（如 Node.js/浏览器），无需编译生成 JS”。

### 2. 声明全局函数（`declare function`）

用于声明全局可用的函数（如自定义全局工具函数）：

```typescript
// src/types/global.d.ts
/**
 * 格式化日期
 * @param date 日期对象或字符串
 * @param format 格式字符串（如 'YYYY-MM-DD'）
 * @returns 格式化后的日期字符串
 */
declare function formatDate(date: Date | string, format?: string): string;

// 重载函数声明（支持多参数类型）
declare function add(a: number, b: number): number;
declare function add(a: string, b: string): string;
```

- 说明：仅声明函数签名（参数类型、返回值类型、文档注释），无函数体（实现逻辑在其他 JS 文件中）。

### 3. 声明接口/类型别名（`interface/type`）

用于共享类型结构（项目内最常用场景），`.d.ts` 中全局接口/类型可省略 `declare`：

```typescript
// src/types/common.d.ts
// 接口（可扩展）
interface User {
  id: number;
  name: string;
  age?: number; // 可选属性
  readonly createdAt: Date; // 只读属性
}

// 类型别名（支持联合、交叉等复杂类型）
type Nullable<T> = T | null; // 可空类型
type ApiResponse<T> = {
  code: number;
  data: T;
  msg: string;
};
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE'; // 联合类型
```

- 使用：其他文件无需导入，直接使用（需确保 `.d.ts` 被 `tsconfig.json` 包含）：
  ```typescript
  const user: User = { id: 1, name: '张三', createdAt: new Date() };
  const response: ApiResponse<User> = { code: 200, data: user, msg: 'success' };
  const method: HttpMethod = 'GET';
  ```

### 4. 声明模块类型（`declare module`）

#### 场景 1：为 JS 模块补全类型

对无自带类型的 JS 模块（如自定义工具库），声明其导出内容：

```typescript
// src/types/utils.d.ts
// 声明名为 "utils-js" 的 JS 模块的类型
declare module 'utils-js' {
  export function deepClone<T>(obj: T): T; // 导出函数
  export const version: string; // 导出变量
  export interface Options {
    debug: boolean;
  } // 导出类型
}
```

- 使用：导入模块时，TS 自动识别类型：
  ```typescript
  import { deepClone, Options } from 'utils-js';
  const obj = deepClone({ a: 1 });
  const opt: Options = { debug: true };
  ```

#### 场景 2：模块增强（扩展已有模块类型）

对已有模块（如 `vite`、`express`）的类型进行扩展：

```typescript
// src/types/vite-env.d.ts
// 扩展 Vite 的 ImportMetaEnv 类型（添加自定义环境变量）
interface ImportMetaEnv {
  readonly VITE_API_BASE: string;
  readonly VITE_APP_TITLE: string;
}

// 扩展 Vite 的 ImportMeta 类型（确保环境变量类型生效）
interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

- 说明：无需重新声明整个模块，仅添加新类型即可（TS 自动合并原有类型）。

#### 场景 3：声明非 JS 模块类型（如 CSS、图片）

TS 默认不识别非 JS/TS 文件（如 `.css`、`.png`），需通过 `declare module` 声明其类型：

```typescript
// src/types/asset.d.ts
// 声明 CSS 模块类型（支持 CSS Modules）
declare module '*.css' {
  const classes: { [key: string]: string };
  export default classes;
}

// 声明图片资源类型
declare module '*.png' {
  const src: string;
  export default src;
}

declare module '*.svg' {
  import React from 'react';
  export const ReactComponent: React.FC<React.SVGProps<SVGSVGElement>>;
  const src: string;
  export default src;
}
```

- 作用：导入 `import './index.css'` 或 `import logo from './logo.png'` 时，TS 不报错且能识别类型。

### 5. 声明命名空间（`declare namespace`）

早期 TS 用于组织全局类型的语法（类似模块），现在更推荐用 ES 模块（`import/export`），但旧项目可能遇到：

```typescript
// src/types/legacy.d.ts
declare namespace Utils {
  function formatDate(date: Date): string;
  type Status = 'success' | 'fail';
  interface User {
    id: number;
  }
}
```

- 使用：
  ```typescript
  const status: Utils.Status = 'success';
  const dateStr = Utils.formatDate(new Date());
  ```

### 6. 声明枚举（`enum`）

全局枚举类型（`.d.ts` 中可省略 `declare`）：

```typescript
// src/types/enum.d.ts
enum OrderStatus {
  PENDING = 'pending',
  PAID = 'paid',
  SHIPPED = 'shipped',
  DELIVERED = 'delivered',
}
```

- 使用：直接在项目中使用 `OrderStatus.PAID`，TS 提供类型校验和智能提示。

## 五、.d.ts 的配置与识别（关键！）

TS 编译器默认不会自动识别所有 `.d.ts` 文件，需通过 `tsconfig.json` 配置确保识别：

### 1. 核心配置项（`tsconfig.json`）

```json
{
  "compilerOptions": {
    // 1. 类型根目录：TS 优先从这些目录查找类型定义（默认包含 node_modules/@types）
    "typeRoots": ["node_modules/@types", "src/types"],
    // 2. 明确指定要加载的类型库（可选，按需配置）
    "types": ["node", "vite/client", "jest"],
    // 3. 模块解析方式：必须设为 Node（确保能找到 node_modules 中的类型）
    "moduleResolution": "Node",
    // 4. 允许 TS 识别 JSON 模块（可选，避免导入 JSON 报错）
    "resolveJsonModule": true,
    // 5. 目标 ES 版本（需兼容项目依赖的类型）
    "target": "ES2018",
    // 6. 生成类型定义文件（若项目是库，需开启，供外部使用）
    "declaration": true,
    // 7. 类型定义文件输出目录（与 declaration 配套）
    "declarationDir": "dist/types"
  },
  // 8. 指定要编译的文件/目录（确保 .d.ts 被包含）
  "include": ["src/**/*", "src/types/**/*.d.ts"],
  // 9. 排除不需要编译的文件
  "exclude": ["node_modules", "dist"]
}
```

### 2. 关键配置说明

- `typeRoots`：指定 TS 查找类型定义的目录，默认包含 `node_modules/@types`（社区 `@types/xxx` 包的位置），需添加项目自定义类型目录（如 `src/types`）；
- `types`：明确指定要加载的类型库（如 `node` 对应 `@types/node`），若不配置则加载 `typeRoots` 下所有类型；
- `include`：必须包含 `.d.ts` 文件所在目录（如 `src/types/**/*.d.ts`），否则 TS 无法识别；
- `declaration: true`：若项目是库（需对外提供类型），TS 会自动为 `.ts` 文件生成对应的 `.d.ts` 文件，输出到 `declarationDir`。

## 六、.d.ts 的常见问题与排查

### 1. 错误：“找不到 xxx 的类型定义文件”

- 原因：缺少对应的 `.d.ts` 文件（未安装 `@types/xxx`，或库无自带类型）；
- 解决：
  1. 安装社区类型包：`pnpm add -D @types/xxx`；
  2. 若无社区类型：手动创建 `.d.ts` 文件，用 `declare module 'xxx'` 声明空模块（临时规避）；
  3. 检查 `tsconfig.json` 的 `typeRoots` 和 `include` 是否包含 `.d.ts` 目录。

### 2. 错误：“类型 xxx 已存在”

- 原因：同一类型在多个 `.d.ts` 文件中重复声明（TS 类型合并冲突）；
- 解决：
  1. 统一类型声明到一个 `.d.ts` 文件；
  2. 若需扩展类型：使用接口合并（`interface` 可重复声明自动合并），避免重复定义类型别名（`type` 不可重复）。

### 3. .d.ts 类型不生效/无智能提示

- 原因：
  1. `.d.ts` 文件未被 `tsconfig.json` 的 `include` 包含；
  2. `moduleResolution` 未设为 `Node`；
  3. 编辑器缓存（如 VS Code 未重新加载类型）；
- 解决：
  1. 检查 `tsconfig.json` 配置，确保 `.d.ts` 路径被包含；
  2. 执行 `npx tsc --clearCache` 清除 TS 缓存；
  3. 重启 VS Code，或执行“开发者：重新加载窗口”。

### 4. 运行时“xxx is not defined”

- 原因：`declare` 声明的变量/函数在运行环境中实际不存在（`declare` 仅告诉 TS 类型，不创建实际代码）；
- 解决：确保被声明的实体（如 `process`、`formatDate`）确实存在于运行环境中（如安装 Node.js、引入对应的 JS 文件）。

## 七、.d.ts 的最佳实践

### 1. 目录结构规范

建议在项目根目录创建 `src/types` 目录，按功能分类存放 `.d.ts` 文件：

```
src/
├── types/
│   ├── global.d.ts       # 全局类型（window、全局变量/函数）
│   ├── common.d.ts       # 通用类型（接口、类型别名）
│   ├── api.d.ts          # 接口请求/响应类型
│   ├── asset.d.ts        # 资源类型（CSS、图片）
│   ├── vite-env.d.ts     # 框架/工具类型扩展（如 Vite、Express）
│   └── enum.d.ts         # 枚举类型
└── ... 其他源码目录
```

### 2. 优先使用库自带类型

- 安装第三方库时，先检查是否自带 `.d.ts`（查看 `node_modules/[库名]` 下是否有 `.d.ts` 文件）；
- 优先使用库自带类型，而非 `@types/[库名]`（避免版本不兼容）。

### 3. 避免过度声明

- 仅为“无类型的实体”（JS 库、全局变量）编写 `.d.ts`；
- TS 源码中已定义的类型（如项目内 `.ts` 文件的接口），无需重复在 `.d.ts` 中声明。

### 4. 为库项目生成类型定义

若项目是第三方库（需对外提供），开启 `tsconfig.json` 的 `declaration: true` 和 `declarationDir: "dist/types"`，编译时自动生成 `.d.ts` 文件，与 JS 产物一起发布。

## 八、总结

`.d.ts` 是 TypeScript 生态的“类型基础设施”，核心是“描述类型、不包含逻辑”，解决了 TS 对 JS 生态的兼容、项目类型共享、智能提示和类型校验四大问题。

关键要点：

1. 来源：库自带、社区 `@types/xxx`、手动编写；
2. 语法：核心是 `declare`（声明已有实体类型）、`interface/type`（共享类型）、`declare module`（模块类型）；
3. 配置：需通过 `tsconfig.json` 的 `typeRoots`、`include` 确保 TS 识别；
4. 核心价值：让 TypeScript 能无缝对接 JS 生态，同时提升项目代码质量和开发效率。

掌握 `.d.ts` 的使用，是 TypeScript 项目规范化、工程化的关键一步～
