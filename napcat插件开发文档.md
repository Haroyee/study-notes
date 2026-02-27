# napcat插件开发文档

# 插件机制原理

字数

1657 字

阅读时间

7 分钟

在开始开发 NapCat 插件之前，了解插件系统的运作机制有助于更合理地设计和编写插件。

## 插件是什么？

NapCat 插件本质上是一个 **TypeScript/JavaScript 模块**，它被 NapCat 的插件管理器加载后，通过生命周期钩子与 NapCat 主程序进行交互。

插件的工作方式可以概括为：

> **NapCat 加载你的插件 → 传给你一个上下文对象 `ctx` → 你通过 `ctx` 调用 NapCat 提供的各种能力**

## 核心架构

整个插件系统由三个关键部分组成：

NapCat Core
(QQ 底层)OneBot11
Adapter插件管理器
(加载/卸载插件)NapCatPluginContext
(插件上下文对象 ctx)你的插件plugin_init(ctx) ← 初始化时收到 ctx
plugin_onmessage(ctx) ← 消息到来时使用 ctx 处理
plugin_onevent(ctx) ← 事件到来时使用 ctx 处理
plugin_cleanup(ctx) ← 卸载时使用 ctx 清理传递给

## ctx 能做什么？

`ctx`（即 `NapCatPluginContext`）是插件与 NapCat 交互的唯一桥梁。它提供了 **三个层次** 的能力：

### 层次一：调用 OneBot Action（最常用）

通过 `ctx.actions.call(...)` 调用标准的 OneBot11 协议 Action，这是最主要的使用方式。



```
// 发送消息
await ctx.actions.call('send_msg', params, ctx.adapterName, ctx.pluginManager.config);

// 获取群列表
await ctx.actions.call('get_group_list', void 0, ctx.adapterName, ctx.pluginManager.config);

// 撤回消息
await ctx.actions.call('delete_msg', { message_id }, ctx.adapterName, ctx.pluginManager.config);
```

这些 Action 就是 NapCat 对外暴露的 OneBot11 API（如 `send_msg`、`get_group_list`、`set_group_ban` 等），与通过 HTTP/WebSocket 调用的 API 完全一致，只是插件内部可以直接调用，无需走网络请求。

API 查阅方式

- **使用 AI 查询**：官方插件模板 [napcat-plugin-template](https://github.com/NapNeko/napcat-plugin-template) 的 `.vscode/mcp.json` 中已预配置了 [napcat.apifox.cn](https://napcat.apifox.cn/) 的 MCP 服务器。使用 VS Code + GitHub Copilot 打开项目后，可以直接在 Copilot Chat 中用自然语言查询 OneBot 和 NapCat 提供的各种 API 接口。
- **手动查阅**：也可以直接访问 https://napcat.apifox.cn/ 浏览完整的 API 文档。

### 层次二：使用插件框架能力

`ctx` 还提供了一系列插件框架本身的能力，这些与 OneBot 协议无关：

| 能力     | 通过                              | 用途                                   |
| :------- | :-------------------------------- | :------------------------------------- |
| 路由注册 | `ctx.router`                      | 注册 HTTP API、静态文件、WebUI 页面    |
| 日志记录 | `ctx.logger`                      | 统一格式的日志输出                     |
| 数据存储 | `ctx.dataPath` / `ctx.configPath` | 插件的数据和配置持久化                 |
| 配置面板 | `ctx.NapCatConfig`                | 在 WebUI 生成配置界面                  |
| 插件管理 | `ctx.pluginManager`               | 管理其他插件（查询、启停、加载）       |
| 插件通信 | `ctx.getPluginExports()`          | 获取其他插件的导出对象，实现插件间交互 |

### 层次三：访问 NapCat 底层（高级用法）

`ctx` 还暴露了 NapCat 的内部实例，允许执行标准 OneBot 协议之外的操作：

- `ctx.core` — NapCat 底层核心实例
- `ctx.oneBot` — OneBot11 适配器实例

注意

直接使用 `ctx.core` 和 `ctx.oneBot` 属于高级用法，这些内部 API 可能在版本升级时发生变化。一般情况下优先使用 `ctx.actions.call(...)` 来调用标准 OneBot Action。

## 插件的两种典型用途

根据你要实现的功能，NapCat 插件大致可以分为两种模式（或两者结合）：

### 模式一：Bot 功能型插件

这是最常见的模式。**监听消息/事件 → 调用 OneBot Action 做出响应。**



```
// 典型流程：收到消息 → 处理 → 调用 OneBot Action 回复
export const plugin_onmessage = async (ctx, event) => {
    if (event.raw_message === 'ping') {
        // 通过 ctx.actions 调用 OneBot Action 发送消息
        await ctx.actions.call('send_msg', {
            message: 'pong',
            message_type: event.message_type,
            group_id: String(event.group_id),
        }, ctx.adapterName, ctx.pluginManager.config);
    }
};
```

适用场景：关键词回复、命令处理、定时任务、消息转发、群管理等。

### 模式二：WebUI 功能型插件

不处理任何 QQ 消息，**仅通过 `ctx.router` 注册路由和页面**，在 NapCat 的 WebUI 中提供额外的管理界面或工具。



```
export const plugin_init = async (ctx) => {
    // 注册一个 WebUI 页面
    ctx.router.page({
        path: 'dashboard',
        title: '数据面板',
        htmlFile: 'webui/index.html',
    });

    // 注册 API
    ctx.router.getNoAuth('/stats', (req, res) => {
        res.json({ code: 0, data: { visits: 42 } });
    });
};
```

适用场景：数据可视化面板、配置管理工具、系统监控等。

### 模式三：两者结合

实际开发中，很多插件会 **同时** 使用这两种模式 —— 处理消息的同时提供一个 WebUI 管理界面。官方插件模板 [napcat-plugin-template](https://github.com/NapNeko/napcat-plugin-template) 就是这种模式的典型示例。

关于插件模板

[napcat-plugin-template](https://github.com/NapNeko/napcat-plugin-template) 会随着 NapCat 项目的更新而持续调整（如 API 变化、最佳实践优化等）。建议开发前以模板仓库的最新版本为准，并关注仓库的 Release 和 README 以了解变更。

## 生命周期概览

NapCat 通过调用你导出的特定函数来驱动插件运行：

NapCat 启动扫描 plugins 目录加载插件模块调用 plugin_init(ctx)\n初始化：注册路由、加载配置等开始监听消息/事件plugin_onmessage(ctx, event)\n消息到来plugin_onevent(ctx, event)\n其他事件到来插件卸载/重载时plugin_cleanup(ctx)\n清理：释放资源、停止定时器等

## 与外部 OneBot 框架的区别

既然可以通过 HTTP/WebSocket 调用 OneBot API，为什么还需要插件？以下是两种方式的对比：

| 对比项     | 外部 OneBot 客户端 | NapCat 插件              |
| :--------- | :----------------- | :----------------------- |
| 通信方式   | 网络请求 (HTTP/WS) | 直接函数调用             |
| 部署方式   | 独立进程/独立服务  | 嵌入 NapCat 内部         |
| 性能       | 有网络开销         | 零网络开销，直接调用     |
| 能力范围   | 仅限 OneBot 协议   | OneBot + NapCat 内部 API |
| WebUI 集成 | 需要独立搭建前端   | 直接嵌入 NapCat WebUI    |
| 用户安装   | 需要额外配置       | 放入 plugins 目录即可    |

综上，插件是一种更轻量、集成度更高的开发方式，适用于需要与 NapCat WebUI 联动或对性能有较高要求的场景。

## 下一步

- [快速开始开发插件](https://napneko.github.io/develop/plugin/)
- [项目结构与最佳实践](https://napneko.github.io/develop/plugin/structure)
- [热重载开发](https://napneko.github.io/develop/plugin/hot-reload) — 改代码即时生效，提升开发效率
- [插件 API 参考](https://napneko.github.io/develop/plugin/api/doc)

# NapCat 插件开发概览

字数

2659 字

阅读时间

12 分钟

NapCat 提供了一套灵活的插件系统，支持开发者扩展 Bot 的功能。本文档介绍 NapCat 插件的开发流程。

前置知识

如果你还不了解 NapCat 插件系统的工作原理，建议先阅读 [插件机制原理](https://napneko.github.io/develop/plugin/mechanism)。

说明

推荐使用 [napcat-plugin-template](https://github.com/NapNeko/napcat-plugin-template) 模板仓库快速开始开发。

## 开发前准备

在开始开发之前，你需要了解以下资源：

- **[napcat-plugin-template](https://github.com/NapNeko/napcat-plugin-template)**: 官方提供的插件开发模板，内置了构建流程和 CI/CD，推荐使用。
- **[napcat-plugin-index](https://github.com/NapNeko/napcat-plugin-index)**: 插件索引仓库，发布后的插件会收录在这里。

## 快速开始

### 1. 使用模板创建项目

最简单的方式是直接通过 [napcat-plugin-template](https://github.com/NapNeko/napcat-plugin-template) 创建你的 GitHub 仓库。该模板提供了完整的开发环境配置，包括 TypeScript 支持、构建脚本以及 CI/CD 流程。

### 2. 项目结构说明

一个标准的 NapCat 插件项目结构如下 (以模板为例)：



```
napcat-plugin-template/
├── src/
│   ├── index.ts              # 插件主入口，负责导出生命周期函数
│   ├── config.ts             # 配置文件定义及 WebUI Schema
│   ├── types.ts              # TypeScript 类型定义
│   ├── core/
│   │   └── state.ts          # 全局状态管理
│   ├── handlers/             # 业务逻辑处理模块
│   │   └── message-handler.ts # 示例：消息处理器
│   ├── services/             # 服务层，API 路由等
│   └── webui/                # (可选) 插件的 WebUI 前端代码
├── package.json              # 项目依赖及元数据
└── tsconfig.json             # TypeScript 配置
```

## 编写核心逻辑

在 `src/index.ts` 中，你需要导出插件的生命周期函数。这是插件被 NapCat 加载时执行的入口点。请务必遵循 NapCat 的生命周期。

### 插件入口 (`src/index.ts`)

插件需要导出 `PluginModule` 接口中定义的生命周期函数。其中 `plugin_init` 是唯一必选的函数。



```
import type { PluginModule, OB11Message, OB11PostSendMsg } from "napcat-types";
import { EventType } from "napcat-types";

// 1. 插件初始化（必选）
// 插件加载时调用，用于初始化资源、注册路由等
export const plugin_init: PluginModule['plugin_init'] = async (ctx) => {
    ctx.logger.log('My Plugin has been initialized!');
    
    // 初始化时可以做一些准备工作
};

// 2. 消息处理（可选）
// 收到事件时调用，需要通过 post_type 判断是否为消息事件
export const plugin_onmessage: PluginModule['plugin_onmessage'] = async (ctx, event) => {
    // 过滤非消息事件
    if (event.post_type !== EventType.MESSAGE) return;

    // 简单的 Ping-Pong 示例
    if (event.raw_message === 'ping') {
        ctx.logger.log(`收到来自 ${event.user_id} 的 ping`);

        // 构造发送消息的参数
        const params: OB11PostSendMsg = {
            message: 'pong',
            message_type: event.message_type,
            ...(event.message_type === 'group' && event.group_id
                ? { group_id: String(event.group_id) }
                : {}),
            ...(event.message_type === 'private' && event.user_id
                ? { user_id: String(event.user_id) }
                : {}),
        };
        // 通过 actions 调用 OneBot11 Action (需传入 adapter 和 config)
        await ctx.actions.call('send_msg', params, ctx.adapterName, ctx.pluginManager.config);
    }
};

// 3. 事件处理（可选）
// 处理所有 OneBot 事件（包括通知、请求等）
export const plugin_onevent: PluginModule['plugin_onevent'] = async (ctx, event) => {
    ctx.logger.log('收到事件:', event);
};

// 4. 插件卸载（可选）
// 插件重载或卸载时调用，必须清理定时器、关闭连接等
export const plugin_cleanup: PluginModule['plugin_cleanup'] = (ctx) => {
    ctx.logger.log('My Plugin is cleaning up...');
    // 清理资源
};
```

关于 actions.call 调用

`ctx.actions.call(actionName, params, adapter, config)` 需要 4 个参数：

- `actionName`: OneBot11 Action 名称（如 `'send_msg'`、`'get_login_info'` 等）
- `params`: 请求参数
- `adapter`: 适配器名称，使用 `ctx.adapterName`
- `config`: 网络配置，使用 `ctx.pluginManager.config`

### 生命周期函数一览

| 函数名                     | 是否必选 | 说明                                              |
| :------------------------- | :------- | :------------------------------------------------ |
| `plugin_init`              | 必选     | 插件加载时调用，用于初始化                        |
| `plugin_onmessage`         | 可选     | 收到事件时调用（需通过 `post_type` 判断事件类型） |
| `plugin_onevent`           | 可选     | 收到所有 OneBot 事件时调用                        |
| `plugin_cleanup`           | 可选     | 插件卸载/重载时调用                               |
| `plugin_config_schema`     | 可选     | 导出配置 Schema（同 `plugin_config_ui`）          |
| `plugin_config_ui`         | 可选     | 导出配置 Schema，用于在 WebUI 生成配置面板        |
| `plugin_get_config`        | 可选     | 获取插件配置的自定义实现                          |
| `plugin_set_config`        | 可选     | 保存插件配置的自定义实现                          |
| `plugin_config_controller` | 可选     | 配置 UI 控制器，用于动态控制配置界面              |
| `plugin_on_config_change`  | 可选     | 配置变更时触发的回调                              |

### 消息处理的最佳实践

为了保持代码整洁，建议将复杂的业务逻辑分离到 `src/handlers` 目录下的独立文件中，而不是全部写在 `plugin_onmessage` 里。

例如创建 `src/handlers/message.ts`：



```
import type { NapCatPluginContext, OB11Message, OB11PostSendMsg } from "napcat-types";

export async function handleGroupMessage(ctx: NapCatPluginContext, message: OB11Message) {
  const { raw_message, group_id } = message;

  // 封装发送群消息的辅助函数
  async function sendGroupMsg(text: string) {
    const params: OB11PostSendMsg = {
      message: text,
      message_type: 'group',
      group_id: String(group_id),
    };
    await ctx.actions.call('send_msg', params, ctx.adapterName, ctx.pluginManager.config);
  }

  // 示例：简单的关键词回复
  if (raw_message.includes("喵")) {
    await sendGroupMsg("喵喵喵？");
  }

  // 示例：调用 OneBot 11 Action
  if (raw_message === "#info") {
       const loginInfo = await ctx.actions.call(
       'get_login_info', void 0, ctx.adapterName, ctx.pluginManager.config
     );
     await sendGroupMsg(`当前登录账号: ${loginInfo.nickname} (${loginInfo.user_id})`);
  }
}
```

然后在 `index.ts` 中引入使用：



```
import type { PluginModule } from "napcat-types";
import { EventType } from "napcat-types";
import { handleGroupMessage } from "./handlers/message";

export const plugin_onmessage: PluginModule['plugin_onmessage'] = async (ctx, event) => {
    if (event.post_type !== EventType.MESSAGE) return;
    if (event.message_type === "group") {
        await handleGroupMessage(ctx, event);
    }
};
```

### NapCatPluginContext 核心属性

`NapCatPluginContext` 是插件开发中最重要的上下文对象，包含以下属性：

- **`ctx.core`**: `NapCatCore` 实例，底层核心。
- **`ctx.oneBot`**: `NapCatOneBot11Adapter` 实例，OneBot11 适配器。
- **`ctx.actions`**: `ActionMap` 对象，通过 `ctx.actions.call(actionName, params, ctx.adapterName, ctx.pluginManager.config)` 调用 OneBot11 Action。
- **`ctx.pluginName`**: 当前插件名称。
- **`ctx.pluginPath`**: 插件所在目录路径。
- **`ctx.configPath`**: 插件配置文件路径。
- **`ctx.dataPath`**: 插件数据存储目录路径。
- **`ctx.NapCatConfig`**: 配置构建工具类，提供 `text`/`number`/`boolean`/`select` 等静态方法。
- **`ctx.adapterName`**: 适配器名称。
- **`ctx.pluginManager`**: 插件管理器，支持查询/启停其他插件。
- **`ctx.logger`**: 使用 NapCat 内置的日志系统，保持日志格式统一。
- **`ctx.router`**: 路由注册器，用于注册 API 路由、页面和静态文件。
- **`ctx.getPluginExports(pluginId)`**: 获取其他已加载插件的导出对象。

## 配置文件

NapCat 插件支持通过导出 `plugin_config_ui` 或 `plugin_config_schema` 来定义配置项。这允许用户在 WebUI 中修改插件的设置。



```
import { PluginConfigSchema } from 'napcat-types';

export const plugin_config_ui: PluginConfigSchema = [
  {
    key: 'enable',
    label: '启用插件',
    type: 'boolean',
    default: true,
    description: '是否启用本插件的核心功能'
  },
  {
    key: 'prefix',
    label: '命令前缀',
    type: 'string',
    default: '/',
    placeholder: '请输入命令前缀，例如 / 或 #',
    description: '触发命令的前缀符号'
  }
];
```

## WebUI 开发 (可选)

如果插件需要提供图形化界面，可以在 `src/webui` 目录下进行 React 开发。模板中已经配置好了 Vite 构建脚本，运行 `npm run build` 时，构建后的静态资源会被自动打包进插件中，用户可以通过 NapCat 的管理界面访问。

## 发布插件

### 发布插件流程

1. 配置 `package.json`

   : 确保你的

    

   ```
   package.json
   ```

    

   包含正确的插件元数据：

   

   ```
   {
     "name": "napcat-plugin-your-plugin",
     "plugin": "我的插件",
     "version": "1.0.0",
     "main": "index.mjs",
     "description": "这是我的 NapCat 插件",
     "author": "YourName",
     "napcat": {
       "tags": ["工具", "娱乐"], 
       "minVersion": "4.14.0",
       "homepage": "https://github.com/yourname/napcat-plugin-your-plugin" 
     }
   }
   ```

2. **发布版本**: 推送 `v*` 格式的 Git Tag (例如 `v1.0.0`) 到你的远程仓库。

3. **CI/CD 自动发布**: 如果你基于模板开发并配置了 Secret，GitHub Actions 会自动构建并发布 Release。

4. **提交索引**: 同时 CI 会自动向 [napcat-plugin-index](https://github.com/NapNeko/napcat-plugin-index) 提交 PR。等待维护者审核合并后，你的插件就会出现在 NapCat 的插件市场中供用户下载。

## 自动化发布 (CI/CD)

NapCat 插件模板内置了 GitHub Actions 工作流，可自动完成构建、发布 Release 以及提交到插件市场的全过程。

### 工作流介绍

模板仓库包含两个主要的工作流文件：

1. **`release.yml`**: 负责插件的构建和 Release 发布。

   - **触发条件**: 当你推送 `v*` 格式的 tag (如 `v1.0.0`) 时自动触发。

   - 主要任务

     :

     - 安装依赖 (`pnpm install`)
     - 构建插件 (`npm run build`)
     - 打包构建产物为 `.zip` 文件
     - 创建 GitHub Release 并上传 `.zip` 包

2. **`update-index.yml`**: 负责将发布后的插件自动提交到 [NapCat 插件索引仓库](https://github.com/NapNeko/napcat-plugin-index)。

   - **触发条件**: 当 `release.yml` 成功发布 Release 后触发。

   - 主要任务

     :

     - 抓取 `package.json` 中的插件元数据 (版本、描述、作者等)
     - 向 NapCat 插件索引仓库提交 Pull Request (PR)
     - 等待仓库维护者审核合并

### 配置步骤

为了让自动提交索引的功能正常工作，你需要进行少量的配置：

#### 1. 设置 Repository Secret

由于 GitHub Actions 默认权限无法跨仓库提交 PR，你需要提供一个具有权限的 Token。

1. **生成 Token**:
   - 访问 [GitHub Developer Settings](https://github.com/settings/tokens)
   - 生成一个新的 **Personal Access Token (Classic)**
   - 勾选 `public_repo` (对于公开仓库) 权限
   - 复制生成的 Token
2. **配置 Secret**:
   - 进入你的插件仓库 -> `Settings` -> `Secrets and variables` -> `Actions`
   - 点击 `New repository secret`
   - Name: `INDEX_PAT` (必须完全一致)
   - Value: 粘贴刚才复制的 Token
   - 点击 `Add secret` 保存

#### 2. 完善 `package.json`

自动化脚本会从 `package.json` 读取插件信息，请务必完整填写：



```
{
  "name": "napcat-plugin-demo",       // 插件ID (必须以 napcat-plugin- 开头)
  "plugin": "插件显示名称",           // 插件在 WebUI 中的显示名称
  "version": "1.0.0",                 // 版本号
  "main": "index.mjs",                // 入口文件
  "description": "这是插件描述",        // 插件描述
  "author": "YourName",               // 作者
  "napcat": {
    "tags": ["工具", "娱乐"],          // 插件标签
    "minVersion": "4.14.0",           // 最低支持的 NapCat 版本
    "homepage": "https://..."         // 项目主页
  }
}
```

### 完整的发布操作

配置完成后，发布新版本的流程如下：

1. **修改版本号**: 更新 `package.json` 中的 `version` 字段 (例如 `1.0.1`)。

2. **提交代码**: 将修改推送到 GitHub。

3. 打标签

   :

   

   ```
   git tag v1.0.1
   git push origin v1.0.1
   ```

4. 等待执行

   :

   - GitHub Actions 会自动运行 `release.yml` 构建并发布 Release。
   - 随后触发 `update-index.yml`，自动向官方索引仓库提交 PR。

5. **完成**: 等待 PR 被合并后，插件将收录至 NapCat 插件市场。

## 另请参阅

- [热重载开发](https://napneko.github.io/develop/plugin/hot-reload) — 使用调试服务实现改代码即时生效，无需重启 NapCat
- [NapCat 插件模板](https://github.com/NapNeko/napcat-plugin-template)
- [NapCat 插件索引](https://github.com/NapNeko/napcat-plugin-index)

# 项目结构与最佳实践

字数

980 字

阅读时间

4 分钟

NapCat 插件模板（`napcat-plugin-template`）提供了一套经过生产验证的项目结构。理解该结构有助于编写更易维护和扩展的插件。

## 目录结构详解

以下是一个标准的插件项目的目录树：



```
src/
├── index.ts              # [入口] 插件的主入口文件
├── config.ts             # [配置] 定义插件配置结构及默认值
├── types.ts              # [类型] 定义插件内部使用的 TypeScript 类型
├── core/
│   └── state.ts          # [状态] 全局状态管理单例
├── handlers/             # [逻辑] 具体的业务逻辑处理
│   └── message-handler.ts
├── services/             # [服务] 外部服务接口或复杂功能封装
│   └── api-service.ts
└── webui/                # [前端] 插件的 WebUI 界面代码
```

### 1. `index.ts` - 插件入口

这是插件被 NapCat 加载的地方。它的主要职责是：

- 导出插件的生命周期函数（实现 `PluginModule` 接口）。
- 在 `plugin_init` 中初始化全局状态、注册路由等。
- 在 `plugin_onmessage` 中处理消息。
- 在 `plugin_cleanup` 中清理资源。

**最佳实践**：不要在 `index.ts` 中编写具体的业务逻辑。应该将逻辑委托给 `handlers/` 中的函数。

### 2. `core/state.ts` - 全局状态管理

插件通常需要维护一些全局状态，例如：

- NapCat 的上下文对象 (`NapCatPluginContext`)
- 当前的配置信息 (`config`)
- 插件的元数据（名称、路径等）

模板中使用了一个单例模式的 `PluginState` 类来管理这些信息。这种方式允许在项目的任意位置（包括深层嵌套的函数中）直接访问 `ctx` 或 `config`，无需逐层传递参数。



```
// 使用示例
import { pluginState } from "../core/state";

export function doSomething() {
    // 直接获取 logger
    pluginState.logger.info("Doing something...");
    
    // 直接获取配置
    if (pluginState.config.enableFeatureA) {
        // ...
    }
}
```

### 3. `handlers/` - 业务逻辑

这里存放具体的事件处理函数。

- `message-handler.ts`: 处理群消息、私聊消息。
- `request-handler.ts`: 处理加群请求、好友请求。
- `notice-handler.ts`: 处理群通知（如成员变动）。

建议将不同类型的逻辑拆分到不同的文件中。

### 4. `services/` - 服务层

如果你的插件需要调用外部 API（如查询天气、获取游戏数据），或者包含复杂的独立功能模块（如定时任务管理器），建议放在 `services/` 目录下。

### 5. `webui/` - 前端界面

NapCat 插件支持通过 WebUI 提供图形化配置或展示界面。模板中预置了基于 React + Vite 的前端项目结构。

- 构建脚本会自动将 `webui/` 下的代码打包，并嵌入到插件发布包中。
- 在 `plugin_init` 中通过 `ctx.router.static` 将 WebUI 静态资源暴露给用户访问。

## 常用开发模式

### 依赖注入 vs 全局单例

虽然依赖注入（Dependency Injection）在大型应用中很常见，但在插件开发中，使用 **全局单例** (`pluginState`) 通常更简洁高效。它避免了在每个函数调用中透传 `ctx` 对象。

### 错误处理

在 `handlers` 中捕获错误非常重要，防止一个未捕获的异常导致整个插件甚至 Bot 崩溃。



```
try {
    // 你的逻辑
} catch (e) {
    pluginState.logger.error("处理消息时发生错误", e);
}
```

### 配置热重载

NapCat 支持配置的热重载。当用户在 WebUI 中修改配置后，插件可以通过 `plugin_on_config_change` 钩子感知变化：



```
import type { PluginModule } from "napcat-types";

export const plugin_on_config_change: PluginModule['plugin_on_config_change'] = async (_ctx, ui, key, value, _currentConfig) => {
    _ctx.logger.info(`配置项 ${key} 已变更为:`, value);
    // 根据变更更新内存中的状态
};
```

或者通过 `plugin_config_controller` 获得更完整的 UI 控制能力。

# 插件配置与 WebUI 开发

字数

1679 字

阅读时间

8 分钟

NapCat 插件不仅支持配置文件，还支持提供图形化界面（WebUI）供用户交互。本指南将介绍如何定义配置以及开发 WebUI 界面。

## 1. 配置文件定义

在 `src/index.ts` 中，你需要导出配置的 **WebUI Schema** 和可选的配置读写钩子。

### 定义配置接口

首先定义 TypeScript 接口，以便在代码中获得类型提示。



```
// src/types.ts
export interface PluginConfig {
  enable: boolean;
  prefix: string;
  adminList: number[];
}
```

### 定义 WebUI Schema

NapCat 使用 `PluginConfigSchema` 来自动生成配置面板。这是最推荐的配置方式，无需手动编写前端代码即可拥有一个漂亮的配置界面。

通过导出 `plugin_config_ui` 或 `plugin_config_schema` 来定义：



```
// src/index.ts
import { PluginConfigSchema } from 'napcat-types';

export const plugin_config_ui: PluginConfigSchema = [
  {
    key: 'enable',
    label: '启用插件',
    type: 'boolean',
    default: true,
    description: '是否启用本插件的核心功能'
  },
  {
    key: 'prefix',
    label: '命令前缀',
    type: 'string',
    default: '/',
    placeholder: '请输入命令前缀，例如 / 或 #',
    description: '触发命令的前缀符号'
  },
  {
    key: 'adminList',
    label: '管理员列表',
    type: 'multi-select',
    default: [],
    options: [
        { label: '用户A', value: 123456 },
        { label: '用户B', value: 654321 }
    ],
    description: '拥有管理员权限的用户 QQ 号'
  }
];
```

### 支持的控件类型

`PluginConfigItem` 的 `type` 字段支持以下值：

| 类型           | 说明                         |
| :------------- | :--------------------------- |
| `string`       | 文本输入框                   |
| `number`       | 数字输入框                   |
| `boolean`      | 开关                         |
| `select`       | 下拉单选                     |
| `multi-select` | 下拉多选                     |
| `text`         | 多行文本框                   |
| `html`         | 自定义 HTML 展示（不保存值） |

### PluginConfigItem 完整属性



```
interface PluginConfigItem {
    key: string;                    // 配置项唯一标识
    type: 'string' | 'number' | 'boolean' | 'select' | 'multi-select' | 'html' | 'text';
    label: string;                  // 显示标签
    description?: string;           // 描述说明
    default?: unknown;              // 默认值
    options?: { label: string; value: string | number }[]; // select/multi-select 的选项
    placeholder?: string;           // 输入框占位符
    reactive?: boolean;             // 标记为响应式：值变化时触发 schema 刷新
    hidden?: boolean;               // 是否隐藏此字段
}
```

### 使用 NapCatConfig 构建器

除了手动编写 Schema 数组，你还可以使用 `ctx.NapCatConfig` 提供的静态方法快速构建：



```
import { NapCatPluginContext, PluginConfigSchema } from 'napcat-types';

export const plugin_init = (ctx: NapCatPluginContext) => {
    // NapCatConfig 提供了便捷的构建方法
    const schema: PluginConfigSchema = ctx.NapCatConfig.combine(
        ctx.NapCatConfig.text('apiKey', 'API Key', '', 'OpenAI API 密钥'),
        ctx.NapCatConfig.number('maxRetries', '最大重试次数', 3),
        ctx.NapCatConfig.boolean('debug', '调试模式', false, '开启后输出详细日志'),
        ctx.NapCatConfig.select('model', '模型选择', [
            { label: 'GPT-3.5', value: 'gpt-3.5-turbo' },
            { label: 'GPT-4', value: 'gpt-4' }
        ], 'gpt-3.5-turbo'),
        ctx.NapCatConfig.multiSelect('features', '启用功能', [
            { label: '翻译', value: 'translate' },
            { label: '摘要', value: 'summary' }
        ], ['translate']),
        ctx.NapCatConfig.html('<p style="color:gray">这是一段说明文字</p>'),
        ctx.NapCatConfig.plainText('纯文本说明内容')
    );
};
```

### 自定义配置读写

默认情况下 NapCat 会自动管理配置读写。如果需要自定义行为，可以导出以下钩子：



```
import type { PluginModule } from 'napcat-types';
import { PluginConfig } from './types';

// 自定义配置读取
export const plugin_get_config: PluginModule['plugin_get_config'] = async (ctx) => {
    // 从自定义位置读取配置
    return loadMyConfig(ctx.configPath);
};

// 自定义配置保存
export const plugin_set_config: PluginModule['plugin_set_config'] = async (ctx, config) => {
    saveMyConfig(ctx.configPath, config);
};
```

### 配置 UI 动态控制

通过 `plugin_config_controller` 可以在运行时动态修改配置界面：



```
import type { PluginModule, PluginConfigUIController } from 'napcat-types';

export const plugin_config_controller: PluginModule['plugin_config_controller'] = async (_ctx, ui, initialConfig) => {
    // 根据初始配置动态显示/隐藏字段
    if (initialConfig.mode === 'simple') {
        ui.hideField('advancedOption');
    }

    // 返回清理函数（可选）
    return () => {
        // 清理资源
    };
};

// 配置变更时的回调
export const plugin_on_config_change: PluginModule['plugin_on_config_change'] = async (_ctx, ui, key, value, _currentConfig) => {
    // 当 mode 字段变化时，动态调整 UI
    if (key === 'mode') {
        if (value === 'advanced') {
            ui.showField('advancedOption');
        } else {
            ui.hideField('advancedOption');
        }
    }
};
```

`PluginConfigUIController` 提供以下方法：

| 方法                         | 说明                     |
| :--------------------------- | :----------------------- |
| `updateSchema(schema)`       | 更新整个配置 Schema      |
| `updateField(key, field)`    | 更新指定字段的部分属性   |
| `removeField(key)`           | 移除指定字段             |
| `addField(field, afterKey?)` | 添加新字段（可指定位置） |
| `showField(key)`             | 显示指定字段             |
| `hideField(key)`             | 隐藏指定字段             |
| `getCurrentConfig()`         | 获取当前配置值           |

## 2. 在 WebUI 中使用自定义前端

如果你需要更复杂的交互界面（如数据可视化、文件管理等），可以使用 React/Vue 开发完整的 SPA 页面。`napcat-plugin-template` 的 `webui` 目录正是为此准备的。

### 开启 WebUI 路由

在 `src/index.ts` 中注册 API 和静态资源：



```
// src/index.ts
import type { PluginModule } from "napcat-types";
import { registerApiRoutes } from './services/api-service';

export const plugin_init: PluginModule['plugin_init'] = async (ctx) => {
    // 1. 注册后端 API (供前端调用)
    registerApiRoutes(ctx);

    // 2. 托管静态文件 (前端 build 产物)
    // 假设你的前端 build output 在插件目录下的 webui/dist
    // 访问路径: http://host:port/plugin/<plugin-id>/files/<urlPath>/
    ctx.router.static('/static', 'webui/dist');
    
    // 3. 注册侧边栏页面入口 (可选)
    // 访问路径: http://host:port/plugin/<plugin-id>/page/dashboard
    ctx.router.page({
        title: '我的插件',
        path: 'dashboard',                // 路由路径
        htmlFile: 'webui/dist/index.html', // 入口 HTML 文件
        icon: '🔌',                       // 页面图标
        description: '插件管理面板'        // 页面描述
    });

    // 4. 也可以使用内存静态文件（适合动态生成内容）
    // 访问路径: http://host:port/plugin/<plugin-id>/mem/dynamic/config.json
    ctx.router.staticOnMem('/dynamic', [
        {
            path: '/config.json',
            content: () => JSON.stringify({ version: '1.0.0' }),
            contentType: 'application/json'
        }
    ]);
};
```

### 开发 WebUI 前端

模板中的 `src/webui` 是一个标准的 React + Vite 项目。

1. **启动开发服务**: 进入 `src/webui` 运行 `npm install && npm run dev`。
2. **调用后端接口**: 使用 `fetch` 或 `axios` 调用你在插件中注册的 API。

#### 路由前缀说明

| 路由类型   | 前缀                                   | 注册方法                                                     |
| :--------- | :------------------------------------- | :----------------------------------------------------------- |
| 鉴权 API   | `/api/Plugin/ext/<plugin-id>/`         | `ctx.router.get/post/put/delete/api`                         |
| 无鉴权 API | `/plugin/<plugin-id>/api/`             | `ctx.router.getNoAuth/postNoAuth/putNoAuth/deleteNoAuth/apiNoAuth` |
| 静态文件   | `/plugin/<plugin-id>/files/<urlPath>/` | `ctx.router.static(urlPath, localPath)`                      |
| 内存文件   | `/plugin/<plugin-id>/mem/<urlPath>/`   | `ctx.router.staticOnMem(urlPath, files)`                     |
| 扩展页面   | `/plugin/<plugin-id>/page/<path>`      | `ctx.router.page(definition)`                                |

建议在前端封装请求工具，自动处理前缀。

### 前后端通信示例

**后端 (node)**:



```
// src/services/api-service.ts
import type { NapCatPluginContext } from "napcat-types";

export function registerApiRoutes(ctx: NapCatPluginContext) {
    // 注册需要鉴权的 GET 接口
    // 访问: /api/Plugin/ext/<plugin-id>/status
    ctx.router.get('/status', (_req, res) => {
        res.json({ status: 'ok', time: Date.now() });
    });

    // 注册无需鉴权的 POST 接口
    // 访问: /plugin/<plugin-id>/api/webhook
    ctx.router.postNoAuth('/webhook', async (req, res) => {
        const data = req.body;
        ctx.logger.info('收到 webhook:', data);
        res.status(200).json({ received: true });
    });
}
```

**前端 (react)**:



```
// src/webui/src/App.tsx
useEffect(() => {
    // 假设封装了 fetcher 自动添加前缀
    fetcher.get('/status').then(data => {
        console.log('插件状态:', data);
    });
}, []);
```

## 3. 构建与发布

当你运行插件根目录的 `npm run build` 时，构建脚本通常会：

1. 构建插件 TypeScript 代码 (`src/index.ts` 等)。
2. 构建 WebUI 前端代码 (`src/webui`)。
3. 将前端产物 (`dist`) 复制到插件输出目录 (`dist/webui/dist`)。
4. 打包为 `napcat-plugin-xxx.zip` 或直接发布文件夹。

确保 `package.json` 中的 `files` 字段包含了所有必要的文件。

# 进阶功能开发

字数

1624 字

阅读时间

8 分钟

本章节将介绍 NapCat 插件的高级功能，包括高级 API 路由使用、数据持久化存储以及与其他插件的交互。

## 1. 高级路由与网络服务

`NapCatPluginContext` 提供了强大的路由注册能力，支持 GET/POST/PUT/DELETE 等多种 HTTP 方法，并自动处理插件路径前缀。

### 路由注册 API 概览

所有的路由注册方法都在 `ctx.router` 对象下，分为三类：

#### 需要鉴权的路由

路径前缀：`/api/Plugin/ext/<plugin-id>/`

| 方法     | 签名                      | 说明                                                         |
| :------- | :------------------------ | :----------------------------------------------------------- |
| `get`    | `(path, handler)`         | 注册 GET 接口                                                |
| `post`   | `(path, handler)`         | 注册 POST 接口                                               |
| `put`    | `(path, handler)`         | 注册 PUT 接口                                                |
| `delete` | `(path, handler)`         | 注册 DELETE 接口                                             |
| `api`    | `(method, path, handler)` | 通用方法，`method` 支持 `'get' | 'post' | 'put' | 'delete' | 'patch' | 'all'` |

#### 无需鉴权的路由

路径前缀：`/plugin/<plugin-id>/api/`

| 方法           | 签名                      | 说明                 |
| :------------- | :------------------------ | :------------------- |
| `getNoAuth`    | `(path, handler)`         | 注册 GET 开放接口    |
| `postNoAuth`   | `(path, handler)`         | 注册 POST 开放接口   |
| `putNoAuth`    | `(path, handler)`         | 注册 PUT 开放接口    |
| `deleteNoAuth` | `(path, handler)`         | 注册 DELETE 开放接口 |
| `apiNoAuth`    | `(method, path, handler)` | 通用方法             |

#### 静态文件与页面

| 方法          | 签名                   | 路径前缀                               | 说明                 |
| :------------ | :--------------------- | :------------------------------------- | :------------------- |
| `static`      | `(urlPath, localPath)` | `/plugin/<plugin-id>/files/<urlPath>/` | 注册本地静态文件服务 |
| `staticOnMem` | `(urlPath, files)`     | `/plugin/<plugin-id>/mem/<urlPath>/`   | 注册内存静态文件服务 |
| `page`        | `(pageDefinition)`     | `/plugin/<plugin-id>/page/<path>`      | 注册单个页面         |
| `pages`       | `(pageDefinitions)`    | 同上                                   | 批量注册页面         |

### 请求/响应对象

路由处理器接收包装后的请求和响应对象：



```
// 请求对象 PluginHttpRequest
interface PluginHttpRequest {
    path: string;                                        // 请求路径
    method: string;                                      // 请求方法
    query: Record<string, string | string[] | undefined>; // 查询参数
    body: unknown;                                       // 请求体
    headers: Record<string, string | string[] | undefined>; // 请求头
    params: Record<string, string>;                      // 路由参数
    raw: unknown;                                        // 原始请求对象
}

// 响应对象 PluginHttpResponse
interface PluginHttpResponse {
    status(code: number): PluginHttpResponse; // 设置状态码
    json(data: unknown): void;                // 发送 JSON 响应
    send(data: string | Buffer): void;        // 发送文本响应
    setHeader(name: string, value: string): PluginHttpResponse; // 设置响应头
    sendFile(filePath: string): void;         // 发送文件
    redirect(url: string): void;              // 重定向
    raw: unknown;                             // 原始响应对象
}
```

### 示例：实现一个 Webhook 接收器

假设你需要接收 GitHub Webhook 推送通知到 QQ 群：



```
// src/services/webhook.ts
import type { NapCatPluginContext, OB11PostSendMsg } from "napcat-types";

export function registerWebhook(ctx: NapCatPluginContext) {
    // 注册无需鉴权的 POST 接口
    // 外部访问地址: http://napcat-ip:port/plugin/<your-plugin-id>/api/webhook/github
    ctx.router.postNoAuth('/webhook/github', async (req, res) => {
        const event = req.headers['x-github-event'];
        const payload = req.body as any;
        
        ctx.logger.info(`收到 GitHub 事件: ${event}`);
        
        if (event === 'push') {
            const repo = payload.repository.full_name;
            const pusher = payload.pusher.name;
            const msg = `项目 ${repo} 收到来自 ${pusher} 的新提交`;
            
            // 通过 actions 发送消息到群 (建议从配置读取群号)
            const params: OB11PostSendMsg = {
                message: msg,
                message_type: 'group',
                group_id: '123456',
            };
            await ctx.actions.call('send_msg', params, ctx.adapterName, ctx.pluginManager.config);
        }
        
        res.json({ status: 'ok' });
    });
}
```

## 2. 数据持久化

插件通常需要保存运行时数据（如签到记录、积分等），NapCat 提供了数据目录供插件使用。

### 获取数据目录

使用 `ctx.dataPath` 获取分配给当前插件的专属数据目录。也可以通过 `ctx.pluginManager.getPluginDataPath(pluginId)` 获取。



```
import fs from 'fs';
import path from 'path';
import { NapCatPluginContext } from "napcat-types";

// 保存数据
function saveData(ctx: NapCatPluginContext, data: any) {
    const filePath = path.join(ctx.dataPath, 'data.json');
    // 确保目录存在（NapCat 通常会自动创建 dataPath，但为了保险）
    if (!fs.existsSync(ctx.dataPath)) {
        fs.mkdirSync(ctx.dataPath, { recursive: true });
    }
    fs.writeFileSync(filePath, JSON.stringify(data, null, 2));
}

// 读取数据
function loadData(ctx: NapCatPluginContext) {
    const filePath = path.join(ctx.dataPath, 'data.json');
    if (fs.existsSync(filePath)) {
        return JSON.parse(fs.readFileSync(filePath, 'utf-8'));
    }
    return {};
}
```

### 推荐：使用轻量级数据库

对于复杂数据，建议使用 `sqlite3` 或 `leveldb` (如 `keyv`)。

## 3. 插件间通信

有时候一个插件需要调用另一个插件的功能（例如积分插件调用经济插件的接口）。

### 使用 `ctx.getPluginExports`

NapCat 提供了 `ctx.getPluginExports<T>(pluginId)` 方法来获取其他已加载插件的导出对象（即该插件的 `PluginModule`）。

**插件 A (Provider)**:



```
// src/index.ts
import { NapCatPluginContext } from "napcat-types";

export const plugin_init = (ctx: NapCatPluginContext) => {
    ctx.logger.log('Plugin A initialized');
};

// 导出公共方法供其他插件调用
export const somePublicMethod = () => {
    return "Hello from Plugin A";
}
```

**插件 B (Consumer)**:



```
// src/index.ts
import { NapCatPluginContext, PluginModule } from "napcat-types";

// 定义 Plugin A 的导出类型（可选，用于类型安全）
interface PluginAExports extends PluginModule {
    somePublicMethod: () => string;
}

export const plugin_init = (ctx: NapCatPluginContext) => {
    // 获取插件 A 的导出，泛型参数用于类型提示
    const pluginA = ctx.getPluginExports<PluginAExports>('plugin-a-id');
    if (pluginA && pluginA.somePublicMethod) {
        console.log(pluginA.somePublicMethod()); // "Hello from Plugin A"
    }
}
```

## 4. 调试与日志

使用 `ctx.logger` 输出日志，日志会自动带上插件名称前缀，方便排查问题。

`ctx.logger` 实现了 `PluginLogger` 接口，提供以下方法：



```
ctx.logger.log("普通日志");     // 通用日志输出
ctx.logger.debug("调试信息");   // 仅在 debug 模式显示
ctx.logger.info("普通信息");    // 信息级别
ctx.logger.warn("警告信息");    // 警告级别
ctx.logger.error("错误信息", new Error("oops")); // 错误级别
```

在开发过程中，建议开启 `debug: true` 配置以便通过控制台查看详细的调试日志。

提示

NapCat 插件模板中，配置更新通常通过 WebUI 触发。插件可以通过 `plugin_on_config_change` 钩子实时感知配置变更，实现无需重启的热重载。

## 5. 定时任务 (Cron Job)

如果你的插件需要定时执行任务（如每天早上发送早报），可以使用 `node-schedule` 或 `cron` 库。

**注意**: 在插件卸载 (`plugin_cleanup`) 时，**务必取消**所有定时任务，否则会导致内存泄漏或重复执行。



```
// src/services/cron-service.ts
import cron from "node-cron";

let task: cron.ScheduledTask;

export function startDailyReport() {
    // 每天 8:00 执行
    task = cron.schedule("0 8 * * *", () => {
        console.log("发送早报...");
        // sendReport();
    });
}

export function stopDailyReport() {
    if (task) task.stop();
}
```

在 `src/index.ts` 中：



```
import { NapCatPluginContext } from "napcat-types";
import { startDailyReport, stopDailyReport } from "./services/cron-service";

export const plugin_init = (ctx: NapCatPluginContext) => {
    startDailyReport();
};

export const plugin_cleanup = (ctx: NapCatPluginContext) => {
    stopDailyReport();
};
```

## 6. 插件管理器 API

通过 `ctx.pluginManager` 可以管理其他插件。`IPluginManager` 接口提供以下方法：

| 方法                                    | 说明                     |
| :-------------------------------------- | :----------------------- |
| `getPluginPath()`                       | 获取插件根目录           |
| `getPluginConfig()`                     | 获取插件启用状态配置     |
| `getAllPlugins()`                       | 获取所有已扫描的插件列表 |
| `getLoadedPlugins()`                    | 获取所有已加载的插件列表 |
| `getPluginInfo(pluginId)`               | 获取指定插件信息         |
| `setPluginStatus(pluginId, enable)`     | 启用/禁用插件            |
| `loadPluginById(pluginId)`              | 加载指定插件             |
| `unregisterPlugin(pluginId)`            | 注销插件                 |
| `uninstallPlugin(pluginId, cleanData?)` | 卸载并删除插件           |
| `reloadPlugin(pluginId)`                | 重新加载插件             |
| `loadDirectoryPlugin(dirname)`          | 从目录加载插件           |
| `getPluginDataPath(pluginId)`           | 获取插件数据目录         |
| `getPluginConfigPath(pluginId)`         | 获取插件配置目录         |

# 热重载开发

字数

2677 字

阅读时间

12 分钟

在传统的插件开发流程中，每次修改代码都需要手动构建、复制文件、重启 NapCat，效率较低。NapCat 提供了 **热重载（HMR）** 机制，允许你在开发过程中修改代码后自动重新加载插件，无需重启 NapCat。

适用场景

热重载适用于 **本地开发和调试阶段**。当你完成开发并准备发布时，仍然使用 [发布插件](https://napneko.github.io/develop/plugin/publish) 中的标准流程。

安全警告

调试服务基于 WebSocket 通信，**默认不启用认证（无 token）**，任何能访问该端口的客户端都可以执行插件管理操作（加载、卸载、重载插件等）。

- **强烈建议在本地环境调试**（默认监听 `127.0.0.1`），避免远程调试
- **未配置 token 时，切勿将调试端口暴露在公网中**，否则任何人都可以连接并操控你的 NapCat 插件
- 如果确实需要远程调试，**必须同时启用认证并设置高强度 token**，且通过防火墙严格限制来源 IP

## 架构概览

热重载机制由三个组件协同工作：

本地开发环境NapCat 端WebSocket
JSON-RPCnapcat-plugin-debug
(NapCat 端插件)napcatHmrPlugin
(Vite 插件)PluginManager APIVite build --watch自动部署自动重载

| 组件                                                         | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **[napcat-plugin-debug](https://github.com/NapNeko/napcat-plugin-debug)** | NapCat 端插件，启动 WebSocket 服务器（默认 `ws://127.0.0.1:8998`），将 PluginManager 的 API 暴露为 JSON-RPC 接口 |
| **napcatHmrPlugin** (Vite 插件)                              | 集成到 Vite 构建流程中，在每次 `writeBundle` 时自动连接调试服务、复制 dist/ 到远程、调用 reloadPlugin。来自 `napcat-plugin-debug-cli/vite` |
| **[napcat-plugin-debug-cli](https://www.npmjs.com/package/napcat-plugin-debug-cli)** | 可选的 CLI 工具，提供交互式 REPL、手动部署等功能             |

## 前置配置

### 1. NapCat 端：安装调试插件

在 NapCat 的插件市场中搜索并安装 `napcat-plugin-debug`（插件调试服务），启用后它会自动在 `ws://127.0.0.1:8998` 启动 WebSocket 调试服务。

<details class="details custom-block" style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border: 1px solid rgba(0, 0, 0, 0); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-radius: 8px; padding: 16px 16px 8px; line-height: 24px; font-size: 14px; color: rgb(223, 223, 214); background-color: rgba(255, 255, 255, 0.05); margin: 16px 0px; font-family: &quot;HarmonyOS Sans SC&quot;, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><p style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); margin: 8px 0px; overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; line-height: 24px;">在 NapCat WebUI 的插件配置中可以修改以下参数：</p><table tabindex="0" style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: inherit; border-collapse: collapse; text-indent: 0px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; display: block; margin: 20px 0px; overflow-x: auto;"><thead style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); --vp-code-copy-copied-text-content: &quot;已复制&quot;;"><tr style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px 0px 0px; border-style: solid; border-color: rgb(46, 46, 50) rgb(228, 228, 231) rgb(228, 228, 231); --vp-code-copy-copied-text-content: &quot;已复制&quot;; background-color: rgb(27, 27, 31); transition: background-color 0.5s;"><th style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; text-align: left; font-size: 14px; font-weight: 600; color: inherit; background-color: rgb(32, 33, 39);">配置项</th><th style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; text-align: left; font-size: 14px; font-weight: 600; color: inherit; background-color: rgb(32, 33, 39);">默认值</th><th style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; text-align: left; font-size: 14px; font-weight: 600; color: inherit; background-color: rgb(32, 33, 39);">说明</th></tr></thead><tbody style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); --vp-code-copy-copied-text-content: &quot;已复制&quot;;"><tr style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px 0px 0px; border-style: solid; border-color: rgb(46, 46, 50) rgb(228, 228, 231) rgb(228, 228, 231); --vp-code-copy-copied-text-content: &quot;已复制&quot;; background-color: rgb(27, 27, 31); transition: background-color 0.5s;"><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;">调试服务端口</td><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;"><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">8998</code></td><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;">WebSocket 监听端口</td></tr><tr style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px 0px 0px; border-style: solid; border-color: rgb(46, 46, 50) rgb(228, 228, 231) rgb(228, 228, 231); --vp-code-copy-copied-text-content: &quot;已复制&quot;; background-color: rgb(32, 33, 39); transition: background-color 0.5s;"><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;">监听地址</td><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;"><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">127.0.0.1</code></td><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;"><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">请勿改为<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">0.0.0.0</code>，除非你清楚安全风险</strong></td></tr><tr style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px 0px 0px; border-style: solid; border-color: rgb(46, 46, 50) rgb(228, 228, 231) rgb(228, 228, 231); --vp-code-copy-copied-text-content: &quot;已复制&quot;; background-color: rgb(27, 27, 31); transition: background-color 0.5s;"><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;">启用认证</td><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;"><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">false</code></td><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;">启用后客户端需提供 token</td></tr><tr style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px 0px 0px; border-style: solid; border-color: rgb(46, 46, 50) rgb(228, 228, 231) rgb(228, 228, 231); --vp-code-copy-copied-text-content: &quot;已复制&quot;; background-color: rgb(32, 33, 39); transition: background-color 0.5s;"><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;">认证 Token</td><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;">空</td><td style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0.8px; border-style: solid; border-color: rgb(46, 46, 50); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-image: none 100% / 1 / 0 stretch; padding: 8px 16px; font-size: 14px;">客户端连接时的认证 token</td></tr></tbody></table><blockquote style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px 0px 0px 1.6px; border-style: solid; border-color: rgb(228, 228, 231) rgb(228, 228, 231) rgb(228, 228, 231) rgb(46, 46, 50); margin: 16px 0px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; padding-left: 16px; transition: border-color 0.5s; color: rgb(152, 152, 159);"><p style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); margin: 0px; overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; line-height: 24px; font-size: 14px; transition: color 0.5s; color: inherit;">默认配置仅监听本地回环地址<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">127.0.0.1</code>，只有本机可以连接，这是最安全的方式。如果将监听地址改为<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">0.0.0.0</code><span>&nbsp;</span>且未启用认证，等同于将插件管理权限完全暴露在网络中。</p></blockquote></details>

### 2. 插件项目：安装依赖

在你的插件项目中，安装 `napcat-plugin-debug-cli` 和 `ws` 作为开发依赖：



```
pnpm add -D napcat-plugin-debug-cli ws
```

TIP

如果你使用 [napcat-plugin-template](https://github.com/NapNeko/napcat-plugin-template) 模板，这些依赖和配置已经预置好，无需额外操作。

### 3. 配置 Vite 插件

在 `vite.config.ts` 中引入并添加 `napcatHmrPlugin`（模板中已预置）：



```
import { napcatHmrPlugin } from 'napcat-plugin-debug-cli/vite'

export default defineConfig({
  // ...其他配置
  plugins: [
    nodeResolve(),
    copyAssetsPlugin(),
    napcatHmrPlugin(),  // 构建完成后自动部署+重载
  ],
})
```

可选配置项：



```
napcatHmrPlugin({
  wsUrl: 'ws://192.168.1.100:8998',  // 调试服务地址（默认 ws://127.0.0.1:8998）
  token: 'mySecret',                  // 认证 token
  enabled: true,                       // 是否启用（默认 true）
})
```

关于远程连接

上述 `wsUrl` 指向远程地址仅作为示例。实际开发中 **建议在本地调试**（使用默认的 `ws://127.0.0.1:8998`）。如果你的 NapCat 运行在远程服务器上，请确保调试服务已启用 token 认证，并通过 SSH 隧道或防火墙限制访问，而非直接将端口暴露在公网。

### 4. 配置 npm scripts

在 `package.json` 中添加以下脚本（模板中已预置）：



```
{
  "scripts": {
    "build": "vite build",
    "push": "vite build",
    "dev": "vite build --watch"
  }
}
```

| 脚本    | 说明                                   |
| :------ | :------------------------------------- |
| `build` | 仅构建（如调试服务在线则自动部署）     |
| `push`  | 构建 + 自动部署到远程 + 重载插件       |
| `dev`   | 持续构建 + 每次构建后自动部署 + 热重载 |

不要使用 `deploy` 作为脚本名

pnpm 有内置的 `pnpm deploy` 命令（用于 workspace 部署），会覆盖 `package.json` 中的同名脚本，导致 `ERR_PNPM_CANNOT_DEPLOY` 错误。请使用 `push` 或其他名称代替。

> 因为 `napcatHmrPlugin` 已集成到 Vite 构建流程中，所以 `push` 和 `dev` 只需要 `vite build` / `vite build --watch` 即可，无需额外启动 CLI 进程。

## 开发流程

### 一、首次部署

首次开发时，使用 `push` 命令将插件构建并部署到 NapCat：



```
pnpm run push
```

执行 `vite build` 时，Vite 插件 `napcatHmrPlugin` 会在构建完成的 `writeBundle` 钩子中自动执行部署流程：

是否vite buildVite 构建 → 产出 dist/napcatHmrPlugin: writeBundle 钩子连接调试服务 WebSocketRPC: getDebugInfo()获取远程插件目录路径读取 dist/package.json 的 name 字段复制 dist/ → 远程插件目录/插件名/插件是否已注册？RPC: reloadPlugin(id)RPC: loadDirectoryPlugin(path)部署完成 ✓

#### 插件如何知道 NapCat 的插件目录？

Vite 插件 **不需要**你手动配置 NapCat 的安装路径。它的工作流程是：

1. 构建完成后，`napcatHmrPlugin` 连接 NapCat 端的 `napcat-plugin-debug` WebSocket 服务（默认 `ws://127.0.0.1:8998`）
2. 通过 JSON-RPC 调用 `getDebugInfo()` 方法，NapCat 返回当前实际的插件目录绝对路径
3. 读取 `dist/package.json` 的 `name` 字段，确定插件名称
4. 将 `dist/` 的内容复制到 `<远程插件目录>/<插件名>/` 下
5. 调用 `reloadPlugin` 或 `loadDirectoryPlugin` 触发热重载

这意味着无论 NapCat 安装在哪里，Vite 插件都能自动适配。整个流程在一个进程中完成，无需启动额外的 CLI。

### 二、持续开发（热重载）

完成首次部署后，使用 `dev` 命令进入热重载开发模式：



```
pnpm run dev
```

等价于带 `--watch` 的 `vite build`，只需要 **一个进程**：

1. **Vite watch 模式**：监听 `src/` 下的源码变更，自动重新构建到 `dist/`
2. **napcatHmrPlugin**：每次 `writeBundle`（构建完成）时自动部署到远程并调用热重载

修改源码Vite watch
重新构建 dist/napcatHmrPlugin
writeBundle 钩子WebSocket
JSON-RPCNapCat:
plugin_cleanup()
↓ 清除模块缓存
↓ import()
↓ plugin_init(ctx)

**改一行代码，保存即生效，无需重启 NapCat。**

NapCat 端在收到重载指令后会：

1. 调用旧插件实例的 `plugin_cleanup()` 释放资源
2. 清除 Node.js 模块缓存
3. 重新 `import()` 加载新代码
4. 调用 `plugin_init(ctx)` 初始化新实例

注意 plugin_cleanup

热重载依赖 `plugin_cleanup` 正确释放资源（如定时器、WebSocket 连接、事件监听器等）。如果不清理，可能导致重复注册或内存泄漏。请务必在 `plugin_cleanup` 中做好清理工作。

### 三、交互调试（REPL）

直接运行 CLI 而不带 `--watch` 或 `--deploy` 参数，会进入交互式 REPL 模式：



```
npx napcat-debug
```

在 REPL 中可以使用以下命令：

| 命令           | 说明                                   |
| :------------- | :------------------------------------- |
| `list`         | 列出所有插件及其加载状态               |
| `reload <id>`  | 手动重载指定插件                       |
| `load <id>`    | 加载指定插件                           |
| `unload <id>`  | 卸载指定插件                           |
| `info <id>`    | 查看插件详细信息（路径、版本、状态等） |
| `deploy [dir]` | 部署插件到远程并重载                   |
| `watch <dir>`  | 启动文件监听                           |
| `unwatch`      | 停止文件监听                           |
| `status`       | 查看调试服务状态和 HMR 状态            |
| `ping`         | 心跳测试（含延迟）                     |
| `quit`         | 退出                                   |

## CLI 完整用法

> CLI 是独立的调试工具，提供 REPL 交互、手动部署等功能。日常开发推荐使用上述 Vite 插件方式（`pnpm run dev`），无需手动运行 CLI。



```
napcat-debug [ws-url] [options]
```

| 参数                  | 说明                                       |
| :-------------------- | :----------------------------------------- |
| `ws://host:port`      | 调试服务地址（默认 `ws://127.0.0.1:8998`） |
| `-t, --token <token>` | 认证 token（当调试服务启用认证时使用）     |
| `-w, --watch <dir>`   | 监听指定目录，文件变更时自动热重载         |
| `-W, --watch-all`     | 监听远程插件目录下的所有插件               |
| `-d, --deploy [dir]`  | 部署插件到远程并重载（默认当前目录）       |
| `-v, --verbose`       | 详细输出（显示事件通知等）                 |

### 常用组合



```
# 默认连接本地调试服务，进入 REPL
napcat-debug

# 连接远程 NapCat
napcat-debug ws://192.168.1.100:8998

# 带认证连接
napcat-debug --token mySecret

# 构建后一键部署
napcat-debug --deploy .

# 监听 dist/ 目录自动热重载
napcat-debug --watch ./dist

# 监听远程所有插件（用于直接编辑远程文件的场景）
napcat-debug --watch-all
```

## 完整数据流



```
源码修改 (src/)
    │
    ▼
Vite --watch 重新构建 → dist/
    │
    ▼
napcatHmrPlugin writeBundle 钩子
  → 复制 dist/ → 远程插件目录/<pluginName>/
    │
    ▼
WebSocket → JSON-RPC reloadPlugin(pluginId)
    │
    ▼
NapCat 端:
  1. plugin_cleanup(ctx)     — 卸载旧实例、释放资源
  2. 清除 Node.js 模块缓存    — 确保 import() 拿到新代码
  3. import(pluginEntry)      — 重新加载插件模块
  4. plugin_init(ctx)         — 初始化新实例、注册路由等
```

## 常见问题

### 热重载后插件行为异常？

检查 `plugin_cleanup` 是否正确清理了所有资源：

- 定时器（`clearInterval` / `clearTimeout`）
- WebSocket 连接
- 文件监听器
- 事件监听器

### 连接调试服务失败？

1. 确认 `napcat-plugin-debug` 已在 NapCat 中启用
2. 检查端口是否正确（默认 `8998`）
3. 如果启用了认证，确认 token 正确（Vite 插件通过 `napcatHmrPlugin({ token: '...' })` 配置）
4. 如果 NapCat 运行在远程机器上，建议通过 SSH 隧道转发端口（如 `ssh -L 8998:127.0.0.1:8998 user@server`），而非直接开放端口

### 安全最佳实践

| 场景                                    | 建议做法                                            |
| :-------------------------------------- | :-------------------------------------------------- |
| 本地开发（NapCat 和编辑器在同一台机器） | 使用默认配置即可，无需额外设置                      |
| 远程开发（NapCat 在远程服务器）         | 使用 SSH 隧道转发端口，保持调试服务监听 `127.0.0.1` |
| 必须直接远程连接                        | 启用认证 + 设置高强度 token + 防火墙限制来源 IP     |
| 生产环境                                | **不要启用调试插件**，仅在开发阶段使用              |

### 部署时提示 `dist/` 不存在？

先运行 `pnpm run build` 构建项目，确保 `dist/` 目录已生成。



# 发布插件

字数

3186 字

阅读时间

14 分钟

本章节将详细介绍如何通过 CI/CD 自动化流程发布你的 NapCat 插件，并将其收录到官方插件索引中。

前提条件

- 你已经基于 [napcat-plugin-template](https://github.com/NapNeko/napcat-plugin-template) 创建了自己的插件仓库
- 插件代码已经开发完成，并可以正常构建
- 你有一个 GitHub 账号

## 整体流程

整个发布流程是自动化的，只需完成两步操作：**配置 Secrets** 和 **推送 tag**。

推送 v* tagCI 自动构建打包 zip创建 GitHub Release自动提交 PR\n到索引仓库索引仓库\nCI 自动审核维护者合并

模板仓库内置了两个 GitHub Actions 工作流，分别负责：

| 工作流文件         | 功能                                          |
| :----------------- | :-------------------------------------------- |
| `release.yml`      | 自动构建项目、打包 zip、创建 GitHub Release   |
| `update-index.yml` | Release 发布后，自动向插件索引仓库提交更新 PR |

## 第一步：填写插件元信息

编辑你的 `package.json`，确保以下字段已正确填写：



```
{
    "name": "napcat-plugin-your-name",      // 插件唯一 ID，必须以 napcat-plugin- 开头
    "plugin": "你的插件显示名",               // 在插件市场中显示的名称
    "version": "1.0.0",
    "description": "插件功能描述",
    "author": "你的名字",
    "napcat": {                              // 索引专用字段
        "tags": ["工具"],                    // 插件标签，见下方可用标签
        "minVersion": "4.14.0",             // 支持的最低 NapCat 版本
        "homepage": ""                       // 留空则默认使用仓库地址
    }
}
```

### 命名规范

- `name` 字段（即插件 ID）**必须**以 `napcat-plugin-` 开头
- 仅允许**小写字母、数字和短横线** `-`，禁止使用中文或大写字母
- 例如：`napcat-plugin-music`、`napcat-plugin-group-manager`

### 可用标签

以下标签可在 `napcat.tags` 数组中使用：

```
官方` `工具` `娱乐` `AI` `群管` `管理` `自动化` `语音` `表情` `撤回` `游戏` `音乐` `图片` `视频` `搜索` `翻译` `天气` `签到` `抽奖` `其他
```

## 第二步：配置仓库 Secrets

在你的**插件仓库**的 **Settings > Secrets and variables > Actions** 中，需要添加以下 Secret：

### 必选配置

| Secret 名称 | 说明                                                         |
| :---------- | :----------------------------------------------------------- |
| `INDEX_PAT` | GitHub Personal Access Token，用于自动 fork 索引仓库并提交 PR |

<details class="details custom-block" style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border: 1px solid rgba(0, 0, 0, 0); --vp-code-copy-copied-text-content: &quot;已复制&quot;; border-radius: 8px; padding: 16px 16px 8px; line-height: 24px; font-size: 14px; color: rgb(223, 223, 214); background-color: rgba(255, 255, 255, 0.05); margin: 16px 0px; font-family: &quot;HarmonyOS Sans SC&quot;, sans-serif; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; white-space: normal; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"><p style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); margin: 8px 0px; overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; line-height: 24px;"><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">推荐方式：Classic Token（简单快速）</strong></p><ol style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); list-style: decimal; margin: 16px 0px; padding: 0px 0px 0px 1.25rem; --vp-code-copy-copied-text-content: &quot;已复制&quot;;"><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">进入 GitHub<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Settings &gt; Developer settings &gt; Personal access tokens &gt; Tokens (classic)</strong></li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">点击<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Generate new token (classic)</strong></li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">设置名称，例如<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">napcat-plugin-index</code></li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">勾选<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">public_repo</code><span>&nbsp;</span>权限（位于<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">repo</code><span>&nbsp;</span>下）</li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">设置合理的过期时间</li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">点击<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Generate token</strong>，<strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">立即复制并保存</strong>，此 Token 只会显示一次</li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">回到你的插件仓库，在<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Settings &gt; Secrets and variables &gt; Actions</strong><span>&nbsp;</span>中添加名为<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">INDEX_PAT</code><span>&nbsp;</span>的 Secret，值为上一步复制的 Token</li></ol><p style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); margin: 8px 0px; overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; line-height: 24px;"><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">备选方式：Fine-grained Token（更精确的权限控制）</strong></p><ol style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); list-style: decimal; margin: 16px 0px; padding: 0px 0px 0px 1.25rem; --vp-code-copy-copied-text-content: &quot;已复制&quot;;"><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">进入 GitHub<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Settings &gt; Developer settings &gt; Personal access tokens &gt; Fine-grained tokens</strong></li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">点击<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Generate new token</strong></li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">设置名称，例如<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">napcat-plugin-index</code></li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;"><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Resource owner</strong><span>&nbsp;</span>选择你自己的账号</li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;"><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Repository access</strong><span>&nbsp;</span>选择<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">All repositories</strong>（因为 CI 会自动 fork 索引仓库到你的账号下）</li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">展开<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Repository permissions</strong>，设置以下权限：<ul style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); list-style: disc; margin: 8px 0px 0px; padding: 0px 0px 0px 1.25rem; --vp-code-copy-copied-text-content: &quot;已复制&quot;;"><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;;"><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Contents</strong>:<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">Read and write</code>（推送分支到 fork 仓库）</li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;"><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Pull requests</strong>:<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">Read and write</code>（创建 PR）</li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;"><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Workflows</strong>:<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">Read and write</code>（触发工作流）</li></ul></li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">设置合理的过期时间</li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">点击<span>&nbsp;</span><strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">Generate token</strong>，<strong style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-weight: 600; --vp-code-copy-copied-text-content: &quot;已复制&quot;;">立即复制并保存</strong></li><li style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); overflow-wrap: break-word; --vp-code-copy-copied-text-content: &quot;已复制&quot;; margin-top: 8px;">回到你的插件仓库，添加名为<span>&nbsp;</span><code style="box-sizing: border-box; --un-rotate: 0; --un-rotate-x: 0; --un-rotate-y: 0; --un-rotate-z: 0; --un-scale-x: 1; --un-scale-y: 1; --un-scale-z: 1; --un-skew-x: 0; --un-skew-y: 0; --un-translate-x: 0; --un-translate-y: 0; --un-translate-z: 0; --un-pan-x: ; --un-pan-y: ; --un-pinch-zoom: ; --un-scroll-snap-strictness: proximity; --un-ordinal: ; --un-slashed-zero: ; --un-numeric-figure: ; --un-numeric-spacing: ; --un-numeric-fraction: ; --un-border-spacing-x: 0; --un-border-spacing-y: 0; --un-ring-offset-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-shadow: 0 0 rgb(0 0 0 / 0); --un-shadow-inset: ; --un-shadow: 0 0 rgb(0 0 0 / 0); --un-ring-inset: ; --un-ring-offset-width: 0px; --un-ring-offset-color: #fff; --un-ring-width: 0px; --un-ring-color: rgb(147 197 253 / .5); --un-blur: ; --un-brightness: ; --un-contrast: ; --un-drop-shadow: ; --un-grayscale: ; --un-hue-rotate: ; --un-invert: ; --un-saturate: ; --un-sepia: ; --un-backdrop-blur: ; --un-backdrop-brightness: ; --un-backdrop-contrast: ; --un-backdrop-grayscale: ; --un-backdrop-hue-rotate: ; --un-backdrop-invert: ; --un-backdrop-opacity: ; --un-backdrop-saturate: ; --un-backdrop-sepia: ; --tw-border-spacing-x: 0; --tw-border-spacing-y: 0; --tw-translate-x: 0; --tw-translate-y: 0; --tw-rotate: 0; --tw-skew-x: 0; --tw-skew-y: 0; --tw-scale-x: 1; --tw-scale-y: 1; --tw-pan-x: ; --tw-pan-y: ; --tw-pinch-zoom: ; --tw-scroll-snap-strictness: proximity; --tw-gradient-from-position: ; --tw-gradient-via-position: ; --tw-gradient-to-position: ; --tw-ordinal: ; --tw-slashed-zero: ; --tw-numeric-figure: ; --tw-numeric-spacing: ; --tw-numeric-fraction: ; --tw-ring-inset: ; --tw-ring-offset-width: 0px; --tw-ring-offset-color: #fff; --tw-ring-color: rgb(59 130 246 / .5); --tw-ring-offset-shadow: 0 0 #0000; --tw-ring-shadow: 0 0 #0000; --tw-shadow: 0 0 #0000; --tw-shadow-colored: 0 0 #0000; --tw-blur: ; --tw-brightness: ; --tw-contrast: ; --tw-grayscale: ; --tw-hue-rotate: ; --tw-invert: ; --tw-saturate: ; --tw-sepia: ; --tw-drop-shadow: ; --tw-backdrop-blur: ; --tw-backdrop-brightness: ; --tw-backdrop-contrast: ; --tw-backdrop-grayscale: ; --tw-backdrop-hue-rotate: ; --tw-backdrop-invert: ; --tw-backdrop-opacity: ; --tw-backdrop-saturate: ; --tw-backdrop-sepia: ; --tw-contain-size: ; --tw-contain-layout: ; --tw-contain-paint: ; --tw-contain-style: ; border-width: 0px; border-style: solid; border-color: rgb(228, 228, 231); font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, &quot;Liberation Mono&quot;, &quot;Courier New&quot;, monospace; font-feature-settings: normal; font-variation-settings: normal; font-size: 12.25px; --vp-code-copy-copied-text-content: &quot;已复制&quot;; color: rgb(168, 177, 255); border-radius: 4px; padding: 3px 6px; background-color: rgba(101, 117, 133, 0.16); transition: color 0.25s, background-color 0.5s;">INDEX_PAT</code><span>&nbsp;</span>的 Secret</li></ol></details>

无需手动 fork 索引仓库

工作流会**自动完成**以下操作，你只需配置 `INDEX_PAT` 即可：

1. 通过 PAT 获取你的 GitHub 用户名
2. 自动 fork [napcat-plugin-index](https://github.com/NapNeko/napcat-plugin-index) 到你的账号下（如果尚未 fork）
3. 同步 fork 到上游最新状态
4. 将更新推送到你的 fork
5. 从 fork 向官方仓库发起 PR

### 可选配置（AI Release Note）

如果你希望每次发布时自动用 AI 生成 Release Note，可以额外配置：

| Secret 名称  | 必填 | 说明                                                         |
| :----------- | :--- | :----------------------------------------------------------- |
| `AI_API_URL` | 是   | 兼容 OpenAI 格式的 API 地址（如 `https://api.openai.com/v1/chat/completions`） |
| `AI_API_KEY` | 是   | 对应的 API 密钥                                              |
| `AI_MODEL`   | 否   | 模型名称，默认 `gpt-4o-mini`                                 |

TIP

不配置 AI 相关 Secret 不影响发布流程。未配置或 AI 调用失败时，会自动回退到默认的 Release Note 模板。

## 第三步：发布！

配置完成后，执行以下命令即可触发发布流程：



```
git tag v1.0.0
git push origin v1.0.0
```

推送 `v*` 格式的 tag 后，CI 将自动完成后续所有步骤。

你也可以在 GitHub 仓库页面的 **Actions** 标签页中，手动触发 `Build and Release` 工作流并填入版本号。

## CI 工作流详解

### release.yml — 自动构建与发布

这个工作流在你推送 `v*` tag 或手动触发时执行，完成以下任务：

Checkout 代码安装依赖读取版本号构建项目打包 zip生成 Release Note创建 GitHub Release触发索引更新

#### 核心步骤说明

**1. 版本号解析**

CI 会按优先级获取版本号：

- 手动触发时输入的版本号
- tag 名称（如 `v1.0.0` → `1.0.0`）
- 自动生成时间戳版本（兜底）

**2. 构建与打包**

执行 `pnpm run build` 构建项目后，自动将产物打包为 zip 文件：



```
release/
├── index.mjs       # 插件主入口
├── package.json    # 精简后的 package.json
└── webui/          # WebUI 构建产物（如有）
```

修改插件名称

如果你的插件不叫 `napcat-plugin-template`，需要修改 `release.yml` 中 **Prepare release package** 步骤的 `PLUGIN_NAME` 环境变量为你的实际插件名。

**3. Release Note 生成**

- 如果配置了 AI Secret → 使用 AI 根据 commit 记录生成结构化 Release Note
- 如果未配置或 AI 调用失败 → 使用 `.github/prompt/default.md` 模板回退
- 如果模板也不存在 → 直接使用 commit log

你可以自定义 AI 的提示词，在 `.github/prompt/release_note_prompt.txt` 文件中修改即可。

**4. 触发索引更新**

Release 创建成功后，会通过 `workflow_dispatch` 自动触发 `update-index.yml` 工作流。

为什么要手动触发？

因为 GitHub Actions 中，由默认 `GITHUB_TOKEN` 创建的 Release **不会**自动触发 `on.release` 事件。所以 `release.yml` 在最后一步通过 API 手动触发索引更新工作流。

### update-index.yml — 自动更新插件索引

这个工作流在 Release 发布后被触发，完成以下任务：

读取 package.json\n元信息构造下载链接验证资源可达自动 fork\n索引仓库同步 fork\n到上游最新更新\nplugins.v4.json通过 fork\n提交 PR

#### 核心步骤说明

**1. 提取元信息**

CI 会从你的 `package.json` 中自动读取：

| 字段来源                          | 映射到索引字段 |
| :-------------------------------- | :------------- |
| `name`                            | `id`           |
| `plugin` (或 `name`)              | `name`         |
| `description`                     | `description`  |
| `author`                          | `author`       |
| `napcat.tags`                     | `tags`         |
| `napcat.minVersion`               | `minVersion`   |
| `napcat.homepage` (或 `homepage`) | `homepage`     |

下载链接会自动构造为：`https://github.com/<你的仓库>/releases/download/<版本号>/<插件ID>.zip`

**2. 验证资源可达**

CI 会 curl 检查下载链接是否可访问。如果第一次失败，会等待 30 秒后重试（因为 Release 资源上传可能有延迟）。两次都失败则跳过索引更新。

**3. 自动 fork 与同步**

CI 会通过 GitHub API 自动完成以下操作：

- 使用 `INDEX_PAT` 获取你的 GitHub 用户名
- 检查你账号下是否已有 `napcat-plugin-index` 的 fork，如果没有则自动创建
- 调用 `merge-upstream` API 将 fork 同步到官方索引仓库的最新状态

**4. 提交 PR**

CI 会 checkout 官方索引仓库，使用 Node.js 脚本更新 `plugins.v4.json` 文件：

- 如果插件 ID 已存在 → 更新该条目
- 如果是新插件 → 追加到列表

然后通过 [peter-evans/create-pull-request](https://github.com/peter-evans/create-pull-request) Action 的 `push-to-fork` 功能，将更新推送到你的 fork 仓库，并从 fork 向官方索引仓库发起 PR。

为什么要通过 fork 提交？

你的 PAT 没有官方索引仓库 [napcat-plugin-index](https://github.com/NapNeko/napcat-plugin-index) 的写入权限，这是正常的。工作流自动将你的 fork 作为中转，完整流程是：



```
自动 fork 索引仓库 → 同步到上游最新 → 推送更新到 fork → 从 fork 向官方仓库发起 PR
```

## 索引仓库的自动审核

你的 PR 提交到 [napcat-plugin-index](https://github.com/NapNeko/napcat-plugin-index) 后，索引仓库有一套完善的自动化审核流程：

| 工作流                                 | 触发条件                  | 功能                                                         |
| :------------------------------------- | :------------------------ | :----------------------------------------------------------- |
| **PR 自动审核** (`validate-pr.yml`)    | PR 修改 `plugins.v4.json` | 校验 JSON 格式、字段完整性、ID 唯一性、版本号格式、包名规范、下载链接可达性 |
| **校验结果评论** (`comment-on-pr.yml`) | 审核完成后                | 在 PR 中评论校验结果，自动打标签（新插件 / 插件更新 / 插件删除） |
| **AI 安全审计** (`security-scan.yml`)  | PR 更新/新增插件          | 下载插件 zip 进行静态代码分析 + AI 深度审查，检测高危代码    |
| **链接定时巡检** (`check-links.yml`)   | 每天定时执行              | 检查所有已索引插件的下载链接是否有效，失效时自动创建 Issue   |

### 审核结果

- **校验通过 + 无安全风险** → 低风险 PR 自动合并，或等待维护者审核合并
- **校验未通过** → Bot 会在 PR 下评论具体的错误信息，你需要修复后重新提交
- **发现高危代码** → 需要人工审核

常见校验失败原因

- 插件 ID 不符合 `napcat-plugin-xxx` 格式
- 必填字段为空
- `downloadUrl` 不是可访问的 `.zip` 链接
- 版本号不符合 [semver](https://semver.org/) 格式
- 插件包内 `package.json` 的 `name` 与索引中的 `id` 不一致

## 版本更新发布

后续版本更新流程与首次发布完全一致：

1. 修改 `package.json` 中的 `version`（可选，CI 会以 tag 为准）

2. 推送新 tag：

   

   ```
   git tag v1.1.0
   git push origin v1.1.0
   ```

3. CI 自动构建、发布、更新索引

## 自定义 Release Note 模板

### 默认模板

在 `.github/prompt/default.md` 中自定义默认 Release Note 模板，支持 `{VERSION}` 占位符：



```
## {VERSION}

### 核心更新
- 更新版本至 {VERSION}

### 安装说明
1. 下载 `你的插件名.zip`
2. 解压到 NapCat 的 `plugins` 目录
3. 重启 NapCat
```

### AI Prompt 自定义

在 `.github/prompt/release_note_prompt.txt` 中自定义 AI 生成 Release Note 的 system prompt，支持 `{VERSION}` 和 `{PREV_VERSION}` 占位符。

## 常见问题

### Q: 能不能手动提交 PR 到索引仓库？

不建议。索引仓库已采用全自动审核机制，手动提交的 PR 将不会被合入。请使用模板仓库的 CI 自动提交流程。

如果你不想使用模板仓库，请自行实现等效的 CI 工作流。

### Q: 不用模板仓库，自己写 CI 行不行？

可以，只要你的 CI 能够：

1. 构建产物并打包为 `<插件ID>.zip`（包含 `index.mjs`、`package.json`、`webui/`）
2. 创建 GitHub Release 并上传 zip
3. 向 `NapNeko/napcat-plugin-index` 仓库提交 PR，更新 `plugins.v4.json`

可以参考模板仓库的 [release.yml](https://github.com/NapNeko/napcat-plugin-template/blob/main/.github/workflows/release.yml) 和 [update-index.yml](https://github.com/NapNeko/napcat-plugin-template/blob/main/.github/workflows/update-index.yml)。

### Q: INDEX_PAT Token 过期了怎么办？

重新生成一个新的 PAT，更新仓库 Secret 即可。建议设置较长的过期时间，或使用 Fine-grained Token。

### Q: 提交 PR 到索引仓库时报 403 / Permission denied？

请检查以下几点：

1. **PAT 权限不足**：Classic Token 需要 `public_repo` 权限；Fine-grained Token 需要 `Contents: Read and write` + `Pull requests: Read and write` 权限，且 Repository access 需要选择 **All repositories**
2. **PAT 已过期**：重新生成并更新 Secret
3. **工作流版本过旧**：确保你使用的是最新版模板的 `update-index.yml`，新版工作流已内置自动 fork 功能，无需手动 fork 或修改工作流

### Q: Release 发布了但索引没更新？

检查 Actions 页面中 `update-index.yml` 的运行日志。常见原因：

- `INDEX_PAT` 未配置或已过期
- PAT 权限不足（见上一条）
- 下载链接验证失败（Release 资源尚未上传完成）
- `package.json` 中缺少必要字段

### Q: 下载链接验证失败？

GitHub Release 资源上传可能有延迟。CI 已内置 30 秒重试机制。如果仍失败，可以等资源上传完成后手动触发 `update-index.yml` 工作流。

# NapCat 插件开发 API 参考文档

字数

1094 字

阅读时间

5 分钟

## NapCatPluginContext

插件的核心上下文对象，包含了几乎所有你需要的功能。在所有生命周期函数中作为第一个参数传入。

### 属性一览

| 属性               | 类型                                     | 说明                   |
| :----------------- | :--------------------------------------- | :--------------------- |
| `core`             | `NapCatCore`                             | NapCat 底层核心实例    |
| `oneBot`           | `NapCatOneBot11Adapter`                  | OneBot11 协议适配器    |
| `actions`          | `ActionMap`                              | OneBot11 Action 调用器 |
| `pluginName`       | `string`                                 | 当前插件名称           |
| `pluginPath`       | `string`                                 | 插件所在目录           |
| `configPath`       | `string`                                 | 插件配置文件路径       |
| `dataPath`         | `string`                                 | 插件数据存储目录       |
| `NapCatConfig`     | `NapCatConfigClass`                      | 配置构建工具类         |
| `adapterName`      | `string`                                 | 适配器名称             |
| `pluginManager`    | `IPluginManager`                         | 插件管理器             |
| `logger`           | `PluginLogger`                           | 日志记录器             |
| `router`           | `PluginRouterRegistry`                   | 路由注册器             |
| `getPluginExports` | `<T>(pluginId: string) => T | undefined` | 获取其他插件的导出     |

### `ctx.actions`

OneBot11 Action 调用器。

**调用签名**:



```
ctx.actions.call(actionName, params, ctx.adapterName, ctx.pluginManager.config)
```

- `actionName`: Action 名称（字符串）
- `params`: 请求参数（不同 action 参数不同，无参数时传 `void 0`）
- `adapter`: 适配器名称，使用 `ctx.adapterName`
- `config`: 网络配置，使用 `ctx.pluginManager.config`

**常用 Action 示例**:



```
// 通用发送消息 (推荐用法)
const params: OB11PostSendMsg = {
    message: 'Hello',
    message_type: 'group',
    group_id: '123456',
};
await ctx.actions.call('send_msg', params, ctx.adapterName, ctx.pluginManager.config);

// 获取登录信息
const info = await ctx.actions.call('get_login_info', void 0, ctx.adapterName, ctx.pluginManager.config);

// 获取群列表
const groups = await ctx.actions.call('get_group_list', void 0, ctx.adapterName, ctx.pluginManager.config);

// 获取版本信息
const version = await ctx.actions.call('get_version_info', void 0, ctx.adapterName, ctx.pluginManager.config);

// 撤回消息
await ctx.actions.call('delete_msg', { message_id }, ctx.adapterName, ctx.pluginManager.config);
```

### `ctx.logger`

带有插件名称前缀的日志记录器，实现 `PluginLogger` 接口。

| 方法         | 说明                              |
| :----------- | :-------------------------------- |
| `log(...)`   | 输出普通日志                      |
| `debug(...)` | 输出调试日志（仅 debug 模式可见） |
| `info(...)`  | 输出信息日志                      |
| `warn(...)`  | 输出警告日志                      |
| `error(...)` | 输出错误日志                      |

### `ctx.pluginManager`

插件管理器接口 (`IPluginManager`)，提供对其他插件的管理能力。

| 方法                                    | 返回值                    | 说明                     |
| :-------------------------------------- | :------------------------ | :----------------------- |
| `getPluginPath()`                       | `string`                  | 获取插件根目录           |
| `getPluginConfig()`                     | `PluginStatusConfig`      | 获取插件启用状态配置     |
| `getAllPlugins()`                       | `PluginEntry[]`           | 获取所有已扫描的插件列表 |
| `getLoadedPlugins()`                    | `PluginEntry[]`           | 获取所有已加载的插件列表 |
| `getPluginInfo(pluginId)`               | `PluginEntry | undefined` | 获取指定插件信息         |
| `setPluginStatus(pluginId, enable)`     | `Promise<void>`           | 启用/禁用插件            |
| `loadPluginById(pluginId)`              | `Promise<boolean>`        | 加载指定插件             |
| `unregisterPlugin(pluginId)`            | `Promise<void>`           | 注销插件                 |
| `uninstallPlugin(pluginId, cleanData?)` | `Promise<void>`           | 卸载并删除插件           |
| `reloadPlugin(pluginId)`                | `Promise<boolean>`        | 重新加载插件             |
| `loadDirectoryPlugin(dirname)`          | `Promise<void>`           | 从目录加载插件           |
| `getPluginDataPath(pluginId)`           | `string`                  | 获取插件数据目录         |
| `getPluginConfigPath(pluginId)`         | `string`                  | 获取插件配置目录         |

### `ctx.router`

WebUI 和 API 路由注册器 (`PluginRouterRegistry`)。

#### 需要鉴权的路由

路径前缀：`/api/Plugin/ext/<plugin-id>/`

| 方法     | 签名                      | 说明                     |
| :------- | :------------------------ | :----------------------- |
| `api`    | `(method, path, handler)` | 注册指定 HTTP 方法的 API |
| `get`    | `(path, handler)`         | 注册 GET API             |
| `post`   | `(path, handler)`         | 注册 POST API            |
| `put`    | `(path, handler)`         | 注册 PUT API             |
| `delete` | `(path, handler)`         | 注册 DELETE API          |

#### 无需鉴权的路由

路径前缀：`/plugin/<plugin-id>/api/`

| 方法           | 签名                      | 说明                         |
| :------------- | :------------------------ | :--------------------------- |
| `apiNoAuth`    | `(method, path, handler)` | 注册指定 HTTP 方法的开放 API |
| `getNoAuth`    | `(path, handler)`         | 注册 GET 开放 API            |
| `postNoAuth`   | `(path, handler)`         | 注册 POST 开放 API           |
| `putNoAuth`    | `(path, handler)`         | 注册 PUT 开放 API            |
| `deleteNoAuth` | `(path, handler)`         | 注册 DELETE 开放 API         |

#### 页面与静态文件

| 方法          | 签名                   | 路径前缀                               | 说明                 |
| :------------ | :--------------------- | :------------------------------------- | :------------------- |
| `page`        | `(pageDefinition)`     | `/plugin/<plugin-id>/page/<path>`      | 注册页面             |
| `pages`       | `(pageDefinitions)`    | 同上                                   | 批量注册页面         |
| `static`      | `(urlPath, localPath)` | `/plugin/<plugin-id>/files/<urlPath>/` | 注册本地静态文件服务 |
| `staticOnMem` | `(urlPath, files)`     | `/plugin/<plugin-id>/mem/<urlPath>/`   | 注册内存静态文件服务 |

### `ctx.NapCatConfig`

配置构建工具类，提供便捷的静态方法生成 `PluginConfigItem`：

| 方法          | 签名                                                         | 说明                    |
| :------------ | :----------------------------------------------------------- | :---------------------- |
| `text`        | `(key, label, defaultValue?, description?, reactive?)`       | 文本输入                |
| `number`      | `(key, label, defaultValue?, description?, reactive?)`       | 数字输入                |
| `boolean`     | `(key, label, defaultValue?, description?, reactive?)`       | 布尔开关                |
| `select`      | `(key, label, options, defaultValue?, description?, reactive?)` | 下拉单选                |
| `multiSelect` | `(key, label, options, defaultValue?, description?, reactive?)` | 下拉多选                |
| `html`        | `(content)`                                                  | HTML 展示               |
| `plainText`   | `(content)`                                                  | 纯文本展示              |
| `combine`     | `(...items)`                                                 | 合并多个配置项为 Schema |

### `ctx.getPluginExports`



```
getPluginExports<T = PluginModule>(pluginId: string): T | undefined
```

获取其他已加载插件的导出模块对象。支持泛型参数以获得类型提示。

## OB11Message 消息对象

当收到 `plugin_onmessage` 事件时，传递的消息对象。

### 属性

| 属性           | 类型                  | 说明               |
| :------------- | :-------------------- | :----------------- |
| `message_id`   | `number`              | 消息 ID            |
| `user_id`      | `number`              | 发送者 QQ          |
| `group_id`     | `number`              | 群号（私聊时为 0） |
| `message_type` | `'private' | 'group'` | 消息类型           |
| `raw_message`  | `string`              | 原始消息文本       |
| `message`      | `MessageSegment[]`    | 消息段数组         |
| `sender`       | `OB11Sender`          | 发送者信息         |
| `time`         | `number`              | 时间戳             |

------

更多详细类型定义请参考 TypeScript 提示或查阅 `napcat-types` 包。



# NapCat Types 定义参考

字数

1195 字

阅读时间

7 分钟

> 本文档列出了开发插件时最常用的类型定义。完整的类型定义请参考 napcat-types 源码。

注意

随着 napcat-types 的更新，部分内容可能已过时，本文仅供参考。请以 napcat-types 最新源码为准。

## 插件上下文



```
export interface NapCatPluginContext {
    core: NapCatCore;
    oneBot: NapCatOneBot11Adapter;
    actions: ActionMap;
    pluginName: string;
    pluginPath: string;
    configPath: string;
    dataPath: string;
    NapCatConfig: NapCatConfigClass;
    adapterName: string;
    pluginManager: IPluginManager;
    logger: PluginLogger;
    router: PluginRouterRegistry;
    getPluginExports: <T = PluginModule>(pluginId: string) => T | undefined;
}
```

## 插件模块定义



```
export interface PluginModule<T extends OB11EmitEventContent = OB11EmitEventContent, C = unknown> {
    // 必选：插件初始化
    plugin_init: (ctx: NapCatPluginContext) => void | Promise<void>;
    // 可选：消息/事件处理（需通过 event.post_type 判断事件类型）
    plugin_onmessage?: (ctx: NapCatPluginContext, event: OB11Message) => void | Promise<void>;
    // 可选：事件处理（非消息类 OneBot 事件）
    plugin_onevent?: (ctx: NapCatPluginContext, event: T) => void | Promise<void>;
    // 可选：插件卸载/清理
    plugin_cleanup?: (ctx: NapCatPluginContext) => void | Promise<void>;
    // 可选：配置 Schema（用于 WebUI 自动生成配置面板）
    plugin_config_schema?: PluginConfigSchema;
    // 可选：配置 UI（同 plugin_config_schema）
    plugin_config_ui?: PluginConfigSchema;
    // 可选：自定义配置读取
    plugin_get_config?: (ctx: NapCatPluginContext) => C | Promise<C>;
    // 可选：自定义配置保存
    plugin_set_config?: (ctx: NapCatPluginContext, config: C) => void | Promise<void>;
    // 可选：配置 UI 控制器（运行时动态控制配置界面）
    plugin_config_controller?: (
        ctx: NapCatPluginContext,
        ui: PluginConfigUIController,
        initialConfig: Record<string, unknown>
    ) => void | (() => void) | Promise<void | (() => void)>;
    // 可选：配置变更回调
    plugin_on_config_change?: (
        ctx: NapCatPluginContext,
        ui: PluginConfigUIController,
        key: string,
        value: unknown,
        currentConfig: Record<string, unknown>
    ) => void | Promise<void>;
}
```

## 配置 UI 定义



```
export type PluginConfigSchema = PluginConfigItem[];

export interface PluginConfigItem {
    key: string;
    type: 'string' | 'number' | 'boolean' | 'select' | 'multi-select' | 'html' | 'text';
    label: string;
    description?: string;
    default?: unknown;
    options?: { label: string; value: string | number }[];
    placeholder?: string;
    /** 标记此字段为响应式：值变化时触发 schema 刷新 */
    reactive?: boolean;
    /** 是否隐藏此字段 */
    hidden?: boolean;
}
```

## 配置 UI 控制器



```
export interface PluginConfigUIController {
    updateSchema: (schema: PluginConfigSchema) => void;
    updateField: (key: string, field: Partial<PluginConfigItem>) => void;
    removeField: (key: string) => void;
    addField: (field: PluginConfigItem, afterKey?: string) => void;
    showField: (key: string) => void;
    hideField: (key: string) => void;
    getCurrentConfig: () => Record<string, unknown>;
}
```

## 日志接口



```
export interface PluginLogger {
    log(...args: unknown[]): void;
    debug(...args: unknown[]): void;
    info(...args: unknown[]): void;
    warn(...args: unknown[]): void;
    error(...args: unknown[]): void;
}
```

## 路由相关类型



```
export type HttpMethod = 'get' | 'post' | 'put' | 'delete' | 'patch' | 'all';

export interface PluginRouterRegistry {
    // 需要鉴权
    api(method: HttpMethod, path: string, handler: PluginRequestHandler): void;
    get(path: string, handler: PluginRequestHandler): void;
    post(path: string, handler: PluginRequestHandler): void;
    put(path: string, handler: PluginRequestHandler): void;
    delete(path: string, handler: PluginRequestHandler): void;
    // 无需鉴权
    apiNoAuth(method: HttpMethod, path: string, handler: PluginRequestHandler): void;
    getNoAuth(path: string, handler: PluginRequestHandler): void;
    postNoAuth(path: string, handler: PluginRequestHandler): void;
    putNoAuth(path: string, handler: PluginRequestHandler): void;
    deleteNoAuth(path: string, handler: PluginRequestHandler): void;
    // 页面
    page(page: PluginPageDefinition): void;
    pages(pages: PluginPageDefinition[]): void;
    // 静态文件
    static(urlPath: string, localPath: string): void;
    staticOnMem(urlPath: string, files: MemoryStaticFile[]): void;
}

export interface PluginHttpRequest {
    path: string;
    method: string;
    query: Record<string, string | string[] | undefined>;
    body: unknown;
    headers: Record<string, string | string[] | undefined>;
    params: Record<string, string>;
    raw: unknown;
}

export interface PluginHttpResponse {
    status(code: number): PluginHttpResponse;
    json(data: unknown): void;
    send(data: string | Buffer): void;
    setHeader(name: string, value: string): PluginHttpResponse;
    sendFile(filePath: string): void;
    redirect(url: string): void;
    raw: unknown;
}

export type PluginNextFunction = (err?: unknown) => void;
export type PluginRequestHandler = (
    req: PluginHttpRequest,
    res: PluginHttpResponse,
    next: PluginNextFunction
) => void | Promise<void>;

export interface PluginPageDefinition {
    path: string;        // 页面路径
    title: string;       // 页面标题
    icon?: string;       // 页面图标
    htmlFile: string;    // HTML 文件路径（相对于插件目录）
    description?: string; // 页面描述
}

export type MemoryFileGenerator = () => string | Buffer | Promise<string | Buffer>;

export interface MemoryStaticFile {
    path: string;                                    // 文件路径
    content: string | Buffer | MemoryFileGenerator;  // 文件内容或生成器
    contentType?: string;                            // MIME 类型
}
```

## 插件管理器接口



```
export interface IPluginManager {
    readonly config: NetworkAdapterConfig;
    getPluginPath(): string;
    getPluginConfig(): PluginStatusConfig;
    getAllPlugins(): PluginEntry[];
    getLoadedPlugins(): PluginEntry[];
    getPluginInfo(pluginId: string): PluginEntry | undefined;
    setPluginStatus(pluginId: string, enable: boolean): Promise<void>;
    loadPluginById(pluginId: string): Promise<boolean>;
    unregisterPlugin(pluginId: string): Promise<void>;
    uninstallPlugin(pluginId: string, cleanData?: boolean): Promise<void>;
    reloadPlugin(pluginId: string): Promise<boolean>;
    loadDirectoryPlugin(dirname: string): Promise<void>;
    getPluginDataPath(pluginId: string): string;
    getPluginConfigPath(pluginId: string): string;
}

export interface PluginEntry {
    id: string;
    fileId: string;
    name?: string;
    version?: string;
    description?: string;
    author?: string;
    pluginPath: string;
    entryPath?: string;
    packageJson?: PluginPackageJson;
    enable: boolean;
    loaded: boolean;
    runtime: PluginRuntime;
}

export type PluginRuntimeStatus = 'loaded' | 'error' | 'unloaded';

export interface PluginRuntime {
    status: PluginRuntimeStatus;
    error?: string;
    module?: PluginModule;
    context?: NapCatPluginContext;
}
```

## NapCatConfig 构建器



```
export interface INapCatConfigStatic {
    text(key: string, label: string, defaultValue?: string, description?: string, reactive?: boolean): PluginConfigItem;
    number(key: string, label: string, defaultValue?: number, description?: string, reactive?: boolean): PluginConfigItem;
    boolean(key: string, label: string, defaultValue?: boolean, description?: string, reactive?: boolean): PluginConfigItem;
    select(key: string, label: string, options: { label: string; value: string | number }[],
        defaultValue?: string | number, description?: string, reactive?: boolean): PluginConfigItem;
    multiSelect(key: string, label: string, options: { label: string; value: string | number }[],
        defaultValue?: (string | number)[], description?: string, reactive?: boolean): PluginConfigItem;
    html(content: string): PluginConfigItem;
    plainText(content: string): PluginConfigItem;
    combine(...items: PluginConfigItem[]): PluginConfigSchema;
}
```

## 事件类型枚举



```
export enum EventType {
    META = "meta_event",
    REQUEST = "request",
    NOTICE = "notice",
    MESSAGE = "message",
    MESSAGE_SENT = "message_sent"
}
```

在 `plugin_onmessage` 中需要通过 `event.post_type` 判断事件类型：



```
if (event.post_type !== EventType.MESSAGE) return;
```

## ActionMap

`ActionMap` 提供 `call` 方法调用 OneBot11 Action：



```
// call 方法签名
actions.call(
    actionName: string,     // Action 名称
    params: unknown,        // 请求参数，无参数时传 void 0
    adapter: string,        // 适配器名称，使用 ctx.adapterName
    config: NetworkAdapterConfig  // 网络配置，使用 ctx.pluginManager.config
): Promise<any>
```

## 消息发送参数



```
export interface OB11PostSendMsg {
    message_type?: "private" | "group";  // 消息类型
    user_id?: string;                    // 用户 QQ 号（私聊时）
    group_id?: string;                   // 群号（群聊时）
    message: string | MessageSegment[];  // 消息内容
    auto_escape?: boolean | string;      // 是否作为纯文本发送
}
```