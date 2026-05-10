# tk-ATx

个人 Pulsar 插件合集。每个子目录是独立的 Pulsar 插件，可单独安装使用。

## 插件列表

### md-wysiwyg

Typora 级别的所见即所得 Markdown 编辑器，基于 Milkdown（ProseMirror 驱动）。

* GFM 完整支持（表格、任务列表、删除线）

* KaTeX 数学公式（`$...$` / `$$...$$`）

* Mermaid 图表渲染（流程图、时序图等）

* highlight.js 代码高亮（18 种语言）

* 选中 mark 元素时展开为原始分隔符语法编辑

* `Alt+W` 一键切换 WYSIWYG / 源码模式

* 编辑器宽度、字体大小可配置

## 安装

### 前提

* [Pulsar](https://pulsar-edit.dev/)（Atom 的社区延续版本）

* Node.js 18+

### 准备源码

```bash
git clone https://github.com/nekomorph-woo/tk-ATx.git
cd tk-ATx/md-wysiwyg
npm install
npm run build
```

如果 `pulsar -p` 不可用，先在 Pulsar 菜单中执行 **Pulsar > Install Shell Commands**。

### 本地开发安装

```bash
cd /path/to/tk-ATx/md-wysiwyg
pulsar -p install
pulsar -p link --dev
pulsar --dev /path/to/tk-ATx/md-wysiwyg
```

开发窗口打开后，在 `.md` 文件中按 `Alt+W` 触发插件。

### 日常使用安装

```bash
cd /path/to/tk-ATx/md-wysiwyg
pulsar -p install
pulsar -p link
```

安装完成后重启 Pulsar，或按 `Cmd+Shift+F5` 重载。

### 更新

```bash
cd /path/to/tk-ATx
git pull
cd md-wysiwyg
npm install
npm run build
```

然后 `Cmd+Shift+F5` 重载 Pulsar。

## 使用

### md-wysiwyg

打开任意 `.md` 文件，按 `Alt+W` 切换到 WYSIWYG 渲染视图，再按 `Alt+W` 切换回源码。

也可通过菜单 **Packages > Markdown WYSIWYG > Toggle WYSIWYG** 或右键菜单切换。

#### 配置

在 Pulsar Settings > Packages > Markdown WYSIWYG 中调整：

| 配置项                  | 默认值 | 说明                   |
| -------------------- | --- | -------------------- |
| Editor Max Width     | 900 | 编辑器内容区最大宽度（px）       |
| Font Size            | 0   | 自定义字体大小，0 = 跟随主题     |
| Mermaid Render Delay | 500 | Mermaid 图表渲染防抖延迟（ms） |
