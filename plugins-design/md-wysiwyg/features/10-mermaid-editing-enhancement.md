# Mermaid 编辑增强

## 功能描述

增强 Mermaid 代码块体验，提供源码/预览切换、重新渲染、错误提示、主题同步和常见图表模板插入。

## 为什么做？

Mermaid 是该插件的重要差异化能力。当前能渲染图表只是第一步，真实写图时最需要的是快速修错和预览切换。

## 收益

- 提升架构图、流程图、时序图编辑体验。
- 减少 Mermaid 语法错误时的排查成本。
- 让图表能力成为插件亮点。

## 基础设施支持情况

- 现有依赖：Mermaid、自定义 `createMermaidView()`。
- 现有代码：已支持 Mermaid code block 特例和主题变化事件。
- 可能新增代码：NodeView toolbar、错误区域、模板插入 command、手动 render command。
- 暂不需要新增第三方依赖。

## 实现要点

- 图表块右上角提供源码/预览切换。
- 渲染错误显示摘要，尽量保留 Mermaid 原始错误。
- Slash Command 插入 flowchart/sequence/class diagram 模板。
- 安全模式默认 strict，可信文档可打开 unsafe rendering。

## 风险与注意事项

- Mermaid 错误信息不一定包含准确行号。
- 渲染 SVG 时要继续控制安全边界。
