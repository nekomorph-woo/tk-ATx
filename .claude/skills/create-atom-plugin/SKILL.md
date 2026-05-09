---
name: create-atom-plugin
description: 创建新的 Atom/Pulsar 插件骨架。Use when 用户要求创建新插件、生成插件项目、初始化 Atom 包，或提到 "create plugin" / "新建插件" / "生成骨架"。
---

# 创建 Atom/Pulsar 插件

根据用户意图生成推荐插件名，确认后基于骨架模板初始化到当前项目目录。

## 执行流程

### 1. 生成推荐插件名

根据用户描述生成 2-3 个推荐名称，使用 `AskUserQuestion` 确认。

命名规范：
- 全小写连字符：`atom-word-count`、`language-xyz`、`linter-my-lang`
- 前缀按类型选：`atom-`（通用）/ `language-`（语法）/ `theme-`（主题）/ `linter-`（lint）/ `ui-`（界面）

### 2. 初始化骨架

基于 [reference/my-package/](reference/my-package/) 模板，复制到项目根目录 `<PLUGIN_NAME>/`。

替换规则：

| 占位符 | 替换为 | 示例 |
|--------|--------|------|
| `my-package` | 插件名（全小写连字符） | `atom-word-count` |
| `MyPackage` | 驼峰形式 | `AtomWordCount` |
| `A short description of your package` | 根据意图生成的描述 | — |

文件重命名：所有 `my-package*` 文件名替换为 `<PLUGIN_NAME>*`。

重要：
- 用 `cp -r` 复制后用 `sed` 做替换，不修改 reference/ 下的模板
- `my-package` 和 `MyPackage` 的所有出现都要替换（注释、CSS 类名、命令名）
- 完成后用 `find` 展示目录结构确认
