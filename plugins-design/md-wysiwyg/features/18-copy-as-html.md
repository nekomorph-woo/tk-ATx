# 复制为 HTML

## 功能描述

提供“复制为 HTML”命令，将当前选区或整篇 Markdown 渲染为 HTML 并写入剪贴板。

## 为什么做？

很多用户会把 Markdown 内容迁移到邮件、CMS、博客后台或工单系统。复制 HTML 能减少重新排版成本。

## 收益

- 扩展 Markdown 内容的使用出口。
- 对写邮件、发布博客、复制技术文档片段有帮助。
- 不需要一开始实现完整导出系统。

## 基础设施支持情况

- 现有依赖：Milkdown serializer/ProseMirror doc、Markdown AST 生态。
- 可能新增代码：选区提取、Markdown 到 HTML 渲染、剪贴板写入。
- 可能新增依赖：`remark-rehype`、`rehype-stringify`、`rehype-sanitize`，或使用现有 Milkdown/DOM 渲染结果。

## 实现要点

- 初版支持复制整篇为 HTML。
- 后续支持复制选区为 HTML。
- KaTeX 和 Mermaid 可选择复制为当前渲染 HTML/SVG 或保留源码块。
- 剪贴板同时写入 `text/html` 和 `text/plain`。

## 风险与注意事项

- HTML 输出必须 sanitize。
- Mermaid/KaTeX 输出在目标系统中的样式兼容性不可完全保证。
