# 大纲导航

## 功能描述

根据 Markdown 标题生成文档大纲，支持点击跳转、当前位置高亮、折叠层级。

## 为什么做？

长文档是 Markdown 的主要使用场景之一。没有大纲时，WYSIWYG 模式下用户很难快速定位结构。

## 收益

- 提升长文档阅读和编辑效率。
- 让插件更适合技术方案、PRD、知识库文章。
- 可作为后续 TOC 插入功能的基础。

## 基础设施支持情况

- 现有依赖：Milkdown/ProseMirror document tree。
- 可能新增代码：heading scanner、sidebar/panel view、scroll observer、jump command。
- 可能使用 Pulsar API：panel、workspace item、dock。
- 暂不需要新增第三方依赖。

## 实现要点

- 从 ProseMirror doc 中扫描 heading node。
- 文档变化时 debounce 更新大纲。
- 点击大纲项滚动到对应节点。
- 支持 `Alt+W` 切换后保持大纲状态。

## 风险与注意事项

- 大文档频繁扫描需要防抖。
- 标题内容重复时需要稳定定位策略。
