# 粘贴清理

## 功能描述

从网页、Word、Notion、浏览器等来源粘贴内容时，将 HTML 或富文本清理为干净 Markdown 结构。

## 为什么做？

Markdown 写作中粘贴是高频操作。未经处理的粘贴内容容易带入复杂 HTML、脏样式或不符合预期的结构。

## 收益

- 提升资料整理和迁移效率。
- 减少 Markdown 文件中的脏内容。
- 让 WYSIWYG 模式更适合真实写作流。

## 基础设施支持情况

- 现有依赖：Milkdown clipboard plugin、ProseMirror clipboard hooks。
- 可能新增代码：paste parser、HTML sanitize、规则化转换、来源特化清理。
- 可能新增依赖：如需要 HTML 到 Markdown，可评估 `turndown`；如只在 ProseMirror 层处理，优先不新增依赖。

## 实现要点

- 优先处理常见标签：heading、paragraph、list、blockquote、code、table、link、image。
- 去除内联 style、无关 class 和 tracking 属性。
- 保留语义结构，丢弃视觉样式。
- 对复杂 HTML 提供 fallback：粘贴为纯文本。

## 风险与注意事项

- 粘贴来源差异大，不能一次覆盖所有场景。
- 清理过度会丢信息，清理不足会污染文档。
