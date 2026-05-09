# Atom 插件开发 - 进阶开发

> 涵盖服务系统、Tree-sitter、Linter 框架、Autocomplete API、Git 集成、性能优化、Electron 内部机制等高级主题。

---

## 目录

1. [服务系统（Service System）](#1-服务系统)
2. [Tree-sitter 集成](#2-tree-sitter-集成)
3. [Linter 框架](#3-linter-框架)
4. [Autocomplete Provider API](#4-autocomplete-provider-api)
5. [Git 集成 API](#5-git-集成-api)
6. [遥测与度量模式](#6-遥测与度量模式)
7. [性能分析与优化](#7-性能分析与优化)
8. [Electron 内部机制](#8-electron-内部机制)
9. [从 Atom 迁移到 Pulsar](#9-从-atom-迁移到-pulsar)
10. [构建语言语法与主题](#10-构建语言语法与主题)

---

## 1. 服务系统

Atom 的服务系统是一种依赖注入机制，允许包之间松耦合通信。

### 提供服务

在 `package.json` 中声明：

```json
{
  "providedServices": {
    "my-custom-service": {
      "description": "Description of the service",
      "versions": {
        "1.0.0": "provideMyCustomService"
      }
    }
  }
}
```

在主模块中实现：

```js
module.exports = {
  provideMyCustomService() {
    return {
      doSomething() { console.log("Service called!"); },
      getData() { return { key: "value" }; }
    };
  }
};
```

### 消费服务

在 `package.json` 中声明：

```json
{
  "consumedServices": {
    "my-custom-service": {
      "versions": { "1.0.0": "consumeMyCustomService" }
    }
  }
}
```

在主模块中实现：

```js
let myService;
module.exports = {
  consumeMyCustomService(service) {
    myService = service;
    myService.doSomething();
  }
};
```

### 版本协商

使用语义版本号。消费者可指定支持的版本范围（`^`、`~` 或精确版本），Atom 的 `ServiceHub` 自动匹配兼容的提供者与消费者。

### 关键内置服务

| 服务名 | 提供包 | 用途 |
|--------|--------|------|
| `autocomplete.provider` | `autocomplete-plus` | 注册自动补全提供者 |
| `status-bar` | `status-bar` | 添加状态栏元素 |
| `atom.fuzzy-finder` | `fuzzy-finder` | 文件/符号搜索集成 |
| `linter` | `linter` | 注册 linter 提供者 |
| `tree-view` | `tree-view` | 树视图扩展点 |
| `atom-ide.diagnostic-ui` | `atom-ide-ui` | IDE 诊断显示 |

### 核心优势

1. **松耦合** — 包之间从不直接 `require()` 对方
2. **可选依赖** — 提供者未安装时，消费者优雅降级
3. **多实现** — 多个包可提供同一服务接口
4. **生命周期管理** — Atom 在适当的时机调用提供/消费函数

---

## 2. Tree-sitter 集成

Tree-sitter 最初由 GitHub 的 Max Brunsfeld 为 Atom 编辑器创建，是一个增量解析系统。

### 架构

```
tree-sitter (C 库, 编译为 .so/.dylib/.dll)
    ↓
tree-sitter npm 包 (原生 Node.js 插件)
    ↓
Atom 语言包 (如 language-typescript)
    ↓
Atom 内部语法引擎
    ↓
语法高亮、代码折叠、符号导航
```

### 查询语言示例

```scheme
; highlights.scm — 将语法节点映射到高亮作用域
(keyword) @keyword
(string) @string
(comment) @comment
(function_declaration name: (identifier) @function)
(call_expression function: (identifier) @function.call)
```

### 启用的功能

- **语法高亮** — 比 TextMate 正则语法更精确
- **代码折叠** — Tree-sitter 知道代码块的结构边界
- **选择更大语法节点** — `Ctrl+W` 扩展选择到父级语法节点
- **符号导航** — 跳转到函数/类定义
- **增量解析** — 只重新解析变更部分，O(edit_size) 时间复杂度

### 后续影响

Tree-sitter 在 Atom 归档后被 Neovim、Helix、Zed、Sublime Text 4 和 Emacs 采纳。项目主页：[github.com/tree-sitter/tree-sitter](https://github.com/tree-sitter/tree-sitter)

---

## 3. Linter 框架

### 核心组件

| 组件 | 角色 |
|------|------|
| `linter` | 基础 UI 包，在编辑器中显示 lint 消息 |
| `atom-linter` | 辅助库：`exec()`、`parse()`、临时文件管理 |
| `linter-ui-default` | 默认 UI（行内高亮、底部面板、侧边栏气泡） |
| 各语言 linter | `linter-eslint`、`linter-pylint` 等 |

### Linter v2 消息对象格式

```js
{
  type: 'error',                    // 'error' | 'warning' | 'info'
  text: 'Unexpected token',         // 主消息文本
  filePath: '/abs/path/to/file.js', // 文件路径
  range: [[0, 0], [0, 5]],         // 0 索引的行列范围
  description: 'Longer description', // 详细描述（支持 Markdown）
  url: 'https://docs.example.com',  // 参考链接
  solutions: [                      // 自动修复支持
    {
      title: 'Remove token',
      position: [[0, 0], [0, 5]],
      currentText: 'token',
      replaceWith: '',
      priority: 1
    }
  ]
}
```

### Provider 定义

```js
module.exports = {
  provideLinter() {
    return {
      name: 'my-linter',
      grammarScopes: ['source.js', 'source.ts'],
      scope: 'file',
      lintsOnChange: true,
      lint(textEditor) {
        return Promise.resolve([
          { type: 'error', text: 'Unexpected token', range: [[0, 0], [0, 5]] }
        ]);
      }
    };
  }
};
```

在 `package.json` 中注册：

```json
{
  "providedServices": {
    "linter": {
      "versions": { "2.0.0": "provideLinter" }
    }
  }
}
```

### 使用 atom-linter 辅助库

```js
const { exec } = require('atom-linter');

exec('eslint', ['--format', 'json', filePath], { timeout: 10000 })
  .then(output => {
    const results = JSON.parse(output.stdout);
    // 转换为 linter 消息对象...
  });
```

---

## 4. Autocomplete Provider API

`autocomplete-plus` 是 Atom 内置的自动补全引擎。

### 定义 Provider

```js
const provider = {
  selector: '.source.js, .source.ts',
  disableForSelector: '.source.js .comment',
  inclusionPriority: 1,
  excludeLowerPriority: true,
  suggestionPriority: 2,
  filterSuggestions: true,

  getSuggestions({ editor, bufferPosition, scopeDescriptor, prefix, activatedManually }) {
    return [
      { text: 'myFunction()', type: 'function' },
      { text: 'myVariable', type: 'variable' },
    ];
  },

  getSuggestionDetailsOnSelect(suggestion) {
    return Promise.resolve({ ...suggestion, description: 'More details...' });
  },

  onDidInsertSuggestion({ editor, triggerPosition, suggestion }) { /* ... */ },
  dispose() { /* 清理 */ }
};
```

### Suggestion 对象完整字段

```js
{
  text: 'someText',                            // 插入的文本（与 snippet 互斥）
  snippet: 'myFunction(${1:arg1}, ${2:arg2})', // 带 tab-stop 的代码片段
  displayText: 'someText',                     // UI 显示文本
  replacementPrefix: 'so',                      // 要替换的前缀
  type: 'function',                             // 图标类型：variable, constant, property,
                                                //   value, method, function, class, type,
                                                //   keyword, tag, snippet, import, require
  leftLabel: 'String',                          // 左侧标签（如返回类型）
  leftLabelHTML: '<i>...</i>',                  // HTML 版本
  rightLabel: 'myModule',                       // 右侧标签
  rightLabelHTML: '<span>...</span>',           // HTML 版本
  description: 'A doc-string summary',          // 列表底部描述
  descriptionMoreURL: 'https://docs.example.com', // "More..." 链接
  characterMatchIndices: [0, 1, 2]             // 前缀字符高亮索引
}
```

### 注册方式

```json
{
  "providedServices": {
    "autocomplete.provider": {
      "versions": { "4.0.0": "provide" }
    }
  }
}
```

```js
module.exports = {
  provide() {
    return {
      selector: '.source.js',
      getSuggestions(options) { /* ... */ }
    };
  }
};
```

---

## 5. Git 集成 API

### GitRepository 类（核心 API）

通过 `atom.project` 访问，不要直接实例化：

```js
const repo = atom.project.getRepositories()[0];

if (repo) {
  repo.isPathIgnored(filePath);
  repo.getWorkingDirectory();
  repo.relativize(filePath);
  repo.getStatus().then(statuses => { /* ... */ });
  repo.getReferences().then(refs => { /* ... */ });
  repo.getBranches().then(branches => { /* ... */ });
  repo.getAheadBehindCount('main', 'origin/main')
    .then(({ ahead, behind }) => { /* ... */ });
}
```

### atom.github 包服务

更丰富的 GitHub 集成（PR 管理、Issue 跟踪等）：

```json
{
  "consumedServices": {
    "atom.github": { "versions": { "0.0.1": "consumeGitHub" } }
  }
}
```

### 可用操作

| 操作 | 方法 |
|------|------|
| 暂存文件 | `gitHubService.stageFiles(paths)` |
| 取消暂存 | `gitHubService.unstageFiles(paths)` |
| 提交 | `gitHubService.commit(message)` |
| 切换分支 | `repo.checkoutReference(refName)` |
| 获取/推送 | `repo.fetch()` / `repo.push()` |
| 查看差异 | `repo.getDiffStats()` |

---

## 6. 遥测与度量模式

### Atom 原生遥测架构

```
atom/metrics (Atom 包)
  |  使用
  v
atom/telemetry (npm: telemetry-github)
  |  发送到
  v
GitHub 内部分析管道
```

### telemetry-github API

```js
import StatsStore from 'telemetry-github';

const store = new StatsStore('atom', '1.24.1', false, getAccessToken, {
  reportingFrequency: 86400,
  logInDevMode: false,
  verboseMode: false
});
```

### 记录度量

```js
// 计数器
store.incrementCounter('command:toggle-sidebar');

// 自定义事件
await store.addCustomEvent('file-open', { grammar: 'javascript', fileSize: 1024 });

// 计时器
store.addTiming('appStartup', 42, { phase: 'main' });

// 用户偏好
store.setOptOut(true);   // 退出
store.setOptOut(false);  // 加入
```

### 第三方包的遥测实践

```js
module.exports = {
  trackEvent(eventName, metadata = {}) {
    if (!atom.config.get('my-package.enableTelemetry')) return;
    fetch('https://analytics.example.com/track', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ eventName, metadata, timestamp: new Date().toISOString() })
    }).catch(() => {});
  }
};
```

---

## 7. 性能分析与优化

### Chrome DevTools

```bash
# 打开 DevTools
# View > Developer > Toggle Developer Tools (Ctrl+Shift+I / Cmd+Option+I)
```

| 面板 | 用途 |
|------|------|
| **Performance** | CPU 分析、时间线、帧率 |
| **Memory** | 堆快照、分配时间线、对比分析 |
| **Network** | API 调用监控 |
| **Rendering** | 绘制分析、布局抖动检测 |

### Atom 特有分析

```js
// 在 DevTools 控制台中
atom.getLoadSettings()              // 加载计时信息
atom.packages.getLoadedPackages()   // 列出所有已加载包
```

### 常见性能陷阱与修复

**1. 过度 DOM 操作：**

```js
// 差：循环中创建 DOM
items.forEach(item => {
  container.appendChild(document.createElement('div'));
});

// 好：使用 DocumentFragment
const fragment = document.createDocumentFragment();
items.forEach(item => {
  fragment.appendChild(document.createElement('div'));
});
container.appendChild(fragment);
```

**2. 未防抖的事件处理：**

```js
// 差：每次击键都 lint
editor.onDidChange(() => { this.lint(editor); });

// 好：防抖
const { debounce } = require('underscore-plus');
editor.onDidChange(debounce(() => { this.lint(editor); }, 300));
```

**3. 阻塞主进程：**

```js
// 差：同步 I/O
const content = require('fs').readFileSync(filePath, 'utf8');

// 好：异步 I/O
const content = await fs.readFile(filePath, 'utf8');
```

**4. 惰性加载：** 始终使用 `activationCommands` 或 `activationHooks` 避免拖慢启动。

---

## 8. Electron 内部机制

### 进程架构

```
主进程 (Node.js)
  |  管理
  v
BrowserWindow (Chromium 渲染器)
  |  包含
  v
渲染进程 (你的 Atom 包代码在此运行)
```

- **主进程**：管理应用生命周期、创建窗口、处理系统级操作
- **渲染进程**：包含 DOM (Chromium)，可访问 Node.js API 子集

### IPC（进程间通信）

```js
const { ipcRenderer } = require('electron');
ipcRenderer.send('channel-name', { data: 'value' });
ipcRenderer.invoke('async-channel', arg1, arg2).then(result => { /* ... */ });
```

Atom 通常通过命令系统封装 IPC，包很少直接使用 `ipcRenderer`。

### 包开发者需要了解的 Electron 概念

| 概念 | 与包的关系 |
|------|------------|
| **上下文隔离** | 较新版本中包不能直接访问 Node.js/Electron API |
| **Remote 模块** | `require('electron').remote`（已弃用） |
| **Menu API** | 自定义上下文菜单 |
| **Dialog API** | 文件打开/保存对话框 |
| **Clipboard API** | 系统剪贴板读写 |
| **Shell API** | 打开外部 URL、文件 |

### Atom 全局对象

```js
atom.appVersion           // 当前版本
atom.getLoadSettings()    // 启动配置
atom.getWindowLoadTimes() // 性能计时数据
atom.config               // 全局配置系统
atom.keymaps              // 键绑定系统
atom.commands             // 命令系统
atom.packages             // 包管理器
atom.themes               // 主题管理器
atom.grammars             // 语法注册表
atom.project              // 项目/文件系统管理
atom.notifications        // 通知系统
atom.clipboard            // 剪贴板
atom.workspace            // 工作区（窗格、编辑器、面板）
```

---

## 9. 从 Atom 迁移到 Pulsar

### 关键差异

| 领域 | Atom | Pulsar |
|------|------|--------|
| 治理 | GitHub/Microsoft | 社区开放治理 |
| Electron | 冻结在旧版本 | 升级到 v25+ |
| 遥测 | GitHub 遥测 | **已移除** |
| 包管理器 | `apm` | `ppm` |
| 配置目录 | `~/.atom` | `~/.pulsar` |

### 迁移步骤

```bash
# 1. 备份
cp -r ~/.atom ~/.atom-backup

# 2. 复制配置
cp ~/.atom/config.cson ~/.pulsar/
cp ~/.atom/keymap.cson ~/.pulsar/
cp ~/.atom/styles.less ~/.pulsar/
cp ~/.atom/snippets.cson ~/.pulsar/

# 3. 重新安装包
ppm install <package-name>
```

### 兼容性问题

1. **Electron API 变更** — 新版 Electron 移除了弃用 API（如 `remote` 模块）
2. **Chromium 版本升级** — 影响 CSS/JS 行为
3. **Node.js 版本变更** — 渲染进程中的 Node.js API 变化
4. **原生模块重编译** — tree-sitter 语法等原生插件需重编译

### 兼容性策略

```js
const electronVersion = process.versions.electron;
const majorVersion = parseInt(electronVersion.split('.')[0], 10);

if (majorVersion >= 25) {
  useNewAPI();   // Pulsar / 现代 Electron
} else {
  useLegacyAPI(); // 旧版 Atom
}
```

---

## 10. 构建语言语法与主题

### 语言语法（TextMate 格式）

包名约定：`language-<name>`。语法文件 `grammars/mylang.cson`：

```cson
'scopeName': 'source.mylang'
'name': 'MyLang'
'fileTypes': ['mylang', 'ml']
'patterns': [
  {
    'match': '\\b(function|return|if|else)\\b'
    'name': 'keyword.control.mylang'
  }
  {
    'match': '"([^"\\\\]|\\\\.)*"'
    'name': 'string.quoted.double.mylang'
  }
  {
    'match': '\\b\\d+\\.?\\d*\\b'
    'name': 'constant.numeric.mylang'
  }
  {
    'match': '//.*$'
    'name': 'comment.line.double-slash.mylang'
  }
  {
    'begin': '/\\*'
    'end': '\\*/'
    'name': 'comment.block.mylang'
  }
  {
    'begin': '\\{'
    'end': '\\}'
    'name': 'meta.block.mylang'
    'patterns': [{ 'include': '$self' }]
  }
]
```

`package.json`：

```json
{
  "name": "language-mylang",
  "grammars": [
    { "scopeName": "source.mylang", "path": "./grammars/mylang.cson" }
  ]
}
```

### 语法主题

`styles/colors.less`：

```less
@syntax-bg: #1e1e2e;
@syntax-fg: #cdd6f4;
@syntax-accent: #89b4fa;
@syntax-red: #f38ba8;
@syntax-green: #a6e3a1;
@syntax-yellow: #f9e2af;
@syntax-blue: #89b4fa;
@syntax-purple: #cba6f7;
@syntax-cyan: #94e2d5;
```

`styles/syntax.less`：

```less
@import "colors";

.syntax--comment { color: @syntax-accent; font-style: italic; }
.syntax--keyword { color: @syntax-purple; }
.syntax--string { color: @syntax-green; }
.syntax--constant.numeric { color: @syntax-red; }
.syntax--entity.name.function { color: @syntax-blue; }
.syntax--entity.name.type, .syntax--support.type { color: @syntax-yellow; }
```

### UI 主题

关键选择器：

```less
.tab-bar { /* ... */ }
.tab.active { /* ... */ }
.tree-view { /* ... */ }
.status-bar { /* ... */ }
atom-text-editor { /* ... */ }
.settings-view { /* ... */ }
```

### 作用域命名约定

```
<scope.subscope.subsubscope>

主类别: comment, constant, entity, invalid, keyword,
        markup, meta, storage, string, support, variable
子类别: comment.block, comment.line
        constant.numeric, constant.character
        entity.name, entity.other
        keyword.control, keyword.operator
        string.quoted, string.unquoted
        variable.language, variable.parameter
```

---

## 参考链接

| 资源 | URL |
|------|-----|
| Atom Flight Manual - Package Services | https://flight-manual.atom.io/hacking-atom/sections/package-services/ |
| tree-sitter | https://github.com/tree-sitter/tree-sitter |
| autocomplete-plus Provider API Wiki | https://github.com/atom/autocomplete-plus/wiki/Provider-API |
| Linter UI | https://github.com/steelbrain/linter |
| atom-linter 辅助库 | https://github.com/steelbrain/atom-linter |
| GitRepository API | https://flight-manual.atom-editor.cc/api/v1.63.1/GitRepository/ |
| atom/telemetry | https://github.com/atom/telemetry |
| Electron Performance | https://electronjs.org/docs/latest/tutorial/performance |
| Electron IPC | https://electronjs.org/docs/latest/tutorial/ipc |
| Pulsar 文档 | https://docs.pulsar-edit.dev |
| Pulsar Blog | https://blog.pulsar-edit.dev |
| TextMate Grammar Manual | https://manual.macromates.com/en/language_grammars |
| Creating a Theme | https://flight-manual.atom-editor.cc/hacking-atom/sections/creating-a-theme/ |
