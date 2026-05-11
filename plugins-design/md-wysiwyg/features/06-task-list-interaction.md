# 任务列表交互

## 功能描述

让 GFM 任务列表的 checkbox 可直接点击，自动在 Markdown 中切换 `[ ]` 和 `[x]` 状态。

## 为什么做？

任务列表是非常常见的 Markdown 功能，用户看到 checkbox 后自然会想点击它。不可点击会让 WYSIWYG 体验显得“不完整”。

## 收益

- 成本较低但体验提升明显。
- 适合 TODO、计划、PR checklist 等常见场景。
- 强化 GFM 支持的完整度。

## 基础设施支持情况

- 现有依赖：Milkdown GFM task list item schema。
- 可能新增代码：task list item NodeView 或 click handler、状态切换 transaction。
- 暂不需要新增第三方依赖。

## 实现要点

- 点击 checkbox 时更新对应节点 checked 属性。
- 更新后保持光标和滚动位置稳定。
- 支持键盘快捷键切换当前任务项。

## 风险与注意事项

- checkbox 点击要阻止浏览器默认行为和 ProseMirror 错误选区。
- 嵌套任务列表需要确保定位准确。
