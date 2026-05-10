# md-wysiwyg 技术设计文档

## 1. 架构总览

```
┌─────────────────────────────────────────────────────────────┐
│                       Pulsar Workspace                       │
│                                                             │
│  ┌──────────────────┐     ┌──────────────────────────────┐  │
│  │ MdWysiwygPackage │────▶│ MdWysiwygEditor (View)       │  │
│  │                  │     │                              │  │
│  │ - activate       │     │  ┌────────────────────────┐  │  │
│  │ - deactivate     │     │  │ Milkdown Editor        │  │  │
│  │ - toggle         │     │  │  ├─ commonmark         │  │  │
│  │ - _switchToXxx   │     │  │  ├─ gfm                │  │  │
│  │ - _initTheme     │     │  │  ├─ listener           │  │  │
│  └──────────────────┘     │  │  ├─ history / cursor   │  │  │
│                           │  │  ├─ clipboard / indent  │  │  │
│                           │  │  ├─ trailing / upload   │  │  │
│                           │  │  ├─ mathPlugin (custom) │  │  │
│                           │  │  ├─ codeBlockView       │  │  │
│                           │  │  └─ sourceExpansion     │  │  │
│                           │  └────────────────────────┘  │  │
│                           └──────────────────────────────┘  │
│                                                             │
│  切换逻辑:                                                   │
│    TextEditor ──Alt+M──▶ 直接 new MdWysiwygEditor           │
│    MdWysiwygEditor ──Alt+M──▶ workspace.open(filePath)      │
│                                + setText + destroy wysiwyg  │
└─────────────────────────────────────────────────────────────┘
```

用户在源码 TextEditor 中按 `Alt+M`，插件直接实例化 `MdWysiwygEditor` 并通过 `pane.activateItem()` 替换当前视图。在 WYSIWYG 视图中按 `Alt+M`，序列化 Milkdown 内容为 Markdown，打开源码编辑器并销毁 WYSIWYG 视图。

**关键设计**：`_switchToWysiwyg` 不使用 `workspace.open(uri)` 而是直接 `new MdWysiwygEditor()`，避免 Pulsar URI 缓存导致销毁后的 editor 阻止新实例创建。

## 2. 技术栈

### 2.1 运行时环境

| 环境 | 版本 |
|------|------|
| Pulsar | Electron 25+ |
| Node.js | 随 Electron 内置（18.x+） |
| esbuild | ^0.28.0（devDependencies） |

### 2.2 核心编辑引擎

| 包 | 版本 | 许可证 | 用途 |
|----|------|--------|------|
| `@milkdown/kit` | `^7.20.0` | MIT | 编辑器核心全量包 |
| `highlight.js` | `^11.9.0` | MIT | 代码语法高亮 |

`@milkdown/kit` 实际使用的子包：

| 子包路径 | 用途 |
|----------|------|
| `@milkdown/kit/core` | Editor 类、rootCtx、defaultValueCtx |
| `@milkdown/kit/prose` | ProseMirror 状态/视图/插件/Schema |
| `@milkdown/kit/preset/commonmark` | CommonMark 标准语法 |
| `@milkdown/kit/preset/gfm` | GFM 扩展（表格、任务列表、删除线、自动链接） |
| `@milkdown/kit/plugin/listener` | 内容变更事件监听 |
| `@milkdown/kit/plugin/upload` | 图片拖拽/粘贴上传接口 |
| `@milkdown/kit/plugin/clipboard` | 剪贴板处理 |
| `@milkdown/kit/plugin/history` | 撤销/重做 |
| `@milkdown/kit/plugin/indent` | 缩进规则 |
| `@milkdown/kit/plugin/cursor` | 光标管理 |
| `@milkdown/kit/plugin/trailing` | 尾部段落自动补全 |
| `@milkdown/kit/utils` | getMarkdown、$node、$mark、$prose、$remark 等工具 |

### 2.3 数学公式

| 包 | 版本 | 许可证 | 用途 |
|----|------|--------|------|
| `katex` | `^0.16.45` | MIT | LaTeX 渲染引擎 |
| `remark-math` | `^6.0.0` | MIT | 解析 `$...$` 和 `$$...$$` 语法 |

自定义 Milkdown 插件（`milkdown-plugins/math.js`），包含：
- `mathRemark`：remark-math 集成
- `mathInlineNode` / `mathBlockNode`：ProseMirror Node 定义
- `mathInlineViewPlugin` / `mathBlockViewPlugin`：NodeView（KaTeX 渲染，选中时显示源码）

### 2.4 图表渲染

| 包 | 版本 | 许可证 | 用途 |
|----|------|--------|------|
| `mermaid` | `^11.14.0` | MIT | Mermaid 图表渲染 |

通过代码块 NodeView 的 `language === 'mermaid'` 判断（`milkdown-plugins/mermaid.js`）。选中时显示 `<pre>` 源码编辑区，失焦时通过 `mermaid.render()` 渲染 SVG，带防抖延迟。

### 2.5 源码展开

自定义 ProseMirror 事务插件（`milkdown-plugins/paragraph-info.js`）：
- 选中 strong / emphasis / inlineCode mark 时，展开为原始分隔符语法（`**...**`、`*...*`、`` `...` `` `）
- 光标移出范围后，折叠回渲染视图
- 通过 `addToHistory: false` 的事务实现，不产生撤销记录

### 2.6 View 修复

`milkdown-plugins/view.js` 导出 patched `$view` 函数。修复上游 Milkdown `$nodeSchema` 在 `SchemaReady` 之前复制 `nodeSchema.id` 导致 `type.id` 为 `undefined` 的 bug。

### 2.7 完整依赖清单

```json
{
  "dependencies": {
    "@milkdown/kit": "^7.20.0",
    "highlight.js": "^11.9.0",
    "katex": "^0.16.45",
    "mermaid": "^11.14.0",
    "remark-math": "^6.0.0"
  },
  "devDependencies": {
    "esbuild": "^0.28.0"
  }
}
```

## 3. 模块设计

### 3.1 MdWysiwygPackage（主模块）

```
lib/md-wysiwyg.js（module.exports 部分，约第 214 行起）
```

职责：包生命周期管理、命令注册、模式切换、主题适配。

| 方法 | 说明 |
|------|------|
| `activate(state)` | 注册 opener、toggle 命令、恢复已打开的 WYSIWYG 标签页、初始化主题适配 |
| `deactivate()` | 销毁所有订阅 |
| `serialize()` | 保存当前打开的 WYSIWYG URI 列表 |
| `toggle()` | 根据活跃 item 类型自动切换方向 |
| `_switchToWysiwyg(textEditor, pane)` | 直接 new MdWysiwygEditor + pane.activateItem + destroy textEditor |
| `_switchToSource(wysiwygEditor)` | 序列化内容 → workspace.open → setText → destroy wysiwyg |
| `_initThemeAdapter()` | 读取 core.themes 判断 dark/light，设置 data-theme 属性 |
| `_isDarkTheme()` | 检查 UI 主题名中是否包含 dark/night/monokai/one |

### 3.2 MdWysiwygEditor（核心视图）

```
lib/md-wysiwyg.js（class 部分，约第 62 行起）
```

职责：管理单个 Milkdown 编辑器实例的完整生命周期。

构造函数：

```
constructor(filePath)
  ├─ 创建 DOM: element (.md-wysiwyg-editor) > editorContainer (.milkdown-container)
  ├─ 读取 config: fontSize → editorContainer.style.fontSize
  ├─ 读取 config: editorMaxWidth → editorContainer.style.maxWidth
  ├─ 注册 config observers（fontSize, editorMaxWidth）
  └─ _init(filePath)
```

初始化：

```
_init(filePath)
  ├─ fs.readFileSync(filePath) → content
  ├─ loadMilkdown() → kit（首次加载时注入 KaTeX CSS + highlight.js CSS）
  ├─ kit.Editor.make()
  │   .config(ctx => {
  │     ctx.set(rootCtx, editorContainer)
  │     ctx.set(defaultValueCtx, content)
  │     ctx.get(listenerCtx).markdownUpdated(...)
  │     ctx.get(listenerCtx).destroy(...)
  │   })
  │   .use(commonmark, gfm, listener, history, cursor, clipboard, indent, trailing, upload)
  │   .use(mathPlugin, codeBlockViewPlugin, sourceExpansionPlugin)
  │   .create()
  └─ initialized = true
```

Pulsar PaneItem 接口：

| 方法 | 说明 |
|------|------|
| `getTitle()` | 返回文件名 |
| `getLongTitle()` | 文件名 + 修改状态 |
| `getURI()` | `md-wysiwyg://` + 文件路径 |
| `getPath()` | 文件路径 |
| `getElement()` | 根 DOM 元素 |
| `serialize()` | filePath + deserializer 标识 |
| `isModified()` | 是否有未保存修改 |
| `save(filePath)` | 序列化 Milkdown 内容写入文件 |
| `shouldPromptToSave()` | 修改状态下提示保存 |
| `copy()` | 创建同文件路径的新实例 |
| `destroy()` | 销毁 Milkdown、清理 DOM 和订阅 |

### 3.3 Milkdown 加载与 CSS 注入

```
lib/md-wysiwyg.js（顶部函数）
```

`loadMilkdown()` 为单例，首次调用时：
1. `require('./milkdown-bundle.cjs')` 加载打包后的 Milkdown
2. `injectKatexCSS()` — 读取 `katex.min.css`，重写字体路径为 `file://` 绝对路径，注入 `<style>` 标签
3. `injectHljsCSS()` — 通过 `<link>` 标签注入 `atom-one-dark.min.css`

### 3.4 自定义 Milkdown 插件

所有自定义插件位于 `milkdown-plugins/`，通过 `milkdown-entry.mjs` 汇总导出，esbuild 打包为 `lib/milkdown-bundle.cjs`。

#### math-plugin（`math.js`）

| 导出 | 说明 |
|------|------|
| `mathRemark` | remark-math 集成 |
| `mathInlineNode` | `math_inline` ProseMirror Node（inline, atom） |
| `mathBlockNode` | `math_block` ProseMirror Node（block, atom, code） |
| `mathInlineViewPlugin` | 行内公式 NodeView（KaTeX 渲染） |
| `mathBlockViewPlugin` | 块级公式 NodeView（KaTeX 渲染） |

NodeView 行为：
- 默认：`katex.renderToString()` 渲染 HTML（`throwOnError: true`，错误时显示原始 LaTeX）
- 选中（selectNode）：显示纯文本源码，加 `.math-selected` 类
- 取消选中（deselectNode）：重新渲染 KaTeX

#### codeBlockViewPlugin（`highlight.js`）

| 导出 | 说明 |
|------|------|
| `codeBlockViewPlugin` | 代码块 NodeView，使用 patched `$view` |

行为：
- `language === 'mermaid'`：委托给 `createMermaidView`
- 其他语言：highlight.js 语法高亮
- 支持 18 种语言 + 8 种别名映射（js→javascript, ts→typescript, py→python 等）

#### sourceExpansionPlugin（`paragraph-info.js`）

| 导出 | 说明 |
|------|------|
| `sourceExpansionPlugin` | ProseMirror Plugin（appendTransaction） |

行为：
- 光标进入 strong / emphasis / inlineCode mark 范围 → 展开为分隔符语法
- 光标移出 → 折叠回渲染视图
- 通过 plugin state 追踪展开状态和范围，事务映射自动调整位置

#### mermaid（`mermaid.js`）

| 导出 | 说明 |
|------|------|
| `loadMermaid()` | 单例加载 mermaid-bundle.cjs，初始化主题（dark/default） |
| `createMermaidView()` | Mermaid 代码块 NodeView |

NodeView 行为：
- 默认（失焦）：通过 `mermaid.render(id, src)` 渲染 SVG，带防抖（`md-wysiwyg.mermaidRenderDelay` 配置）
- 选中（selectNode）：显示 `<pre>` 源码编辑区
- 失焦（deselectNode）：重新渲染 SVG
- 错误处理：渲染失败时清理 mermaid 创建的临时 DOM 元素，显示错误信息

#### view（`view.js`）

| 导出 | 说明 |
|------|------|
| `patchedView` ($view) | 修复上游 NodeSchema.id bug 的 `$view` 工具函数 |

## 4. 数据流

### 4.1 模式切换（源码 → WYSIWYG）

```
用户按 Alt+M
  → toggle()
  → active = pane.getActiveItem()  // TextEditor
  → _switchToWysiwyg(textEditor, pane)
    → new MdWysiwygEditor(filePath)
      → fs.readFileSync(filePath)
      → loadMilkdown() → require('./milkdown-bundle.cjs')
      → Editor.make()...create()
    → pane.activateItem(wysiwygEditor)
    → pane.destroyItem(textEditor)
```

### 4.2 模式切换（WYSIWYG → 源码）

```
用户按 Alt+M
  → toggle()
  → active = pane.getActiveItem()  // MdWysiwygEditor
  → _switchToSource(wysiwygEditor)
    → wysiwygEditor.getMarkdownContent()
      → milkdownEditor.action(getMarkdown())
    → atom.workspace.open(filePath, { activateItem: true })
    → textEditor.setText(markdown)
    → pane.destroyItem(wysiwygEditor)
```

### 4.3 内容编辑

```
用户在 Milkdown 中编辑
  → ProseMirror 更新 document
  → listenerCtx.markdownUpdated 触发
    → storedMarkdown = md
    → 设置 modified = true
    → emitter.emit('did-change-modified')
```

### 4.4 文件保存

```
用户触发保存 (Cmd+S)
  → MdWysiwygEditor.save()
    → getMarkdownContent()
      → milkdownEditor.action(getMarkdown()) → Markdown 文本
    → fs.promises.writeFile(filePath, markdown)
    → modified = false
```

## 5. 构建系统

```
📦 构建产物（npm run build）
├── milkdown-entry.mjs ──esbuild-milkdown──▶ lib/milkdown-bundle.cjs
├── mermaid (node_modules) ──esbuild-mermaid──▶ lib/mermaid-bundle.cjs
└── lib/md-wysiwyg.js ← 手写 CJS，运行时 require('./milkdown-bundle.cjs')
```

| 脚本 | 入口 | 输出 | 说明 |
|------|------|------|------|
| `esbuild-milkdown.js` | `milkdown-entry.mjs` | `lib/milkdown-bundle.cjs` | 打包 Milkdown + 自定义插件 |
| `esbuild-mermaid.js` | `node_modules/mermaid/dist/mermaid.core.mjs` | `lib/mermaid-bundle.cjs` | 打包 Mermaid |

构建配置特点：
- `platform: 'node'`，`target: 'node20'`
- `external: ['atom', 'electron']`（Pulsar 运行时提供）
- mermaid 构建额外 `external: ['mermaid']`，通过 `require('./mermaid-bundle.cjs')` 运行时按需加载
- node: 协议别名：`node:crypto` → `crypto` 等

## 6. 文件结构

```
md-wysiwyg/
  lib/
    md-wysiwyg.js                 # 主模块 + MdWysiwygEditor（手写 CJS）
    milkdown-bundle.cjs            # [构建产物] Milkdown + 自定义插件
    mermaid-bundle.cjs             # [构建产物] Mermaid 渲染引擎
  milkdown-plugins/
    highlight.js                   # 代码高亮 NodeView + Mermaid 分发
    math.js                        # KaTeX 数学公式插件
    mermaid.js                     # Mermaid 图表 NodeView
    paragraph-info.js              # 源码展开（分隔符语法）
    view.js                        # patched $view 工具函数
  milkdown-entry.mjs               # Milkdown bundle 入口（ESM barrel）
  esbuild-milkdown.js              # Milkdown 构建脚本
  esbuild-mermaid.js               # Mermaid 构建脚本
  styles/
    md-wysiwyg.less                # 编辑器基础样式（使用 @ui-variables）
    prosemirror-overrides.less     # ProseMirror 样式覆盖
    math.less                      # KaTeX 渲染样式
    mermaid.less                   # Mermaid 渲染样式
  keymaps/
    md-wysiwyg.json                # Alt+M → md-wysiwyg:toggle
  menus/
    md-wysiwyg.json                # Packages > Markdown WYSIWYG > Toggle WYSIWYG
  package.json
```

## 7. 配置项

| 配置键 | 类型 | 默认值 | 范围 | 说明 |
|--------|------|--------|------|------|
| `fontSize` | integer | 0 | 0-32 | 自定义字体大小，0 = 使用主题默认 |
| `editorMaxWidth` | integer | 900 | 400-1600 | 编辑器内容区最大宽度（px） |
| `mermaidRenderDelay` | integer | 500 | 100-3000 | Mermaid 渲染防抖延迟（ms） |

配置通过 `atom.config.get/observe` 在 MdWysiwygEditor 构造函数中读取并应用为 inline style。

## 8. 键绑定

| 快捷键 | 命令 | 作用域 | 说明 |
|--------|------|--------|------|
| `Alt+M` | `md-wysiwyg:toggle` | `atom-workspace` | 在 WYSIWYG 和源码模式间切换 |
| `Cmd+B` | Milkdown 内置 | ProseMirror | 加粗 |
| `Cmd+I` | Milkdown 内置 | ProseMirror | 斜体 |
| `Cmd+S` | Pulsar 内置 | 全局 | 保存（MdWysiwygEditor.save 拦截） |

> `Ctrl+M` 在 `atom-text-editor` 作用域中被默认绑定为 `editor:newline`（ASCII 控制字符映射），不可用于插件快捷键。因此选用 `Alt+M`。

## 9. 性能考量

| 场景 | 策略 |
|------|------|
| 大文件（>5000 行） | Mermaid 渲染防抖（可配置）；KaTeX 按需渲染 |
| 频繁编辑 | ProseMirror 增量更新机制天然支持 |
| 模式切换 | WYSIWYG→源码：直接 fs.readFileSync；源码→WYSIWYG：读取磁盘文件重建编辑器 |
| 主题切换 | CSS 变量（@ui-variables）动态响应，不重建编辑器实例 |
| Mermaid DOM 泄漏 | 渲染失败时手动清理 mermaid 创建的临时 DOM 元素（`document.getElementById('d' + id).remove()`） |
| Milkdown 加载 | 单例 require，CSS 注入带幂等检查（katexCSSInjected / hljsCSSInjected） |
