# 链接编辑

## 功能描述

支持选中文本后创建链接，点击链接时显示浮层，可打开链接、编辑 URL、编辑标题、移除链接。

## 为什么做？

链接是 Markdown 文档的核心功能之一。源码模式下链接语法可控，但 WYSIWYG 模式需要更直接的编辑入口。

## 收益

- 提升技术文档、笔记、README 编写体验。
- 减少手写 `[text](url)` 的打断。
- 为复制 HTML、预览发布等后续能力打基础。

## 基础设施支持情况

- 现有依赖：Milkdown commonmark link mark、ProseMirror marks。
- 可能新增代码：`Cmd+K` command、link popover、URL 校验、打开外部链接逻辑。
- 可能使用 Pulsar API：`shell.openExternal` 或 Electron shell。
- 暂不需要新增第三方依赖。

## 实现要点

- `Cmd+K` / `Ctrl+K` 对选区创建链接。
- 光标在链接内时显示浮层。
- 支持编辑 href、title，支持移除 mark。
- URL 输入为空时可自动移除链接。

## 风险与注意事项

- 点击链接默认不应直接跳转，避免打断编辑；可用浮层按钮打开。
- 需要处理没有选区时插入链接文本的交互。
