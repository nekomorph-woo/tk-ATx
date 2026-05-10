## 问题陈述

Pulsar 编辑器的 Markdown 编辑体验停留在纯文本源码层面。用户编辑 `.md` 文件时无法直观看到最终渲染效果，需要频繁切换到预览面板或依赖外部编辑器（如 Typora）才能获得所见即所得的写作体验。现有的 `markdown-preview` 包仅提供分屏预览，不具备真正的 WYSIWYG 编辑能力。

## 解决方案

开发 `md-wysiwyg` 插件，为 Pulsar 提供 Typora 级别的所见即所得 Markdown 编辑体验。通过自定义命令在 WYSIWYG 模式（基于 Milkdown）和 Pulsar 原生源码模式之间一键切换。存储格式始终为纯 Markdown。

**交互模式**：用户在源码编辑器中按 `Alt+M` 切换到 WYSIWYG 视图，再按 `Alt+M` 切换回源码。插件不自动拦截 `.md` 文件打开，而是由用户主动触发。

## 用户故事

1. 作为 Markdown 写作者，我想要在编辑时直接看到渲染后的格式（标题大小、加粗效果、代码高亮），以便专注于内容而非语法
2. 作为 Markdown 写作者，我想要点击已渲染的块级元素（标题、代码块、引用）时看到原始语法以便精确编辑，以便兼顾可视化和精确控制
3. 作为技术文档作者，我想要 GFM 扩展语法（表格、任务列表、删除线、脚注）的正确渲染和编辑，以便撰写规范的 GitHub 风格文档
4. 作为学术写作者，我想要行内和块级 LaTeX 数学公式的实时渲染，以便在编辑器内完成数学内容编写
5. 作为架构师，我想要 Mermaid 图表（流程图、时序图、甘特图等）的实时渲染预览，以便在文档中直观表达系统设计
6. 作为开发者，我想要通过快捷键（`Alt+M`）在 WYSIWYG 模式和纯源码模式之间切换，以便在需要时查看和编辑原始 Markdown 语法
7. 作为 Atom 用户，我想要插件的视觉风格与当前 Atom 主题保持一致，以便获得统一的编辑器体验
8. 作为用户，我想要自定义编辑器宽度和字体大小，以便适应不同屏幕和偏好

## 实现决策

- **核心架构：手动切换模式** — 不自动拦截 `.md` 文件打开。通过 `atom.commands.add` 注册 `md-wysiwyg:toggle` 命令，根据当前活跃 item 类型（`MdWysiwygEditor` 或 `TextEditor`）自动判断切换方向。WYSIWYG 视图通过 `pane.activateItem()` 直接创建实例，不依赖 `workspace.open(uri)` 的 URI 缓存机制
- **编辑引擎：Milkdown v7+** — ProseMirror 驱动的 WYSIWYG 编辑器。作为 npm 依赖引入，通过 esbuild 打包为 CJS bundle 运行时加载
- **Markdown 解析：remark + rehype 生态** — Milkdown 底层使用 remark 解析 Markdown AST
- **数学公式渲染：KaTeX** — 通过自定义 Milkdown 插件集成，支持行内（`$...$`）和块级（`$$...$$`）公式。错误时回退显示原始 LaTeX 源码
- **代码高亮：highlight.js** — 通过自定义代码块 NodeView 实现语法高亮，支持 18 种常用语言及别名映射
- **Mermaid 图表** — 作为代码块 NodeView 的特例（`language === 'mermaid'`），选中时显示源码，失焦时渲染为 SVG 图表。主题跟随编辑器主题（dark/default）
- **源码展开** — 通过 ProseMirror 事务插件实现：选中 strong/emphasis/inlineCode mark 时展开为原始分隔符语法编辑，失焦时折叠回渲染视图
- **主题适配** — 读取 Pulsar 当前 UI 主题名判断 dark/light，设置 `data-theme` 属性。KaTeX/highlight.js 通过运行时注入 CSS。编辑器样式使用 Pulsar 主题变量（`@ui-variables`）
- **构建策略** — esbuild 仅打包第三方依赖（milkdown → `milkdown-bundle.cjs`，mermaid → `mermaid-bundle.cjs`），主入口 `lib/md-wysiwyg.js` 为手写 CJS，运行时 `require()` 加载 bundle

### 模块划分

| 模块 | 文件 | 职责 |
|------|------|------|
| **MdWysiwygPackage** | `lib/md-wysiwyg.js`（下半部分） | 包生命周期、命令注册、toggle 逻辑、主题适配 |
| **MdWysiwygEditor** | `lib/md-wysiwyg.js`（上半部分） | 核心视图，管理 Milkdown 实例的创建/销毁/配置 |
| **Milkdown Plugins** | `milkdown-plugins/*.js` | 数学公式、代码高亮、Mermaid、源码展开等自定义插件 |
| **View Utility** | `milkdown-plugins/view.js` | 修复上游 `$view` 的 NodeSchema.id bug |

### 架构决策

- Milkdown 作为唯一的编辑引擎，不引入 Vditor 或其他编辑器库作为备选
- 存储始终为纯 Markdown，通过 Milkdown 的序列化器保证输出格式一致性
- 不实现图片上传功能（PicGo 等），这是独立关注点，可在后续版本中通过 Service 系统集成
- 不实现导出功能（PDF/HTML），Atom 自身或现有插件已覆盖
- 不自动拦截 `.md` 文件打开，由用户通过快捷键或菜单主动切换

## 测试决策

### 好测试的标准

- 测试编辑器外部行为：输入 Markdown → 验证渲染结果；编辑渲染内容 → 验证 Markdown 输出
- 不测试 ProseMirror 内部状态或 Milkdown 插件实现细节

### 需要测试的模块

| 模块 | 测试重点 |
|------|----------|
| MdWysiwygPackage | 激活/停用生命周期、opener 注册、toggle 命令正确切换 |
| MdWysiwygEditor | Milkdown 实例创建/销毁、内容序列化为纯 Markdown、配置项响应（fontSize、editorMaxWidth） |

> 注意：当前 spec/ 已清理（原脚手架测试不匹配实际实现），测试待后续补充。

## 范围外

- 图片上传集成（PicGo、iUploader 等）
- PDF/HTML/EPUB 导出
- 大纲/TOC 侧栏面板
- 多光标编辑、拖拽排序块级元素
- 协作编辑（Teletype 兼容）
- Vim 模式兼容
- 表格图形化编辑工具栏（增删行列、列宽调整）
- Focus Mode / Typewriter Mode
- 自动拦截 `.md` 文件打开（当前为手动切换模式）

## 补充说明

- **Milkdown 版本**：使用 v7.20.x，通过 `@milkdown/kit` 获取核心包集合
- **Mermaid 版本**：v11.14.x，使用 `securityLevel: 'loose'` 允许 HTML 渲染
- **性能考虑**：大文件（>10000 行）场景下需验证 Milkdown 的渲染性能，必要时引入虚拟滚动
- **兼容性**：仅在 Pulsar（Electron 25+）上测试
- **设计文档位置**：`plugins-design/md-wysiwyg/`
