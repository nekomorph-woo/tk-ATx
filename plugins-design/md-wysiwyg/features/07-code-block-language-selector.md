# 代码块语言选择器

## 功能描述

在代码块顶部显示语言标签，点击后可选择或输入语言，例如 `javascript`、`typescript`、`python`、`bash`、`json`、`mermaid`。

## 为什么做？

代码块高亮依赖语言标记。源码中写 ```` ```js ```` 很自然，但 WYSIWYG 中需要可视化入口来修改语言。

## 收益

- 提升代码块编辑效率。
- 让 highlight.js 能更稳定地发挥作用。
- Mermaid 可以通过语言切换自然进入图表渲染模式。

## 基础设施支持情况

- 现有依赖：highlight.js、自定义 `codeBlockViewPlugin`。
- 现有代码：已维护语言别名和 Mermaid 特例。
- 可能新增代码：代码块 header UI、语言下拉菜单、更新 node attrs 的 command。
- 暂不需要新增第三方依赖。

## 实现要点

- 普通代码块 NodeView 增加 header。
- 语言变更时 dispatch transaction 更新 `language` attr。
- 语言列表优先来自当前 highlight.js 支持集合。
- 支持清空语言，回到 plaintext。

## 风险与注意事项

- `contentDOM` 结构变更要小心，避免破坏代码编辑。
- Mermaid 和普通代码块之间切换时 NodeView 需要正确重建。
