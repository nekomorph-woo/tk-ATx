# Atom 插件开发 - 快速开始

> Atom 已于 2022 年 12 月归档，社区分支 [Pulsar](https://pulsar-edit.dev) 继续维护。本指南同时适用。

---

## 目录

1. [Hello World 完整代码演练](#1-hello-world-完整代码演练)
2. [Package API 基础](#2-package-api-基础)
3. [内置 API 详解](#3-内置-api-详解)
4. [创建视图](#4-创建视图)
5. [键绑定、上下文菜单与状态栏](#5-键绑定上下文菜单与状态栏)
6. [测试你的包](#6-测试你的包)
7. [发布你的包](#7-发布你的包)

---

## 1. Hello World 完整代码演练

### 包结构

```
my-package/
  package.json
  lib/
    my-package.js          # 主模块
    my-package-view.js     # 视图类
  keymaps/
    my-package.json        # 键绑定
  menus/
    my-package.json        # 应用 & 上下文菜单
  spec/
    my-package-spec.js     # 测试
  styles/
    my-package.less        # 样式
```

### package.json

```json
{
  "name": "my-package",
  "main": "./lib/my-package",
  "version": "0.1.0",
  "description": "A short description of my package",
  "activationCommands": {
    "atom-workspace": "my-package:toggle"
  },
  "repository": "https://github.com/you/my-package",
  "license": "MIT",
  "engines": {
    "atom": ">=1.0.0 <2.0.0"
  }
}
```

### 主模块（`lib/my-package.js`）

```js
import MyPackageView from './my-package-view';
import { CompositeDisposable } from 'atom';

export default {
  myPackageView: null,
  modalPanel: null,
  subscriptions: null,

  activate(state) {
    this.myPackageView = new MyPackageView(state.myPackageViewState);

    this.modalPanel = atom.workspace.addModalPanel({
      item: this.myPackageView.getElement(),
      visible: false
    });

    this.subscriptions = new CompositeDisposable();
    this.subscriptions.add(atom.commands.add('atom-workspace', {
      'my-package:toggle': () => this.toggle()
    }));
  },

  deactivate() {
    this.modalPanel.destroy();
    this.subscriptions.dispose();
    this.myPackageView.destroy();
  },

  serialize() {
    return {
      myPackageViewState: this.myPackageView.serialize()
    };
  },

  toggle() {
    if (this.modalPanel.isVisible()) {
      this.modalPanel.hide();
    } else {
      this.modalPanel.show();
    }
  }
};
```

### 视图类（`lib/my-package-view.js`）

```js
export default class MyPackageView {
  constructor(serializedState) {
    this.element = document.createElement('div');
    this.element.classList.add('my-package');

    const message = document.createElement('div');
    message.textContent = 'The MyPackage package is Alive!';
    message.classList.add('message');
    this.element.appendChild(message);
  }

  serialize() { return {}; }
  destroy() { this.element.remove(); }
  getElement() { return this.element; }
}
```

### 执行流程

1. **Atom 启动** → 读取 `package.json`，因 `activationCommands` 存在，**不加载**包
2. **用户触发命令** → Atom 加载主模块，调用 `activate(state)`
3. **`activate()`** → 创建视图、模态面板、注册命令
4. **用户触发 `my-package:toggle`** → 调用 `toggle()`，显示/隐藏面板
5. **Atom 关闭** → 调用 `serialize()` 持久化状态，然后 `deactivate()` 清理

---

## 2. Package API 基础

### 生命周期方法

| 方法 | 调用时机 | 用途 |
|------|----------|------|
| `activate(state)` | 包被激活 | 初始化命令、视图、监听器。`state` 是之前序列化的对象 |
| `deactivate()` | 包被停用 | 清理：释放订阅、移除视图、释放资源 |
| `serialize()` | 关闭前 | 返回可 JSON 序列化的对象用于状态恢复 |

### 激活模式

**惰性激活（推荐）：**

```json
{
  "activationCommands": {
    "atom-workspace": "my-package:toggle"
  }
}
```

**基于钩子的激活：**

```json
{
  "activationHooks": ["language-javascript:grammar-used"]
}
```

**立即激活（不推荐）：** 省略 `activationCommands`，每次 Atom 启动都加载，拖慢启动速度。

### package.json Atom 专有字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `main` | `string` | 入口文件路径 |
| `activationCommands` | `object` | 惰性激活的命令映射 |
| `activationHooks` | `string[]` | 惰性激活的钩子名 |
| `consumedServices` | `object` | 消费的服务 |
| `providedServices` | `object` | 提供的服务 |
| `engines` | `object` | 引擎兼容性 |
| `configSchema` | `object` | 用户配置项的 JSON Schema |
| `atomTestRunner` | `string` | 自定义测试运行器路径 |
| `theme` | `string` | `"ui"` 或 `"syntax"`（主题包专用） |

---

## 3. 内置 API 详解

所有 API 通过全局 `atom` 对象访问。

### 3.1 CompositeDisposable

集中管理资源清理，一次性释放所有 Disposable。

```js
import { CompositeDisposable } from 'atom';

// 在 activate() 中
this.subscriptions = new CompositeDisposable();
this.subscriptions.add(
  atom.commands.add('atom-workspace', { 'my-pkg:cmd': () => doSomething() }),
  atom.workspace.onDidOpen(() => console.log('opened')),
  atom.config.observe('my-pkg.setting', (val) => { this.setting = val; })
);

// 在 deactivate() 中
this.subscriptions.dispose(); // 释放所有资源
```

### 3.2 TextEditor API

```js
const editor = atom.workspace.getActiveTextEditor();
if (!editor) return;

// 文本操作
editor.getText();                          // 获取全部文本
editor.setText('new content');             // 设置全部文本
editor.insertText('hello');                // 在光标处插入
editor.getSelectedText();                  // 获取选中文本

// 光标/位置
editor.getCursorBufferPosition();          // {row, column}
editor.setCursorBufferPosition([5, 10]);
editor.getCursorLineText();                // 当前行文本

// 选择
editor.selectAll();
editor.setSelectedBufferRange([[0, 0], [5, 0]]);

// Buffer
const buffer = editor.getBuffer();
buffer.getRange();
buffer.getLineCount();

// 标记与装饰
const marker = editor.markBufferRange([[0, 0], [0, 5]]);
editor.decorateMarker(marker, { type: 'highlight', class: 'my-highlight' });

// 文件路径
editor.getPath();
editor.getTitle();
```

### 3.3 Workspace API

```js
// 打开文件
atom.workspace.open('path/to/file.js');   // Promise<TextEditor>
atom.workspace.open();                     // 新建无标题编辑器

// 活动项
atom.workspace.getActiveTextEditor();
atom.workspace.getActivePane();
atom.workspace.getActivePaneItem();

// 面板
atom.workspace.addModalPanel({ item: el, visible: true });
atom.workspace.addBottomPanel({ item: el, priority: 100 });
atom.workspace.addLeftPanel({ item: el, priority: 100 });
atom.workspace.addRightPanel({ item: el, priority: 100 });

// 事件
atom.workspace.onDidOpen((event) => { /* uri, item, pane */ });
atom.workspace.onDidDestroyPane((pane) => { /* ... */ });
atom.workspace.onDidChangeActiveTextEditor((editor) => { /* ... */ });
atom.workspace.onDidAddTextEditor(({textEditor}) => { /* ... */ });
```

### 3.4 NotificationManager API

```js
atom.notifications.addInfo('Information message', {
  detail: 'Optional detailed description',
  dismissable: true
});
atom.notifications.addSuccess('Done!');
atom.notifications.addWarning('Proceed with caution');
atom.notifications.addError('Error occurred', {
  detail: 'Stack trace info',
  dismissable: true,
  buttons: [
    { text: 'View Docs', onDidClick: () => shell.openExternal('https://example.com') }
  ]
});
```

### 3.5 Config 系统

```js
// 读取
atom.config.get('my-package.mySetting');

// 写入
atom.config.set('my-package.mySetting', 'newValue');

// 监听变化
this.subscriptions.add(
  atom.config.observe('my-package.mySetting', (newValue) => {
    console.log('Changed to:', newValue);
  })
);
```

在 `package.json` 中定义 Schema：

```json
{
  "configSchema": {
    "showIcons": {
      "type": "boolean",
      "default": true,
      "title": "Show Icons",
      "description": "Toggle icon visibility."
    },
    "maxResults": {
      "type": "integer",
      "default": 10,
      "minimum": 1,
      "maximum": 100
    }
  }
}
```

### 3.6 TooltipManager API

```js
atom.tooltips.add(element, {
  title: 'Tooltip Title',
  text: 'Tooltip body text',
  placement: 'top',
  trigger: 'hover',
  delay: { show: 300 },
  keyBindingCommand: 'my-package:cmd',
  class: 'my-custom-tooltip'
});
```

---

## 4. 创建视图

### 4.1 纯 DOM（推荐）

```js
export default class MyView {
  constructor(serializedState) {
    this.element = document.createElement('div');
    this.element.classList.add('my-view');

    this.heading = document.createElement('h1');
    this.heading.textContent = 'My Package';
    this.element.appendChild(this.heading);

    this.button = document.createElement('button');
    this.button.textContent = 'Click me';
    this.button.addEventListener('click', () => this.increment());
    this.element.appendChild(this.button);
  }

  increment() { /* ... */ }
  serialize() { return {}; }
  destroy() { this.element.remove(); }
  getElement() { return this.element; }
}
```

### 4.2 space-pen Views（旧版）

需要安装 `atom-space-pen-views`：

```bash
apm install atom-space-pen-views
```

```js
import { View } from 'atom-space-pen-views';

export default class MyView extends View {
  static content() {
    this.div({ class: 'my-view' }, () => {
      this.h1('My Package');
      this.button('Click me', { click: 'increment' });
    });
  }

  initialize() { this.count = 0; }
  increment() { this.count++; }
}
```

关键 space-pen 类：
- **`View`** — 基类，用 `static content()` 定义 HTML 模板
- **`SelectListView`** — 带搜索的列表选择器
- **`TextEditorView`** — 包裹 mini TextEditor 的输入框
- **`ScrollView`** — 可滚动容器

### 4.3 atom-select-list（较新替代方案）

```js
import SelectListView from 'atom-select-list';

const listView = new SelectListView({
  items: ['Option A', 'Option B'],
  elementForItem: (item) => {
    const li = document.createElement('li');
    li.textContent = item;
    return li;
  },
  filterQueryForItem: (item) => item,
  didConfirmSelection: (item) => { panel.hide(); },
  didCancelSelection: () => { panel.hide(); }
});

const panel = atom.workspace.addModalPanel({ item: listView.element });
listView.focus();
```

---

## 5. 键绑定、上下文菜单与状态栏

### 5.1 键绑定

`keymaps/my-package.json`：

```json
{
  "atom-workspace": {
    "ctrl-alt-o": "my-package:toggle"
  },
  "atom-text-editor:not([mini])": {
    "ctrl-shift-m": "my-package:transform"
  },
  ".platform-darwin atom-text-editor": {
    "cmd-enter": "my-package:submit"
  }
}
```

关键概念：
- **CSS 选择器优先级**：更具体的选择器胜出
- **`:not([mini])`**：排除 mini 编辑器（搜索栏、输入框）
- **平台选择器**：`.platform-darwin`、`.platform-linux`、`.platform-win32`
- **调试**：`Cmd+Shift+.` 打开 Keybinding Resolver

### 5.2 上下文菜单

`menus/my-package.json`：

```json
{
  "context-menu": {
    "atom-text-editor": [
      { "label": "Toggle my-package", "command": "my-package:toggle" },
      { "type": "separator" },
      { "label": "Transform Selection", "command": "my-package:transform" }
    ]
  }
}
```

### 5.3 状态栏项

在 `package.json` 中声明消费 `status-bar` 服务：

```json
{
  "consumedServices": {
    "status-bar": {
      "versions": { "^1.0.0": "consumeStatusBar" }
    }
  }
}
```

在主模块中：

```js
export default {
  statusBarTile: null,

  consumeStatusBar(statusBar) {
    const element = document.createElement('div');
    element.classList.add('my-package-status');
    element.textContent = 'Ready';

    this.statusBarTile = statusBar.addLeftTile({
      item: element,
      priority: 100
    });
  },

  deactivate() {
    if (this.statusBarTile) this.statusBarTile.destroy();
  }
};
```

---

## 6. 测试你的包

Atom 默认使用 **Jasmine 1.3**（带自定义异步辅助函数）。测试文件放在 `spec/` 下，必须以 `-spec.js` 结尾。

### 基本测试

```js
import MyPackage from '../lib/my-package';

describe('MyPackage', () => {
  let workspaceElement, activationPromise;

  beforeEach(() => {
    workspaceElement = atom.views.getView(atom.workspace);
    activationPromise = atom.packages.activatePackage('my-package');
  });

  describe('when my-package:toggle is triggered', () => {
    it('shows and hides the modal panel', () => {
      expect(workspaceElement.querySelector('.my-package')).not.toExist();

      atom.commands.dispatch(workspaceElement, 'my-package:toggle');
      waitsForPromise(() => activationPromise);

      runs(() => {
        let el = workspaceElement.querySelector('.my-package');
        expect(el).toExist();
        expect(el).isVisible();

        atom.commands.dispatch(workspaceElement, 'my-package:toggle');
        expect(el).not.toBeVisible();
      });
    });
  });
});
```

### 异步测试

```js
describe("opening a file", () => {
  it("should open in an editor", () => {
    waitsForPromise(() => {
      return atom.workspace.open('sample.js').then((editor) => {
        expect(editor.getPath()).toContain('sample.js');
      });
    });
  });
});
```

### 自定义匹配器

| 匹配器 | 说明 |
|--------|------|
| `expect(el).toExist()` | 元素存在于 DOM |
| `expect(el).toBeVisible()` | 元素可见 |
| `expect(el).toHaveFocus()` | 元素获得焦点 |
| `expect(obj).toBeInstanceOf(Class)` | 实例检查 |
| `expect(arr).toHaveLength(n)` | 数组长度检查 |

### 运行测试

```bash
# 编辑器内
# 命令面板: Window: Run Package Specs

# 命令行
atom --test ./spec/my-package-spec.js
atom --test --timeout 60 ./spec/
```

### 使用 Jasmine 3.x

```bash
npm install --save-dev atom-jasmine3-test-runner
```

```json
{
  "atomTestRunner": "./node_modules/atom-jasmine3-test-runner"
}
```

---

## 7. 发布你的包

### 发布到 Pulsar

```bash
# 1. 认证
ppm login

# 2. 确保 package.json 包含 name, version, description, repository, license
# 3. 创建 git tag
git tag v0.1.0
git push origin --tags

# 4. 发布
ppm publish           # 发布当前版本
ppm publish minor    # 升级次版本并发布
ppm publish major    # 升级主版本并发布
ppm publish patch    # 升级补丁版本并发布
```

### CI/CD 示例（GitHub Actions）

```yaml
name: Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '16'
      - run: npm install
      - run: npm test
```

---

## 参考链接

| 资源 | URL |
|------|-----|
| Atom Flight Manual（社区镜像） | https://flight-manual.atom-editor.cc/ |
| Atom API Docs | https://flight-manual.atom-editor.cc/api/v1.63.1/ |
| Pulsar 文档 | https://docs.pulsar-edit.dev/ |
| Pulsar API | https://docs.pulsar-edit.dev/api/pulsar/v1.0.0/TextBuffer/ |
| Package: Word Count 教程 | https://flight-manual.atom-editor.cc/hacking-atom/sections/package-word-count/ |
| Writing Specs | https://flight-manual.atom-editor.cc/hacking-atom/sections/writing-specs/ |
| atom-jasmine3-test-runner | https://github.com/UziTech/atom-jasmine3-test-runner |
| atom-space-pen-views | https://github.com/atom/atom-space-pen-views |
| atom-select-list | https://github.com/atom/atom-select-list |
