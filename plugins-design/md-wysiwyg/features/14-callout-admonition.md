# Callout / Admonition

## 功能描述

支持 GitHub/Obsidian 风格 callout，例如 `> [!NOTE]`、`> [!TIP]`、`> [!WARNING]`，并渲染为带类型样式的提示块。

## 为什么做？

Callout 在技术文档、知识库、教程中非常常见。它本质上仍是标准 blockquote 扩展，适合在不破坏 Markdown 兼容性的前提下增强阅读体验。

## 收益

- 提升技术文档表达能力。
- 兼容常见 Markdown 生态写法。
- 视觉上增强重点信息和警告信息。

## 基础设施支持情况

- 现有依赖：CommonMark blockquote schema。
- 可能新增代码：blockquote NodeView 或 decoration plugin、callout 类型解析、插入 command。
- 暂不需要新增第三方依赖。

## 实现要点

- 解析 blockquote 第一行 `[!TYPE]`。
- 支持 note、tip、important、warning、caution 等类型。
- Slash Command 提供插入入口。
- 序列化保持原始 blockquote Markdown。

## 风险与注意事项

- 不要破坏普通 blockquote。
- 嵌套 blockquote 和多段 callout 需要测试。
