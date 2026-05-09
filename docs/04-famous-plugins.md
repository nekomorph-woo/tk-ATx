# 历史上著名的 Atom 插件

> Atom 在 2015-2020 年间拥有蓬勃的插件生态，许多插件的创新影响了整个编辑器领域。本文盘点最具影响力的 Atom 插件。

---

## 目录

1. [下载量/Star 排行榜](#1-下载量star-排行榜)
2. [开创性功能的插件](#2-开创性功能的插件)
3. [按类别分类的插件](#3-按类别分类的插件)
4. [被移植到 VS Code 的插件](#4-被移植到-vs-code-的插件)
5. [社区必装清单](#5-社区必装清单)
6. [GitHub 官方开发的核心包](#6-github-官方开发的核心包)
7. [生态系统影响力故事](#7-生态系统影响力故事)

---

## 1. 下载量/Star 排行榜

| 插件 | 近似 Star 数 | 下载量 | 类别 |
|------|:---:|:---:|------|
| **minimap** | 4,000-5,000 | 7M+ | 可视化 |
| **emmet** | 4,000-6,000 | 6M+ | 效率工具 |
| **file-icons** | 4,000-5,000 | 5M+ | UI |
| **atom-material-ui** | 3,000-5,000 | 4M+ | 主题/UI |
| **pigments** | 3,000-4,000 | 3M+ | 可视化 |
| **linter** | 2,000-3,000 | 5M+ | Linting |
| **atom-beautify** | 2,000-3,000 | 5M+ | 格式化 |
| **vim-mode-plus** | 2,000-3,000 | 2M+ | 编辑 |
| **autocomplete-plus** | 2,000+ | 8M+ (内置) | 自动补全 |
| **git-plus** | 1,500-2,000 | 3M+ | Git |
| **script** | 1,000-1,500 | 2M+ | 效率工具 |
| **teletype** | 1,000-1,500 | 1M+ | 协作 |
| **atom-ide-ui** | 1,000+ | 2M+ | IDE 功能 |

---

## 2. 开创性功能的插件

### atom-minimap — Minimap 概念

- **仓库**：[atom-minimap/minimap](https://github.com/atom-minimap/minimap)
- **意义**：Atom 的 Minimap 实现首次将其作为**可扩展平台**设计，支持 17+ 子插件（git diff、linter 标记、选中高亮、书签、查找替换、颜色渲染等）。VS Code 2017 年内置的 minimap 直接受此影响。Atom minimap 还支持独立模式和画布渲染，以及 ASCII art 地标注释文化。

### Teletype — P2P 协作编辑

- **仓库**：[atom/teletype](https://github.com/atom/teletype)（GitHub 官方）
- **意义**：首批生产级 **对等网络（P2P）协作代码编辑** 实现之一，使用 WebRTC 和 **CRDT（无冲突复制数据类型）**。不同于 Google Docs（依赖服务器），Teletype 直接在客户端间同步编辑。源自 Martin Kleppmann 的 **Automerge** 研究。VS Code Live Share（2017）受此影响。

### linter — Provider 模式的 Linting 架构

- **仓库**：[steelbrain/linter](https://github.com/steelbrain/linter)
- **意义**：`linter` 包（v2.0, 2016）引入了 UI、核心逻辑和语言特定 provider 的清晰分离。这种 **provider 模式** 后来成为所有编辑器 linting 的主流架构。影响了 VS Code 的诊断 provider API 和 Sublime Text Linter。

### atom-ide-ui — LSP 在编辑器中的早期采用

- **仓库**：[atom/atom-ide-ui](https://github.com/atom/atom-ide-ui)（GitHub 官方）
- **意义**：GitHub Atom IDE 项目的一部分，从 Facebook 的 Nuclide 中提取 UI 层，创建可复用的 IDE 组件（诊断面板、悬停提示、代码透镜、签名帮助、查找引用）。是最早的 **LSP（语言服务器协议）** 消费实现之一。

### Nuclide — 单体 IDE 包

- **仓库**：[facebookarchive/nuclide](https://github.com/facebookarchive/nuclide)（Facebook）
- **意义**：Facebook 基于 Atom 构建的统一 IDE，主要用于 Hack、Flow、React 和 React Native 开发。内置调试器、远程开发、Mercurial 支持。2017 年模块化为 atom-ide-ui，成为 LSP-based IDE 功能的基础。证明了 Electron 编辑器可以支持严肃的 IDE 工作流。

---

## 3. 按类别分类的插件

### 代码导航

| 插件 | 用途 | 意义 |
|------|------|------|
| **symbols-view** | 项目级符号搜索（Cmd+Shift+O） | 内置包，使用 ctags 索引 |
| **atom-ternjs** | JS/Node 跳转定义、查找引用、重命名 | LSP 出现前的 JS 智能导航 |
| **hyperclick** | Cmd/Ctrl+点击跳转到定义 | 推广了"点击导航"UX 模式 |
| **symbols-tree-view** | 侧栏代码符号树 | IDE 级大纲视图，影响 VS Code 的 Outline |
| **goto-definition** | 跳转到符号定义 | LSP 普及前的早期实现 |

### 自动补全 / IntelliSense

| 插件 | 用途 | 意义 |
|------|------|------|
| **autocomplete-plus** | 核心补全引擎 | 引入了 **provider API** 模式，被 VS Code 采纳 |
| **atom-ternjs** | JavaScript IntelliSense | LSP 前的 JS 智能补全 |
| **autocomplete-clang** | C/C++ 补全 | 证明编译语言补全可行 |
| **autocomplete-python** | Python 补全 | Python 补全提供者 |
| **autocomplete-module-import** | npm 包导入补全 | 独特功能：从 import 语句搜索安装 npm 包 |

### Git 集成

| 插件 | 用途 | 意义 |
|------|------|------|
| **git-plus** | 在编辑器内运行 Git 命令 | 首批"编辑器内完整 Git"体验 |
| **merge-conflicts** | 可视化合并冲突解决 | 开创了键盘快捷键导航冲突的 UX |
| **git-blame** | 逐行 Git blame 注释 | 使 blame 成为编辑器一等公民 |
| **git-time-machine** | 通过时间线浏览文件历史 | 独特创新，概念延续到 GitLens Timeline |
| **tree-view-git-status** | 文件树中显示 Git 状态 | 使 Git 感知成为文件浏览器标配 |

### UI / 主题

| 插件 | 用途 | 意义 |
|------|------|------|
| **one-dark-ui** | 默认暗色主题 | 调色板成为暗色模式**事实标准**，直接影响 VS Code 默认暗色主题 |
| **atom-material-ui** | Material Design UI 主题 | 证明设计系统主题可行，移植为 VS Code Material Theme |
| **seti-ui** | 文件图标 + 极简 UI | **首创文件图标概念**，直接移植到 VS Code |
| **file-icons** | 文件类型特定图标 | 成为 vscode-icons 和 Material Icon Theme 的模型 |
| **nord-atom-ui** | Nord 配色方案 | 跨编辑器和终端传播的配色系统 |
| **city-lights-syntax** | 设计工作室级语法主题 | 证明语法主题可以具有策展美学品质 |

### 语言支持

| 插件 | 用途 | 意义 |
|------|------|------|
| **language-babel** | ES6, JSX, Flow, TypeScript | VS Code 存在前的一流 React 开发环境 |
| **atom-typescript** | TypeScript 语言支持 | 社区驱动的 TS 支持，早于大多数编辑器 |
| **language-graphql** | GraphQL 语法高亮 | 早期 GraphQL 编辑器支持 |
| **react** | React JSX 支持 | 半官方 React 包 |
| **turbo-javascript** | 增强 JavaScript 语法 | 超越 Atom 默认的 JS 体验 |

### 效率工具

| 插件 | 用途 | 意义 |
|------|------|------|
| **emmet** | HTML/CSS 快速编写 | VS Code 后来**原生内置** Emmet，证明其不可或缺 |
| **atom-beautify** | 通用代码美化器 | 影响了 Prettier 的诞生 |
| **docblockr** | 自动生成 JSDoc 注释 | 移植为 VS Code Document This |
| **highlight-selected** | 高亮所有选中文本 | VS Code 后来原生内置此功能 |
| **sort-lines** | 按各种条件排序行 | 简单但必备，移植到几乎所有编辑器 |
| **todo-show** | 查找并列出 TODO/FIXME 注释 | 推广了"TODO 聚合器"概念 |
| **trailing-spaces** | 高亮和移除尾随空格 | 现在是大多数编辑器的基本功能 |
| **prettier-atom** | Prettier 代码格式化集成 | 帮助 Prettier 成为主流格式化工具 |

### Linting

| 插件 | 用途 | 意义 |
|------|------|------|
| **linter** (核心) | 基础 linting 框架 | 创建者 steelbrain（Adeel Mujeeb），当时还是青少年 |
| **linter-ui-default** | 默认 linting UI | 行内 + 侧栏 + 底部面板三层显示成为通用 linting UX |
| **linter-eslint** | ESLint 集成 | 最流行的 linter provider，证明 provider 模式可大规模运作 |

---

## 4. 被移植到 VS Code 的插件

| Atom 插件 | VS Code 等价物 | 影响方式 |
|-----------|---------------|----------|
| **Minimap** | 内置 minimap (2017) | VS Code 原生内置 |
| **Emmet** | 内置 Emmet | VS Code 原生内置，无需扩展 |
| **File Icons** | vscode-icons, Material Icon Theme | 数百万安装的扩展类别 |
| **git-blame** | GitLens (30M+ 安装) | 精神继承者 |
| **git-time-machine** | GitLens Timeline | 时间线概念被吸收 |
| **atom-beautify** | Prettier, Beautify | Prettier 成为继任者 |
| **linter** | ESLint, SonarLint, 内置诊断 | provider API 模式被采纳 |
| **Color Picker** | 内置颜色选择器 | VS Code 原生添加 |
| **Pigments** | Color Highlight | 移植为 VS Code 扩展 |
| **Teletype** | VS Code Live Share | 协作先驱 |
| **vim-mode-plus** | VSCodeVim, neovim | Vim 仿真成为标配 |
| **atom-material-ui** | Material Theme | 直接移植 |
| **sync-settings** | 内置 Settings Sync | VS Code 原生添加 |
| **autocomplete-plus** | 内置 IntelliSense | provider API 模式被采纳 |
| **docblockr** | Document This | 概念移植 |

---

## 5. 社区必装清单

### 绝对必备

1. **minimap** — 普遍安装
2. **file-icons** — 普遍安装
3. **linter** + **linter-eslint** — JS 开发者必备
4. **emmet** — Web 开发者必备
5. **pigments** — 可视化颜色显示
6. **atom-beautify** 或 **prettier-atom** — 代码格式化
7. **git-plus** — 不离开编辑器使用 Git
8. **project-manager** — 多项目切换

### 强烈推荐

9. **highlight-selected** — 双击高亮所有匹配
10. **color-picker** — CSS GUI 颜色选择器
11. **docblockr** — JSDoc 生成
12. **sort-lines** — 行排序工具
13. **trailing-spaces** — 空白卫生
14. **merge-conflicts** — Git 冲突解决
15. **sync-settings** — 通过 GitHub Gist 跨机器同步配置

### 高级用户最爱

16. **vim-mode-plus** — Vim 仿真
17. **jumpy** — AceJump 式键盘导航
18. **todo-show** — 项目 TODO 聚合
19. **zen** — 无干扰写作模式
20. **markdown-preview-enhanced** — 富 Markdown 预览（数学、图表、导出）
21. **tablr** — 在编辑器中像电子表格一样编辑 CSV/TSV
22. **script** — 从编辑器直接运行代码
23. **platformio-ide-terminal** — 嵌入式终端

---

## 6. GitHub 官方开发的核心包

| 包 | 描述 | 为什么重要 |
|----|------|------------|
| **autocomplete-plus** | 核心补全引擎 | 内置；provider API 模式影响深远 |
| **autocomplete-snippets** | 代码片段补全集成 | 统一了片段到补全流程 |
| **bracket-matcher** | 内置括号匹配 | 现在是标配功能 |
| **find-and-replace** | 内置查找替换 | 项目级正则搜索 |
| **fuzzy-finder** | Cmd+P 文件查找器 | 标准化了模糊查找 UX |
| **tree-view** | 内置文件浏览器 | 标准文件树实现 |
| **teletype** | 实时协作编辑 | P2P CRDT 协作先驱 |
| **atom-ide-ui** | IDE UI 组件 | 早期 LSP 消费者 |
| **atom-languageclient** | LSP 客户端库 | 让创建语言服务器变简单 |
| **one-dark-ui/one-light-ui** | 默认主题 | 暗色主题成为行业标准 |
| **one-dark-syntax/one-light-syntax** | 默认语法主题 | 被广泛模仿 |
| **symbols-view** | 符号导航 | 内置代码符号搜索 |
| **snippets** | 代码片段系统 | CSON 格式片段 |
| **spell-check** | 内置拼写检查 | |
| **markdown-preview** | 内置 Markdown 预览 | 标准化实时预览 |
| **settings-view** | 包管理 UI | `apm` 包管理器 GUI |

---

## 7. 生态系统影响力故事

### Atom 作为 VS Code 的"概念验证"

Atom 基于 Electron（最初叫 "Atom Shell"）构建，Electron 正是 GitHub 创建的。Microsoft 构建 VS Code 时选择了 Electron 作为运行时——Atom 开创的同一框架。Microsoft 2018 年收购 GitHub，实际上意味着 VS Code 继承了 Atom 的遗产。许多 Atom 社区成员和包作者后来迁移到 VS Code 扩展开发。

### Linter 生态：从 Atom 到行业标准

`linter` 包由 **steelbrain**（Adeel Mujeeb）创建，他当时还是一名青少年。在他领导下，atom-linter 组织维护了 50+ linter provider。provider 模式的架构——分离 UI、核心和语言特定逻辑——真正具有创新性。VS Code 的诊断 provider API、Sublime Text Linter、甚至 LSP 诊断模型的某些方面都可以追溯到 Atom 的 linter 架构。

### Minimap 的 ASCII Art 文化

Atom minimap 社区发展出了独特的 **ASCII art 地标注释** 文化——使用 Figlet 等工具创建大文本注释，在 minimap 中可见以导航大文件。这是 minimap 的一种创造性使用，没有其他编辑器的 minimap 复制了这种文化实践。

### Facebook 的投资与撤出

Facebook 的 **Nuclide**（2015）是对 Atom 作为 IDE 平台的巨大赌注。2017 年 Facebook 将 Nuclide 模块化为 Atom IDE 计划。到 2020 年，Facebook 归档了 Nuclide，转移了对 Atom 工具的关注。这一轨迹反映了更广泛的行业趋势：Atom 作为 IDE 平台被 VS Code 取代。

### CRDT 突破

Teletype 是 **CRDT** 在生产开发工具中的首批真实应用之一。将 CRDT（源自 Martin Kleppmann 在剑桥的研究组）用于 P2P 文本编辑在技术上具有突破性。Teletype 源码仍然是 CRDT 协作编辑的参考实现。

### Emmet：从社区包到内置功能

Emmet（前身 Zen Coding）从独立工具开始，在 Atom 中成为最受欢迎的包之一。它在 Atom 和 Sublime Text 中的成功如此之大，以至于 VS Code 简单地**内置了它**。这是 Atom 包证明某功能如此不可或缺以至于成为原生功能的最清晰例子。

### "可黑客文本编辑器"的遗产

Atom 的标语——"21 世纪的可黑客文本编辑器"——不仅仅是营销。其包系统允许修改几乎所有方面：语法、主题、键映射、片段、树视图、状态栏等。创建包的低门槛（JavaScript + CSS + HTML，无需编译）大幅降低了入门门槛。这种"一切皆包"的哲学直接影响了 VS Code 的扩展模型。

---

## 参考链接

| 资源 | URL |
|------|-----|
| atom-minimap/minimap | https://github.com/atom-minimap/minimap |
| awesome-atom 策展列表 | https://github.com/mehcode/awesome-atom |
| AtomPackages 策展列表 | https://github.com/stevelinus/AtomPackages |
| Atom 包 VS Code 等价物讨论 | https://www.reddit.com/r/vscode/comments/dpw2ij/ |
| Facebook Nuclide 博客 | https://engineering.fb.com/2015/06/23/developer-tools/building-nuclide-a-unified-developer-experience/ |
| Atom IDE UI 介绍 | https://nuclide.io/blog/2017/09/12/Introducing-Atom-IDE-UI/ |
| facebookarchive/nuclide | https://github.com/facebookarchive/nuclide |
| Atom Flight Manual - Snippets | https://flight-manual.atom-editor.cc/using-atom/sections/snippets/ |
