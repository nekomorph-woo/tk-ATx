# Atom 插件开发 - 环境准备

> **重要提示：** Atom 已于 2022 年 12 月 15 日被 GitHub 正式归档。社区分支 [Pulsar](https://pulsar-edit.dev) 继续了开发维护。本文档同时覆盖两个生态。

---

## 目录

1. [系统要求](#1-系统要求)
2. [安装 Atom / Pulsar](#2-安装-atom--pulsar)
3. [生成新插件包](#3-生成新插件包)
4. [包目录结构](#4-包目录结构)
5. [关键配置文件](#5-关键配置文件)
6. [APM / PPM 包管理器命令](#6-apm--ppm-包管理器命令)
7. [以开发模式运行](#7-以开发模式运行)

---

## 1. 系统要求

### 操作系统

| 平台 | 最低版本 |
|------|----------|
| macOS | 10.9+ (Mavericks 及以上) |
| Windows | 7+ (64-bit) |
| Linux | 64-bit (多发行版支持) |

### 运行时依赖

- **Node.js**：Atom 自带 Node.js 运行时，无需单独安装。较新版本 (1.6x+) 基于 Electron 12+，内含 Node.js 14.x。Pulsar 延续此模式。
- **Git**：包管理 (`apm`/`ppm`) 的 `install`、`publish` 等命令需要。
- **Python + C++ 编译工具**：编译原生 Node 模块 (`node-gyp`) 需要。macOS 上为 Xcode Command Line Tools；Windows 上为 Visual Studio Build Tools。

### 磁盘/内存

- 编辑器安装：~300-500 MB
- 原生模块编译：额外 ~1 GB 磁盘空间

---

## 2. 安装 Atom / Pulsar

### Atom（已归档）

Atom 于 2022 年 12 月 15 日被 GitHub 正式归档：

- 预编译二进制文件仍可从 [atom/atom GitHub Releases](https://github.com/atom/atom/releases) 下载
- Atom 包注册中心 (`atom.io`) 已停止接受新包发布
- `apm install` 因注册中心关闭而失效
- Teletype 协作功能已停止
- 不再有安全更新

> 来源：[Sunsetting Atom - The GitHub Blog](https://github.blog/news-insights/product-news/sunsetting-atom/)

### Pulsar（社区分支）—— 推荐

Pulsar 是 Atom 的社区驱动延续版本，维护于 [pulsar-edit.dev](https://pulsar-edit.dev)。

**安装方式：**
- 下载地址：[https://pulsar-edit.dev](https://pulsar-edit.dev)
- GitHub 仓库：[https://github.com/pulsar-edit/Pulsar](https://github.com/pulsar-edit/Pulsar)

**与 Atom 的关键差异：**

| 项目 | Atom | Pulsar |
|------|------|--------|
| 维护方 | GitHub/Microsoft | 社区（开放治理） |
| Electron 版本 | 冻结在旧版本 | 持续升级（v25+） |
| 遥测 | 包含 GitHub 遥测 | **已移除** |
| 品牌名 | "Atom" | "Pulsar" |
| 包管理器 | `apm` | `ppm` |
| 配置目录 | `~/.atom` | `~/.pulsar` |
| 包注册中心 | atom.io（已关闭） | web.pulsar-edit.dev |
| 遥测/分析 | GitHub 遥测（已停止） | 无默认遥测 |
| Electron 版本 | 冻结（较旧） | 活跃更新 |

**从 Atom 迁移到 Pulsar：**

1. 备份 `~/.atom` 目录
2. 从 pulsar-edit.dev 安装 Pulsar
3. 复制配置文件到 `~/.pulsar`：
   - `config.cson` → `~/.pulsar/config.cson`
   - `keymap.cson` → `~/.pulsar/keymap.cson`
   - `styles.less` → `~/.pulsar/styles.less`
   - `snippets.cson` → `~/.pulsar/snippets.cson`
4. 通过 `ppm install <package-name>` 重新安装包

---

## 3. 生成新插件包

### 使用内置包生成器（推荐）

1. 打开 Pulsar/Atom
2. 打开命令面板（`Cmd+Shift+P` / `Ctrl+Shift+P`）
3. 搜索 **"Generate Package"**
4. 输入包名（如 `my-name-word-count`）
5. 生成器创建目录骨架；本地开发安装使用 `pulsar -p install` + `pulsar -p link --dev`

### 使用命令行

```bash
# 生成新包（Atom）
apm init --package <package-name>
apm generate package <package-name>

# 生成语言包
apm generate language <language-name>

# 生成语法主题
apm generate theme <theme-name>

# 生成新包（Pulsar）
ppm init --package <package-name>
ppm generate package <package-name>
```

---

## 4. 包目录结构

生成的包具有如下布局：

```
my-package/
  grammars/          # 语言语法定义（TextMate/CSON 格式）
  keymaps/           # 键绑定定义（JSON/CSON）
  lib/               # 主要源代码
    my-package.js         # 主入口（通过 package.json "main" 导出）
    my-package-view.js    # 视图类
  menus/             # 菜单定义（JSON/CSON）
  spec/              # 测试文件（Jasmine）—— 必须以 -spec.js 结尾
    my-package-spec.js
  snippets/          # 代码片段定义（CSON）
  styles/            # 样式表（Less 或 CSS）
    my-package.less
  index.js           # 备选主入口
  package.json       # 包清单
  README.md          # 包文档
```

并非所有包都需要全部目录。生成器默认不创建 `snippets` 和 `grammars` 目录。

---

## 5. 关键配置文件

### package.json

最重要的文件。Atom/Pulsar 特有字段（超出标准 Node.js 的部分）：

```json
{
  "name": "my-package",
  "main": "./lib/my-package",
  "version": "0.0.0",
  "description": "A short description of your package",
  "activationCommands": {
    "atom-workspace": "my-package:toggle"
  },
  "activationHooks": [
    "language-javascript:grammar-used"
  ],
  "repository": "https://github.com/user/my-package",
  "license": "MIT",
  "engines": {
    "atom": ">=1.0.0 <2.0.0"
  },
  "styles": ["styles/my-package.less"],
  "keymaps": ["keymaps/my-package.json"],
  "menus": ["menus/my-package.json"],
  "snippets": ["snippets/language.cson"],
  "dependencies": {},
  "consumedServices": {
    "status-bar": {
      "versions": {
        "^1.0.0": "consumeStatusBar"
      }
    }
  },
  "providedServices": {
    "my-service": {
      "description": "My custom service",
      "versions": {
        "1.0.0": "provideMyService"
      }
    }
  },
  "configSchema": {
    "showIcons": {
      "type": "boolean",
      "default": true,
      "title": "Show Icons",
      "description": "Toggle icon visibility."
    }
  }
}
```

**关键字段说明：**

| 字段 | 说明 |
|------|------|
| `main` | 入口文件路径，默认 `index.js` 或 `index.coffee` |
| `activationCommands` | 惰性激活——仅在用户触发指定命令时加载 |
| `activationHooks` | 基于钩子的惰性激活（推荐） |
| `styles`/`keymaps`/`menus`/`snippets` | 控制加载顺序的数组 |
| `engines.atom` | Atom/Pulsar 版本兼容范围 |
| `consumedServices`/`providedServices` | 服务 API 声明 |
| `configSchema` | 用户可配置设置的 JSON Schema |
| `atomTestRunner` | 自定义测试运行器路径 |

### Keymaps（`keymaps/my-package.json`）

```json
{
  "atom-workspace": {
    "ctrl-alt-o": "my-package:toggle"
  }
}
```

键映射将按键绑定到命令。CSS 选择器决定绑定生效的作用域。

### Menus（`menus/my-package.json`）

**应用菜单：**

```json
{
  "menu": [
    {
      "label": "Packages",
      "submenu": [
        {
          "label": "My Package",
          "submenu": [
            { "label": "Toggle", "command": "my-package:toggle" }
          ]
        }
      ]
    }
  ]
}
```

**上下文菜单：**

```json
{
  "context-menu": {
    "atom-text-editor": [
      { "label": "Toggle my-package", "command": "my-package:toggle" }
    ]
  }
}
```

### Stylesheets（`styles/my-package.less`）

Atom/Pulsar 推荐使用 **Less**：

```less
@import "ui-variables";

.my-package {
  padding: 10px;
  color: @text-color;
  background-color: @app-background-color;
}
```

通过命令面板运行 **Styleguide**（`Cmd+Ctrl+Shift+G`）查看所有可用 UI 组件。

---

## 6. APM / PPM 包管理器命令

Atom 使用 `apm`；Pulsar 使用 `ppm`，命令完全相同。

Pulsar 也可以通过 `pulsar -p` 调用包管理器；如果命令不可用，先在 Pulsar 菜单中执行 **Pulsar > Install Shell Commands**。

### 常用命令

```bash
# 安装/卸载
apm install <package-name>
apm uninstall <package-name>

# 列出已安装包
apm list
apm ls

# 搜索
apm search <query>
apm view <package-name>

# 生成
apm init --package <name>
apm generate package <name>
apm generate language <name>
apm generate theme <name>

# 安装本地开发的 Pulsar 插件（开发窗口）
cd /path/to/<plugin-name>
pulsar -p install
pulsar -p link --dev
pulsar --dev /path/to/<plugin-name>

# 安装本地开发的 Pulsar 插件（日常使用）
cd /path/to/<plugin-name>
pulsar -p install
pulsar -p link

# 测试
apm test

# 发布
apm login
apm publish           # 发布当前版本
apm publish minor     # 升级次版本并发布
apm publish major     # 升级主版本并发布
apm publish patch     # 升级补丁版本并发布

# 其他
apm star <package-name>
apm stars
apm update
apm featured
apm clean
```

---

## 7. 以开发模式运行

### 命令行方式

```bash
# 以开发模式打开目录
atom --dev /path/to/your/package

# 运行包测试
atom --test /path/to/package
atom --test spec/my-package-spec.js
atom --test --timeout 60 ./spec
```

### 从编辑器内部

- **View > Developer > Open in Dev Mode**

### 热重载

- 命令面板运行 **`window:reload`**（`Cmd+Shift+F5`）重新加载窗口
- 也可以用 `Cmd+R`（类似浏览器刷新）

### 调试工具

| 工具 | 快捷键 | 用途 |
|------|--------|------|
| Developer Console | `Alt+Cmd+I` / `Ctrl+Shift+I` | 标准 Chrome DevTools |
| Keybinding Resolver | `Cmd+Shift+.` | 查看哪个键绑定匹配 |
| Cursor Scope Inspector | 命令面板 "Toggle Cursor Scope" | 显示光标处的活跃作用域 |

### 运行测试

- **编辑器内**：命令面板 `Window: Run Package Specs`（`Alt+Cmd+Ctrl+P`）
- **命令行**：`atom --test spec/`
- Atom 默认使用 **Jasmine 1.3** 进行测试，支持 `waitsForPromise()` 等异步辅助函数

---

## 参考链接

| 资源 | URL |
|------|-----|
| Atom Flight Manual（社区镜像） | https://flight-manual.atom-editor.cc/hacking-atom/ |
| Atom Flight Manual（Wayback Machine） | https://web.archive.org/web/20221215131241/https://flight-manual.atom.io/hacking-atom/ |
| Pulsar 文档 | https://docs.pulsar-edit.dev |
| Pulsar 源码 | https://github.com/pulsar-edit/Pulsar |
| Pulsar 包注册中心 | https://web.pulsar-edit.dev |
| Sunsetting Atom 公告 | https://github.blog/news-insights/product-news/sunsetting-atom/ |
| package-generator | https://github.com/atom/package-generator |
