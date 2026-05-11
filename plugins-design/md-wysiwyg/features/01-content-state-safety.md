# 内容状态安全

## 功能描述

增强 WYSIWYG 编辑态、源码态、文件保存态之间的一致性，确保切换、保存、关闭、重载时不丢内容、不把临时 UI 状态写入 Markdown。

## 为什么做？

这是所有编辑功能的地基。Markdown WYSIWYG 插件最核心的承诺是“看到的是编辑体验，保存的是正确 Markdown”。如果内容状态不可靠，后续工具栏、表格、公式、Mermaid 等功能都会被放大风险。

## 收益

- 降低保存、切换、关闭窗口时的数据丢失风险。
- 让用户敢于长期在 WYSIWYG 模式下写作。
- 为后续复杂 NodeView 和交互块提供统一的序列化边界。

## 基础设施支持情况

- 现有依赖：Milkdown、ProseMirror transaction、listener plugin。
- 现有代码：`MdWysiwygEditor.getMarkdownContent()`、`save()`、`_switchToSource()`、`sourceExpansionPlugin`。
- 可能新增代码：统一的 `beforeSerialize()` / `flushTransientState()` 管线，各 NodeView 注册 flush hook。
- 暂不需要新增第三方依赖。

## 实现要点

- 保存和切回源码前强制收敛所有临时 UI 状态。
- 打开源码 editor 成功后再销毁 WYSIWYG item。
- 关闭 tab、窗口 reload、deactivate 前复用同一套 flush 逻辑。
- 为 source expansion、math、Mermaid 等交互块建立统一测试用例。

## 风险与注意事项

- flush 过程不能意外触发用户可见的撤销历史。
- 临时状态收敛失败时应阻止销毁，并提示用户。
