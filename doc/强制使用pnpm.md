要强制项目**只能使用 pnpm** 安装依赖、执行脚本（禁止 npm/yarn），可以通过以下 3 种核心方式实现（从简单到严格，推荐组合使用），确保团队协作或个人开发时统一包管理器：

### 一、基础方案：`packageManager` 字段（corepack）

在 `package.json` 中添加 `packageManager` 字段，明确指定项目的包管理器及版本，并启用corepack，则在项目中不允许使用除此之外的包管理器

#### 1. 配置步骤

在你的 `package.json` 中新增顶层字段（与 `dependencies` 同级）：

```json
{
  "name": "my-react-router-app",
  "private": true,
  "type": "module",
  // 关键配置：指定只能用 pnpm@指定版本，必须x.y.z
  "packageManager": "pnpm@9.0.0",
  "scripts": { /* ... 原有脚本 ... */ },
  "dependencies": { /* ... 原有依赖 ... */ },
  "devDependencies": { /* ... 原有依赖 ... */ }
}
```

#### 2. 效果

- 除pnpm不允许使用

#### PS

需启用corepack： 	[前端 - 包管理工具之npm也慌了？ - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000044667607)

```bash
#开启
corepack enable
corepack enable npm #npm需要单独开启，不建议开启

#关闭
corepack disable
corepack disable npm #npm需要单独关闭，不建议开启，如果开启了，关闭也无法还原初始状态，而是删除了corepack里面的链接
```

packageManagerStrict

* 默认值：**true**
* 类型：**Boolean**

当此设置被禁用时，如果 pnpm 的版本与 `package.json` 的 `packageManager` 字段中指定的版本不匹配，则 pnpm 不会失败。 启用后，仅检查包名称（自 pnpm v9.2.0 起），因此无论 `packageManager` 字段中指定的版本如何，你仍然可以运行任何版本的 pnpm。

或者，你可以将环境变量 `COREPACK_ENABLE_STRICT` 设置为 `0` 来禁用这个设置。

packageManagerStrictVersion

* 默认值： **false**
* 类型：**Boolean**

启用后，如果 pnpm 的版本与 `package.json`的 `packageManager<span> </span>`字段中指定的版本不完全匹配，则 pnpm 将失败。

managePackageManagerVersions

* 默认值：**true**
* 类型：**Boolean**

启用后，pnpm 将自动下载并运行 `package.json` 的 `packageManager` 字段中指定的 pnpm 版本。 这与 Corepack 使用的字段相同。 示例：

```json
{
"packageManager":"pnpm@9.3.0"
}
```



### 二、严格方案：`preinstall` 钩子（禁止非 pnpm 执行安装）

通过 `package.json` 的 `scripts.preinstall` 钩子，在执行 `install` 命令前强制校验包管理器，**彻底阻止 npm/yarn 安装依赖**（即使用户没安装 pnpm，也会报错提示）。

#### 1. 配置步骤

在 `package.json` 的 `scripts` 中添加 `preinstall` 脚本：

```json
{
  "scripts": {
    "preinstall": "node -e \"if(process.env.npm_execpath.indexOf('pnpm') === -1) throw new Error('❌ 请使用 pnpm 安装依赖！\\n 安装指南：https://pnpm.io/zh/installation')\"",
    "build": "react-router build",
    "dev": "react-router dev",
    "start": "react-router-serve ./build/server/index.js",
    "typecheck": "react-router typegen && tsc"
  }
}
```

#### 2. 原理

- `preinstall` 是 npm/yarn/pnpm 都支持的「前置钩子」，执行 `npm install`/`yarn install`/`pnpm install` 前会先运行该脚本；
- 脚本通过 `process.env.npm_execpath` 检测当前包管理器的执行路径：
  - pnpm 的路径包含 `pnpm` 字符串，npm 包含 `npm`，yarn 包含 `yarn`；
  - 若不是 pnpm，直接抛出错误，终止安装流程。

#### 3. 效果

- 用 `npm install` 会报错：
  ```
  ❌ 请使用 pnpm 安装依赖！
   安装指南：https://pnpm.io/zh/installation
  ```
- 用 `yarn install` 同样报错，只有 `pnpm install` 能正常执行。

### 三、终极方案：`pnpm-workspace.yaml` + `only-allow`（推荐组合）

如果需要更灵活的校验逻辑（如支持多工作区、自定义错误提示），可以使用 `only-allow` 工具（pnpm 官方推荐的包管理器校验工具），配合 `pnpm-workspace.yaml` 强化约束。

#### 1. 安装依赖

先通过 pnpm 安装 `only-allow`（作为 `devDependencies`）：

```bash
pnpm add -D only-allow
```

#### 2. 配置 `preinstall` 脚本

修改 `package.json` 的 `preinstall` 脚本，使用 `only-allow` 直接指定 pnpm：

```json
{
  "scripts": {
    "preinstall": "only-allow pnpm", // 核心：强制只能用 pnpm
    "build": "react-router build",
    "dev": "react-router dev",
    "start": "react-router-serve ./build/server/index.js",
    "typecheck": "react-router typegen && tsc"
  },
  "devDependencies": {
    // 新增 only-allow 依赖（安装后自动出现）
    "only-allow": "^1.2.1",
    /* ... 原有 devDependencies ... */
  }
}
```

#### 3. （可选）添加 `pnpm-workspace.yaml`

如果项目是单包工程，创建空的 `pnpm-workspace.yaml` 文件（放在项目根目录），进一步明确这是 pnpm 项目：

```yaml
# pnpm-workspace.yaml（单包工程无需配置其他内容，仅作为标识）
packages:
  - "." # 表示当前目录为工作区根包
```

#### 4. 效果

- 比自定义 `preinstall` 脚本更简洁，错误提示更友好：
  ```
  ERROR: This project requires using pnpm as package manager for consistency.
  Please install pnpm and try again: https://pnpm.io/installation
  ```
- 自动屏蔽 `npm install`/`yarn install`，且支持 pnpm 所有版本的校验。

### 四、补充：防止用户绕过约束（可选）

为了避免有人手动删除 `preinstall` 脚本或 `packageManager` 字段，可以添加以下辅助措施：

#### 1. 添加 `.npmrc` 配置（禁止 npm 执行脚本）

在项目根目录创建 `.npmrc` 文件，禁止 npm 运行 `preinstall` 钩子的绕过参数：

```ini
# .npmrc
ignore-scripts=false # 强制启用脚本（禁止用户用 npm install --ignore-scripts 绕过）
```

#### 2. 提交配置文件到 Git

将以下文件添加到 Git 跟踪，确保团队成员同步约束：

```bash
# 确保这些文件在 .gitignore 中未被忽略
git add package.json pnpm-workspace.yaml .npmrc
git commit -m "feat: 强制使用 pnpm"
```

#### 3. （进阶）使用 `husky` 拦截 Git 提交（防止配置被篡改）

如果需要严格控制，可通过 `husky` 在 Git 提交前校验 `package.json` 中的 `packageManager` 字段：

```bash
# 安装 husky
pnpm add -D husky
npx husky install
npx husky add .husky/pre-commit "node -e \"const pkg = require('./package.json'); if(!pkg.packageManager?.startsWith('pnpm')) throw new Error('❌ package.json 必须指定 pnpm 为包管理器！')\""
```

这样如果有人修改 `packageManager` 字段为非 pnpm，会被 Git 提交拦截。

### 五、最终配置汇总（推荐组合）

1. `package.json` 中添加 `packageManager` 字段 + `preinstall` 脚本 + `only-allow` 依赖；
2. 根目录创建 `pnpm-workspace.yaml` 和 `.npmrc`；

最终关键文件内容：

- `package.json`（核心部分）：
  ```json
  {
    "packageManager": "pnpm@^9.0.0",
    "scripts": {
      "preinstall": "only-allow pnpm",
      /* ... 其他脚本 ... */
    },
    "devDependencies": {
      "only-allow": "^1.2.1",
      /* ... 其他依赖 ... */
    }
  }
  ```
- `pnpm-workspace.yaml`：
  ```yaml
  packages: [ "." ]
  ```
- `.npmrc`：
  ```ini
  ignore-scripts=false
  ```

### 如何使用（给团队成员的指引）

1. 安装 pnpm（若未安装）：
   ```bash
   # 全局安装 pnpm
   npm install -g pnpm
   # 或用 corepack 启用（Node.js 16.13+ 内置）
   corepack enable pnpm
   ```
2. 安装依赖（只能用以下命令）：
   ```bash
   pnpm install
   ```
3. 执行脚本（与 npm 用法一致，只是把 `npm` 换成 `pnpm`）：
   ```bash
   pnpm dev       # 开发环境
   pnpm build     # 构建生产版本
   pnpm start     # 启动生产服务器
   pnpm typecheck # 类型校验
   ```

通过以上配置，即可强制项目只能使用 pnpm，避免因包管理器不一致导致的依赖冲突、锁文件混乱等问题。
