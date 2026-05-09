# tk-ATx

个人 Atom/Pulsar 插件合集。每个子目录是独立的 Atom 插件，可单独开发验证。

## 项目结构

```
tk-ATx/
  <plugin-name>/          # 每个插件一个目录
  .claude/skills/         # 插件生成 skill
  docs/                   # 开发参考文档
```

## 插件开发准则

- 使用 ES6 Module 语法（`import`/`export`），语言为 JavaScript
- 主模块导出含 `activate()` / `deactivate()` / `serialize()` 生命周期方法的对象
- 用 `CompositeDisposable` 集中管理所有订阅，`deactivate()` 中统一 `dispose()`
- 始终声明 `activationCommands` 实现惰性激活，不要省略导致启动拖慢
- 依赖包间通信用 Service 系统（`consumedServices` / `providedServices`），不直接 `require()`
- 键绑定用 CSS 选择器控制作用域，注意 `:not([mini])` 排除 mini 编辑器
- 样式使用 Less，`@import "ui-variables"` 引入主题变量

## 开发验证

- 使用 `pulsar --dev /path/to/<plugin-name>` 打开编辑器进行开发调试
- `Cmd+Shift+F5`（`window:reload`）热重载插件
- `Alt+Cmd+I` 打开 Chrome DevTools
- `Cmd+Shift+.` 打开 Keybinding Resolver 查看键绑定匹配
- `pulsar --test spec/<plugin-name>-spec.js` 命令行运行测试

## 命名约定

- 插件目录名全小写连字符：`atom-word-count`、`linter-my-lang`
- 文件名与插件目录名一致：`lib/<plugin-name>.js`
- JS 中类名用驼峰：`AtomWordCount`
- CSS 类名用连字符：`.atom-word-count`

## 生成新插件

使用 `/create-atom-plugin` skill，基于 `references/my-package/` 骨架初始化。
