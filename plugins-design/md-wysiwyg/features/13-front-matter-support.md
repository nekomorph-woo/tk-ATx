# Front Matter 支持

## 功能描述

识别文档开头的 YAML front matter，将其作为独立块展示，可折叠、可源码编辑。

## 为什么做？

博客、静态站点、文档站大量使用 front matter。WYSIWYG 编辑器如果把它当普通分隔线和文本处理，会影响用户对文档元数据的编辑信心。

## 收益

- 提升博客/知识库类 Markdown 兼容性。
- 让元数据编辑更清晰。
- 降低误改 front matter 结构的风险。

## 基础设施支持情况

- 现有依赖：Milkdown parser 管线、remark 生态。
- 可能新增代码：front matter remark 插件、ProseMirror node schema、NodeView、序列化规则。
- 可能新增依赖：`remark-frontmatter` 或等价 micromark extension。

## 实现要点

- 仅识别文档开头 `---` 包裹的 YAML。
- 默认折叠显示关键字段数量或标题。
- 展开后以源码块方式编辑。
- 序列化必须保持原始 front matter 位置。

## 风险与注意事项

- 不应误识别正文中的 `---`。
- 暂不做 YAML 表单化编辑，先保证源码块稳定。
