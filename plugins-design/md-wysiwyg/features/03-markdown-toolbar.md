# Markdown 工具栏

## 功能描述

提供固定或浮动工具栏，包含加粗、斜体、行内代码、链接、标题、列表、引用、代码块、表格、公式、Mermaid 等常用操作。

## 为什么做？

工具栏是 WYSIWYG 编辑器的基础交互。它让用户无需记住快捷键和 Markdown 语法，也能完成常见格式编辑。

## 收益

- 降低插件上手门槛。
- 提升格式编辑效率。
- 给复杂功能提供显性入口，例如插入表格、公式、图表。

## 基础设施支持情况

- 现有依赖：Milkdown command、ProseMirror selection、mark/node schema。
- 现有代码：strong、emphasis、inlineCode 已有源码展开能力。
- 可能新增代码：toolbar DOM、command adapter、selection state observer、按钮 active/disabled 状态。
- 暂不需要新增第三方依赖。

## 实现要点

- 先实现常驻顶部工具栏，再考虑选区 Bubble Menu。
- 按钮状态跟随当前选区 mark/node。
- 命令层和 UI 层分离，便于 Slash Command 复用。
- 工具栏按钮需要适配 Pulsar 主题变量。

## 风险与注意事项

- 工具栏不应抢占编辑器焦点。
- 选区变化频繁，需要避免过度更新 DOM。
