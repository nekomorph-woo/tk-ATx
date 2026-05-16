# 源码切换连续性

## 功能描述

在 WYSIWYG 和源码模式之间切换时尽量保持光标位置、滚动位置和当前编辑上下文。

## 为什么做？

当前插件的核心交互是 `Alt+W` 切换。如果每次切换都丢失位置，用户会很难在“可视化编辑”和“精确源码编辑”之间来回工作。

## 收益

* 让手动切换模式真正可用。

* 提升长文档编辑体验。

* 降低用户在源码和 WYSIWYG 模式之间切换的认知成本。

## 基础设施支持情况

* 现有依赖：Pulsar TextEditor API、ProseMirror selection、Milkdown editor view。

* 可能新增代码：Markdown offset 映射、heading/block anchor 映射、scroll restore。

* 暂不需要新增第三方依赖。

## 实现要点

* 初版按当前块文本或 heading 找近似位置。

* 保存切换前的 selection/from/to 和 scrollTop。

* 源码模式恢复到相近行列，WYSIWYG 恢复到相近节点。

* 无法精确映射时退化为滚动比例。

## 风险与注意事项

* Markdown 序列化会改变空白和表格格式，精确 offset 映射困难。

* 应优先实现“足够接近”，不要过早追求完美。

