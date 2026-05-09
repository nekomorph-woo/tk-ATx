# Atom 插件开发 - 完整参考手册

> 完整的 Atom/Pulsar 包开发 API 参考、架构概览和社区资源汇总。

---

## 目录

1. [架构概览](#1-架构概览)
2. [完整生命周期](#2-完整生命周期)
3. [API 参考](#3-api-参考)
4. [Keymap 系统详解](#4-keymap-系统详解)
5. [package.json 完整 Schema](#5-packagejson-完整-schema)
6. [样式系统](#6-样式系统)
7. [代码片段开发](#7-代码片段开发)
8. [语言语法开发](#8-语言语法开发)
9. [测试基础设施](#9-测试基础设施)
10. [CI/CD 集成](#10-cicd-集成)
11. [社区资源](#11-社区资源)
12. [Atom vs Pulsar 速查表](#12-atom-vs-pulsar-速查表)

---

## 1. 架构概览

### 整体架构

```
┌─────────────────────────────────────────────────┐
│                  Electron                       │
│  ┌───────────────┐  ┌────────────────────────┐  │
│  │  Main Process  │  │   Renderer Process    │  │
│  │  (Node.js)     │  │   (Chromium + Node.js) │  │
│  │                │  │                        │  │
│  │  - 窗口管理    │  │  ┌──────────────────┐  │  │
│  │  - 菜单        │  │  │   Atom Core      │  │  │
│  │  - 对话框      │  │  │   - Workspace    │  │  │
│  │  - 文件系统    │  │  │   - TextEditor   │  │  │
│  │                │  │  │   - Commands     │  │  │
│  │                │  │  │   - Config       │  │  │
│  │                │  │  │   - Packages     │  │  │
│  │                │  │  │   - Keymaps      │  │  │
│  │                │  │  │   - Themes       │  │  │
│  │                │  │  │   - Grammars     │  │  │
│  └───────┬───────┘  │  └──────┬───────────┘  │  │
│          │ IPC      │         │              │  │
│          └──────────┘         │              │  │
│                               │  ┌────────┐  │  │
│                               │  │Packages│  │  │
│                               │  │(你的)  │  │  │
│                               │  └────────┘  │  │
│                               └────────────────┘  │
└─────────────────────────────────────────────────┘
```

### 包类型

| 类型 | package.json 标记 | 说明 |
|------|-------------------|------|
| **Package（插件）** | 默认 | 提供功能逻辑 |
| **UI Theme** | `"theme": "ui"` | 编辑器 UI 外观 |
| **Syntax Theme** | `"theme": "syntax"` | 代码高亮配色 |
| **Language** | 无（命名约定 `language-*`） | 语言语法 + 片段 |

### 包之间的关系

```
Packages ──提供/消费──→ Services（服务系统）
  │
  ├── 使用 → Commands（命令系统）
  ├── 使用 → Keymaps（键绑定系统）
  ├── 使用 → Config（配置系统）
  ├── 使用 → Menus（菜单系统）
  ├── 使用 → Grammars（语法注册表）
  ├── 使用 → Views（视图层）
  └── 使用 → Notifications（通知系统）
```

---

## 2. 完整生命周期

### 加载与激活流程

```
Atom 启动
  ↓
读取所有包的 package.json
  ↓
检查 activationCommands / activationHooks
  ↓
[如果有 activationCommands 且未触发] → 包不加载（惰性）
  ↓
[触发条件满足] → 调用 activate(state)
  ↓
[其他包提供的服务可用时] → 调用 consumeXxx(service)
  ↓
[Atom 关闭] → 调用 serialize() → 调用 deactivate()
```

### 生命周期方法完整签名

```js
module.exports = {
  /**
   * 包激活时调用
   * @param {Object} state - 上次 serialize() 返回的状态
   */
  activate(state) { /* ... */ },

  /**
   * 包停用时调用
   * 释放所有资源：事件监听、定时器、DOM 元素等
   */
  deactivate() { /* ... */ },

  /**
   * Atom 关闭前调用
   * @returns {Object} JSON 可序列化的状态对象
   */
  serialize() { return {}; },

  /**
   * 消费其他包提供的服务（可选）
   */
  consumeStatusBar(statusBar) { /* ... */ },

  /**
   * 提供服务给其他包（可选）
   * @returns {Object|Function} 服务对象或工厂函数
   */
  provideMyService() { return { /* ... */ }; }
};
```

### 包管理器 API

```js
// 激活包
atom.packages.activatePackage('my-package');
atom.packages.activatePackageSoon('my-package'); // 惰性激活

// 查询
atom.packages.isPackageActive('my-package');
atom.packages.getLoadedPackages();
atom.packages.getActivePackages();

// 包路径
atom.packages.getPackageDirPaths();  // 包搜索路径
atom.packages.resolvePackagePath('my-package');

// 事件
atom.packages.onDidActivatePackage(({package: pkg}) => { /* ... */ });
atom.packages.onDidDeactivatePackage(({package: pkg}) => { /* ... */ });
```

---

## 3. API 参考

### 3.1 Workspace API

```js
// === 文件操作 ===
atom.workspace.open(path, options?)     // Promise<TextEditor>
atom.workspace.open()                   // 新建空编辑器

// === 活动项 ===
atom.workspace.getActiveTextEditor()    // TextEditor | undefined
atom.workspace.getActivePane()          // Pane
atom.workspace.getActivePaneItem()      // any
atom.workspace.getPanes()               // Pane[]

// === 面板 ===
atom.workspace.addModalPanel({ item, visible })
atom.workspace.addBottomPanel({ item, priority })
atom.workspace.addLeftPanel({ item, priority })
atom.workspace.addRightPanel({ item, priority })
atom.workspace.addTopPanel({ item, priority })

// === 钉 ===
atom.workspace.addOpener(opener)        // opener(uri) => Promise<PaneItem>
atom.workspace.getOpeners()

// === 事件 ===
atom.workspace.onDidOpen(({uri, item, pane}) => {})
atom.workspace.onDidClose(({uri, item, pane}) => {})
atom.workspace.onDidDestroyPane(pane => {})
atom.workspace.onDidChangeActivePaneItem(item => {})
atom.workspace.onDidChangeActiveTextEditor(editor => {})
atom.workspace.onDidAddTextEditor(({textEditor, pane}) => {})
atom.workspace.onDidStopChangingActivePaneItem(item => {})
```

### 3.2 TextEditor API

```js
const editor = atom.workspace.getActiveTextEditor();

// === 文本操作 ===
editor.getText()
editor.setText(text)
editor.insertText(text, options?)
editor.insertNewline()
editor.insertNewlineBelow()
editor.delete()
editor.backspace()
editor.deleteToBeginningOfWord()
editor.deleteToEndOfWord()
editor.deleteLine()
editor.cutSelectedText()
editor.copySelectedText()
editor.pasteText()
editor.selectAll()
editor.selectToFirstCharacterOfLine()
editor.selectToBeginningOfNextParagraph()

// === 光标 ===
editor.getCursorBufferPositions()       // [{row, column}]
editor.setCursorBufferPosition([row, col])
editor.moveCursorToBeginningOfNextParagraph()
editor.moveCursorToScreenPosition({row, column})
editor.getLastCursor()

// === 选择 ===
editor.getSelectedText()
editor.getSelectedBufferRanges()
editor.setSelectedBufferRange([[r1,c1],[r2,c2]], options?)
editor.addSelectionForBufferRange(range)
editor.clearSelections()
editor.getSelections()

// === 文本操作 ===
editor.getLineCount()
editor.buffer.getLineForRow(row)
editor.buffer.getRange()
editor.buffer.getRows()
editor.buffer.getTextInRange(range)
editor.buffer.setTextInRange(range, text)
editor.buffer.insertText(text, position)

// === 标记与装饰 ===
editor.markBufferRange(range, options?)  // Marker
editor.markBufferPosition(position)
editor.decorateMarker(marker, decoration)
editor.getMarkers()

// === 文件信息 ===
editor.getPath()
editor.getTitle()
editor.getLongTitle()
editor.isModified()
editor.save()
editor.saveAs(filePath)

// === 行 ===
editor.lineTextForBufferRow(row)
editor.indentationForBufferRow(row)
editor.setIndentationForBufferRow(row, level)
editor.autoIndentBufferRow(row)
editor.toggleLineCommentInSelection()

// === 折叠 ===
editor.toggleFoldAtBufferRow(row)
editor.foldAll()
editor.unfoldAll()
editor.isFoldedAtBufferRow(row)
editor.foldBufferRow(row)

// === 光标信息 ===
editor.getScrollTop()
editor.getScrollLeft()
editor.scrollToCursorPosition()
editor.scrollToBufferPosition(position)

// === 语法 ===
editor.getGrammar()
editor.setGrammar(grammar)
editor.getScopeDescriptor()

// === 模式 ===
editor.isReadOnly()
editor.getSoftWrap()
editor.setSoftWrap(wrap)
editor.getTabLength()
editor.setTabLength(length)
editor.getAutoIndent()
editor.setAutoIndent(autoIndent)
```

### 3.3 TextBuffer API

```js
const buffer = editor.getBuffer();

// === 基础操作 ===
buffer.getText()
buffer.setText(text)
buffer.getTextInRange(range)
buffer.setTextInRange(range, text)
buffer.insert(text, position)
buffer.delete(range)

// === 行 ===
buffer.getLineCount()
buffer.getLine(row)
buffer.lines                             // string[]

// === 变更事件 ===
buffer.onDidChange(({changes}) => {
  // changes: [{oldRange, newRange, oldText, newText}, ...]
})
buffer.onDidChangeText(event => {})
buffer.onDidStopChanging(() => {})
buffer.onDidDestroy(() => {})

// === 历史 ===
buffer.undo()
buffer.redo()
buffer.canUndo()
buffer.canRedo()
buffer.clearUndoStack()
buffer.createCheckpoint()
buffer.revertToCheckpoint(checkpoint)
buffer.groupChangesSinceCheckpoint(checkpoint)
buffer.groupsSinceCheckpoint(checkpoint)

// === 标记 ===
buffer.markRange(range, options?)
buffer.markPosition(position, options?)
buffer.getMarkers()
buffer.destroy()

// === 批量操作 ===
buffer.transact(() => {
  // 多个操作在同一个 undo group 中
  buffer.insert(text1, pos1);
  buffer.insert(text2, pos2);
})

// === 文件 ===
buffer.getPath()
buffer.getUri()
buffer.setPath(path)
buffer.save()
buffer.isModified()
```

### 3.4 Pane API

```js
const pane = atom.workspace.getActivePane();

pane.getItems()                          // PaneItem[]
pane.getActiveItem()
pane.activateItem(item)
pane.activateNextItem()
pane.activatePreviousItem()
pane.destroyItem(item)
pane.moveItemToPane(item, pane)
pane.splitRight(options?)
pane.splitLeft(options?)
pane.splitDown(options?)
pane.splitUp(options?)
pane.close()
pane.focus()
pane.isActive()
```

### 3.5 Panel API

```js
const panel = atom.workspace.addBottomPanel({ item, priority, visible });

panel.getItem()
panel.isVisible()
panel.show()
panel.hide()
panel.destroy()
```

### 3.6 CommandRegistry API

```js
// 注册命令
const disposable = atom.commands.add('atom-workspace', {
  'my-package:toggle': (event) => { /* ... */ },
  'my-package:do-something': ({target}) => { /* ... */ }
});

// 调度命令
atom.commands.dispatch(targetElement, 'my-package:toggle');

// 查询
atom.commands.findCommands({target: targetElement});
```

### 3.7 ConfigManager API

```js
// 读写
atom.config.get('core.theme')
atom.config.set('core.theme', 'one-dark-ui')
atom.config.unset('my-package.setting')

// 监听
const disposable = atom.config.observe('my-package.setting', value => {});
atom.config.onDidChange('my-package.setting', ({newValue, oldValue}) => {});
atom.config.onDidChange(() => {});

// Schema
atom.config.setSchema('my-package', {
  type: 'object',
  properties: {
    mySetting: { type: 'boolean', default: true }
  }
});
```

### 3.8 NotificationManager API

```js
// 添加通知
const notification = atom.notifications.addInfo('message', { dismissable: true });
const notification = atom.notifications.addSuccess('message');
const notification = atom.notifications.addWarning('message');
const notification = atom.notifications.addError('message', {
  detail: 'stack trace',
  dismissable: true,
  buttons: [{ text: 'Fix', onDidClick: () => {} }]
});

// 全局操作
atom.notifications.clear()

// 事件
atom.notifications.onDidAddNotification(notification => {});
```

### 3.9 GrammarRegistry API

```js
// 查询
atom.grammars.getGrammars()
atom.grammars.grammarForScopeName('source.js')
atom.grammars.selectGrammar(filePath, contents)

// 注册/移除
atom.grammars.addGrammar(grammar)
atom.grammars.removeGrammar(grammar)

// 事件
atom.grammars.onDidAddGrammar(grammar => {})
atom.grammars.onDidUpdateGrammar(grammar => {})
```

### 3.10 TooltipManager API

```js
const disposable = atom.tooltips.add(element, {
  title: 'Title',
  text: 'Body text',
  html: '<b>HTML content</b>',
  placement: 'top',          // 'top', 'bottom', 'left', 'right', 'auto'
  trigger: 'hover',          // 'hover', 'focus', 'click'
  delay: { show: 300, hide: 100 },
  keyBindingCommand: 'my-package:cmd',
  class: 'my-tooltip',
  itemType: 'tooltip'        // 'tooltip' | 'command' | 'key'
});
```

### 3.11 GitRepository API

```js
const repo = atom.project.getRepositories()[0];

if (repo) {
  repo.isPathIgnored(filePath)
  repo.getWorkingDirectory()
  repo.relativize(filePath)
  repo.getShortHead()                     // 当前分支名
  repo.isLoading()                        // 是否正在加载
  repo.isDestroyed()                      // 是否已销毁
  repo.getStatus()                        // Promise<Array>
  repo.getReferences()                    // Promise<Array>
  repo.getBranches()                      // Promise<Array>
  repo.getAheadBehindCount(branch, ref)   // Promise<{ahead, behind}>
  repo.getDiffStats()                     // Promise
  repo.refresh()                          // 刷新状态
  repo.destroy()

  // 事件
  repo.onDidChangeStatus(event => {})
  repo.onDidChangeBranch(branchName => {})
  repo.onDidDestroy(() => {})
}
```

### 3.12 Project API

```js
// 路径
atom.project.getPaths()                   // string[]
atom.project.addPath(path)                // Promise
atom.project.removePath(path)

// 仓库
atom.project.getRepositories()            // GitRepository[]

// 事件
atom.project.onDidChangePaths(paths => {})
atom.project.onDidAddProject(({project}) => {})
```

### 3.13 Clipboard API

```js
atom.clipboard.write(text)
atom.clipboard.read()
atom.clipboard.writeWithMetadata(text, metadata)
```

### 3.14 ViewRegistry API

```js
// 为模型对象注册视图
atom.views.addViewProvider(MyModel, (model) => {
  const element = document.createElement('div');
  // ... 构建 DOM
  return element;
});

// 获取模型对象的视图
const element = atom.views.getView(model);
```

---

## 4. Keymap 系统详解

### 选择器优先级

键绑定使用 CSS 选择器语法，与 CSS 相同的特异性规则：

```json
{
  ".platform-darwin atom-workspace": {
    "cmd-shift-p": "command-palette:toggle"
  },
  "atom-text-editor": {
    "ctrl-c": "core:copy"
  },
  "atom-text-editor:not([mini])": {
    "cmd-d": "editor:duplicate-lines"
  },
  ".platform-linux atom-text-editor": {
    "ctrl-shift-m": "my-package:transform"
  },
  "atom-text-editor[data-grammar='source js']:not([mini])": {
    "cmd-enter": "my-package:run-js"
  }
}
```

### 常用选择器

| 选择器 | 含义 |
|--------|------|
| `atom-workspace` | 全局作用域 |
| `atom-text-editor` | 所有文本编辑器 |
| `atom-text-editor:not([mini])` | 排除 mini 编辑器 |
| `.platform-darwin` | 仅 macOS |
| `.platform-win32` | 仅 Windows |
| `.platform-linux` | 仅 Linux |
| `atom-pane.active` | 活动窗格 |

### 调试工具

- **Keybinding Resolver**：`Cmd+Shift+.` — 显示所有匹配的键绑定
- **KeybindingLogger**：在开发者控制台输入 `atom.keymaps.onDidMatchBinding(event => console.log(event))`

### 按键语法

| 修饰键 | 格式 |
|--------|------|
| Ctrl | `ctrl` |
| Alt | `alt` |
| Shift | `shift` |
| Cmd (macOS) | `cmd` |

示例：`ctrl-alt-cmd-shift-x`

---

## 5. package.json 完整 Schema

### Atom 专有字段

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "description": "Description",
  "main": "./lib/my-package",
  "repository": "https://github.com/user/my-package",
  "license": "MIT",

  // === Atom 特有 ===

  "activationCommands": {
    "atom-workspace": ["my-package:toggle"]
  },

  "activationHooks": [
    "language-javascript:grammar-used",
    "core:loaded-shell-environment"
  ],

  "activationEvents": ["my-package:toggle"],

  "configSchema": {
    "type": "object",
    "properties": {
      "mySetting": {
        "type": "boolean",
        "default": true,
        "title": "My Setting",
        "description": "Description of the setting",
        "order": 1
      }
    }
  },

  "consumedServices": {
    "service-name": {
      "versions": {
        "^1.0.0": "consumeMethodName"
      }
    }
  },

  "providedServices": {
    "service-name": {
      "description": "Description",
      "versions": {
        "1.0.0": "provideMethodName"
      }
    }
  },

  "styles": ["styles/my-package.less"],
  "keymaps": ["keymaps/my-package.json"],
  "menus": ["menus/my-package.json"],
  "snippets": ["snippets/language.cson"],
  "grammars": ["grammars/language.cson"],

  "engines": {
    "atom": ">=1.0.0 <2.0.0"
  },

  "atomTestRunner": "./test/runner",
  "theme": "ui",

  "workspaceOpeners": [".myext"],

  "_atomMenu": { /* 应用菜单定义 */ },

  "_contextMenu": { /* 上下文菜单定义 */ }
}
```

---

## 6. 样式系统

### Less 支持

Atom/Pulsar 推荐使用 Less，自动编译。支持 Less 的所有特性：变量、混入、嵌套、函数等。

### 主题变量

在 `styles/my-package.less` 中导入 UI 变量：

```less
@import "ui-variables";
@import "syntax-variables";

.my-package {
  color: @text-color;
  background-color: @app-background-color;
  border: 1px solid @base-border-color;
  font-size: @font-size-base;
}
```

### 常用 UI 变量

| 变量 | 用途 |
|------|------|
| `@text-color` | 默认文本颜色 |
| `@text-color-subtle` | 次要文本 |
| `@text-color-highlight` | 高亮文本 |
| `@background-color` | 默认背景 |
| `@app-background-color` | 应用背景 |
| `@base-border-color` | 边框颜色 |
| `@pane-item-background-color` | 面板项背景 |
| `@pane-item-border-color` | 面板项边框 |
| `@input-background-color` | 输入框背景 |
| `@button-background-color` | 按钮背景 |
| `@tooltip-background-color` | 工具提示背景 |
| `@tooltip-text-color` | 工具提示文本 |
| `@font-size-base` | 基础字号 |

### Styleguide

通过命令面板运行 `Styleguide`（`Cmd+Ctrl+Shift+G`）查看所有可用 UI 组件和变量。

---

## 7. 代码片段开发

### CSON 格式

文件路径：`snippets/language.cson`

```cson
'.source.js':
  'Console log':
    'prefix': 'cl'
    'body': 'console.log($1);$0'

  'Arrow function':
    'prefix': 'af'
    'body': 'const ${1:name} = (${2:args}) => {\n\t$0\n}'

  'Promise':
    'prefix': 'prom'
    'body': 'new Promise((resolve, reject) => {\n\t$0\n})'

  'Try-catch':
    'prefix': 'tc'
    'body': 'try {\n\t$1\n} catch (error) {\n\tconsole.error(error);\n\t$0\n}'

  'Import default':
    'prefix': 'im'
    'body': 'import ${1:module} from \'${2:module-path}\';$0'
```

### Tab-stop 语法

| 语法 | 含义 |
|------|------|
| `$1`, `$2`... | Tab-stop 位置，按数字顺序跳转 |
| `$0` | 最终光标位置 |
| `${1:default}` | 带默认值的 Tab-stop |
| `${1/(.+)/\\u$1/}` | 变换（首字母大写） |
| `$CLIPBOARD` | 插入剪贴板内容 |

---

## 8. 语言语法开发

### TextMate Grammar（CSON 格式）

```cson
'scopeName': 'source.mylang'
'name': 'MyLang'
'fileTypes': ['mylang', 'ml']
'firstLineMatch': '^#!.*\\bmylang\\b'
'foldingStartMarker': '\\{\\s*$'
'foldingStopMarker': '^\\s*\\}'
'patterns': [
  {
    'comment': 'Keywords'
    'match': '\\b(function|return|if|else|for|while|class|import|export)\\b'
    'name': 'keyword.control.mylang'
  }
  {
    'comment': 'Strings'
    'match': '"([^"\\\\]|\\\\.)*"'
    'name': 'string.quoted.double.mylang'
  }
  {
    'comment': 'Single-quoted strings'
    'match': "'([^'\\\\]|\\\\.)*'"
    'name': 'string.quoted.single.mylang'
  }
  {
    'comment': 'Numbers'
    'match': '\\b(0x[0-9a-fA-F]+|\\d+\\.?\\d*)\\b'
    'name': 'constant.numeric.mylang'
  }
  {
    'comment': 'Line comments'
    'match': '//.*$'
    'name': 'comment.line.double-slash.mylang'
  }
  {
    'comment': 'Block comments'
    'begin': '/\\*'
    'end': '\\*/'
    'name': 'comment.block.mylang'
    'patterns': [
      { 'include': '$self' }
    ]
  }
  {
    'comment': 'Type names'
    'match': '\\b([A-Z][a-zA-Z0-9_]*)\\b'
    'name': 'entity.name.type.mylang'
  }
  {
    'comment': 'Function names'
    'match': '\\b([a-z][a-zA-Z0-9_]*)\\s*(?=\\()'
    'name': 'entity.name.function.mylang'
  }
  {
    'comment': 'Embed HTML'
    'include': 'text.html.basic'
  }
  {
    'comment': 'Nested blocks'
    'begin': '\\{'
    'end': '\\}'
    'name': 'meta.block.mylang'
    'patterns': [
      { 'include': '$self' }
    ]
  }
]
```

### 注入语法

```cson
# 注入到 HTML 中的脚本块
'scopeName': 'source.js.embedded.html'
'InjectionSelector': 'text.html meta.tag.script.html'
'patterns': [
  { 'include': 'source.js' }
]
```

---

## 9. 测试基础设施

### Jasmine 1.3（默认）

```js
// 基本结构
describe('MyPackage', () => {
  beforeEach(() => { /* ... */ });
  afterEach(() => { /* ... */ });

  it('should work', () => {
    expect(value).toBe(expected);
  });
});

// 异步测试
describe('async tests', () => {
  it('waits for promise', () => {
    waitsForPromise(() => {
      return asyncOperation().then(result => {
        expect(result).toBe('expected');
      });
    });
  });

  it('waits for condition', () => {
    let done = false;
    asyncOperation().then(() => { done = true; });
    waitsFor(() => done, 'Async op should complete', 5000);
    runs(() => { expect(done).toBe(true); });
  });
});
```

### Atom 测试辅助

```js
// 创建测试环境
const workspaceElement = atom.views.getView(atom.workspace);
jasmine.attachToDOM(workspaceElement);

// 激活包
const activationPromise = atom.packages.activatePackage('my-package');

// 设置编辑器
waitsForPromise(() => atom.workspace.open('test.js'));
let editor;
runs(() => { editor = atom.workspace.getActiveTextEditor(); });

// 自定义匹配器
expect(element).toExist();
expect(element).toBeVisible();
expect(element).toHaveFocus();
expect(element).toHaveClass('my-class');
expect(element).toHaveAttr('data-value');
expect(element).toHaveCss({ display: 'block' });
expect(element).toBeHidden();
expect(path).toExistOnDisk();

// 临时文件
const tempPath = atom.project.getPaths()[0] + '/temp-test.txt';
fs.writeFileSync(tempPath, 'test content');

// 模拟用户输入
atom.commands.dispatch(workspaceElement, 'my-package:toggle');
```

### Jasmine 3.x 升级

```bash
npm install --save-dev atom-jasmine3-test-runner
```

```json
{ "atomTestRunner": "./node_modules/atom-jasmine3-test-runner" }
```

### 命令行测试

```bash
# 测试整个包
atom --test path/to/package

# 测试特定文件
atom --test path/to/package/spec/my-spec.js

# 超时设置
atom --test --timeout 60 path/to/package/spec

# 指定应用
ATOM_PATH=/path/to/Atom.app atom --test spec/
```

---

## 10. CI/CD 集成

### GitHub Actions

```yaml
name: Test Atom Package
on: [push, pull_request]
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '16'
      - run: npm install
      - run: npm test

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '16'
      - run: npm install
      - run: npm run lint

  publish:
    needs: [test, lint]
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      with:
        node-version: '16'
      - run: npm install
      - run: npm run lint
      - run: echo "$PPM_TOKEN" | ppm login --token /dev/stdin
      - run: ppm publish
    env:
      PPM_TOKEN: ${{ secrets.PPM_TOKEN }}
```

---

## 11. 社区资源

### 官方/权威文档

| 资源 | URL | 状态 |
|------|-----|------|
| Atom Flight Manual（社区镜像） | https://flight-manual.atom-editor.cc/ | 维护中 |
| Atom API Docs v1.63.1 | https://flight-manual.atom-editor.cc/api/v1.63.1/ | 归档 |
| Pulsar 文档 | https://docs.pulsar-edit.dev/ | 活跃 |
| Pulsar API Reference | https://docs.pulsar-edit.dev/api/pulsar/v1.0.0/ | 活跃 |
| Pulsar 包注册中心 | https://web.pulsar-edit.dev | 活跃 |

### 教程与指南

| 教程 | URL |
|------|-----|
| Package: Word Count（官方教程） | https://flight-manual.atom-editor.cc/hacking-atom/sections/package-word-count/ |
| Writing Specs | https://flight-manual.atom-editor.cc/hacking-atom/sections/writing-specs/ |
| Creating a TextMate Grammar | https://flight-manual.atom-editor.cc/hacking-atom/sections/creating-a-legacy-textmate-grammar/ |
| Creating a Theme | https://flight-manual.atom-editor.cc/hacking-atom/sections/creating-a-theme/ |
| Package Services | https://flight-manual.atom-editor.cc/hacking-atom/sections/package-services/ |
| Converting from CoffeeScript | https://flight-manual.atom-editor.cc/hacking-atom/sections/converting-from-coffeescript/ |
| TextMate Grammar Manual | https://manual.macromates.com/en/language_grammars |

### GitHub 仓库

| 仓库 | 说明 |
|------|------|
| [pulsar-edit/Pulsar](https://github.com/pulsar-edit/Pulsar) | Pulsar 编辑器源码 |
| [atom/atom](https://github.com/atom/atom) | Atom 编辑器源码（归档） |
| [atom/atom-keymap](https://github.com/atom/atom-keymap) | 键绑定系统 |
| [atom/autocomplete-plus](https://github.com/atom/autocomplete-plus) | 自动补全引擎 |
| [atom/teletype](https://github.com/atom/teletype) | P2P 协作编辑 |
| [atom/atom-ide-ui](https://github.com/atom/atom-ide-ui) | IDE UI 组件 |
| [atom/atom-languageclient](https://github.com/atom/atom-languageclient) | LSP 客户端库 |
| [atom/atom-space-pen-views](https://github.com/atom/atom-space-pen-views) | View 框架 |
| [atom/atom-select-list](https://github.com/atom/atom-select-list) | 选择列表组件 |
| [steelbrain/linter](https://github.com/steelbrain/linter) | Linter 框架 |
| [steelbrain/atom-linter](https://github.com/steelbrain/atom-linter) | Linter 辅助库 |
| [atom-minimap/minimap](https://github.com/atom-minimap/minimap) | Minimap |
| [tree-sitter/tree-sitter](https://github.com/tree-sitter/tree-sitter) | Tree-sitter 解析器 |
| [UziTech/atom-jasmine3-test-runner](https://github.com/UziTech/atom-jasmine3-test-runner) | Jasmine 3 测试运行器 |
| [semantic-release/apm](https://github.com/semantic-release/apm) | 自动化发布工具 |

### 策展列表

| 资源 | URL |
|------|-----|
| awesome-atom | https://github.com/mehcode/awesome-atom |
| AtomPackages | https://github.com/stevelinus/AtomPackages |

---

## 12. Atom vs Pulsar 速查表

| 概念 | Atom | Pulsar |
|------|------|--------|
| 包管理 CLI | `apm` | `ppm` |
| 配置目录 | `~/.atom` | `~/.pulsar` |
| 包注册中心 | atom.io（已关闭） | web.pulsar-edit.dev |
| 发布命令 | `apm publish` | `ppm publish` |
| 测试命令 | `atom --test` | `pulsar --test` |
| 开发模式 | `atom --dev` | `pulsar --dev` |
| 遥测 | GitHub 遥测 | 已移除 |
| Electron 版本 | 冻结（旧） | 活跃更新（v25+） |
| 核心维护 | GitHub/Microsoft | 社区志愿者 |
| 状态 | 已归档 (2022-12-15) | 活跃开发 |
