# Playwright MCP 浏览器自动化

## 概述

Playwright MCP 是微软官方提供的 Model Context Protocol (MCP) 服务器，提供完整的浏览器自动化能力。它使用 Playwright 的可访问性树（accessibility tree）而非屏幕截图，使得 LLM 能够直接与页面元素交互，无需视觉模型。

## 配置

Playwright MCP 已在 `~/.hermes/config.yaml` 中配置：

```yaml
mcp_servers:
  playwright:
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
```

## 工具列表

### 核心导航与操作

#### mcp_playwright_browser_navigate
导航到指定 URL。

```javascript
mcp_playwright_browser_navigate({
  url: "http://120.35.0.67:28101/seeyon/main.do?method=main"
})
```

#### mcp_playwright_browser_navigate_back
返回上一页。

```javascript
mcp_playwright_browser_navigate_back()
```

#### mcp_playwright_browser_snapshot
获取当前页面的可访问性快照。这是与页面交互的基础。

```javascript
// 获取完整页面快照
mcp_playwright_browser_snapshot()

// 获取特定元素的快照
mcp_playwright_browser_snapshot({
  target: "@e123"
})

// 限制树深度以减少返回数据
mcp_playwright_browser_snapshot({
  depth: 3
})

// 保存到文件
mcp_playwright_browser_snapshot({
  filename: "snapshot.md"
})
```

#### mcp_playwright_browser_click
点击页面上的元素。

```javascript
// 使用 ref ID 点击
mcp_playwright_browser_click({
  target: "@e456"
})

// 使用 CSS/XPath 选择器
mcp_playwright_browser_click({
  target: "button[type='submit']"
})

// 双击
mcp_playwright_browser_click({
  target: "@e789",
  doubleClick: true
})

// 右键点击
mcp_playwright_browser_click({
  target: "@e789",
  button: "right"
})
```

#### mcp_playwright_browser_type
向可编辑元素输入文本。

```javascript
// 清空并输入文本（默认行为）
mcp_playwright_browser_type({
  target: "@e101",
  text: "1003854"
})
```

#### mcp_playwright_browser_press_key
按键盘按键。

```javascript
// 按 Enter 键
mcp_playwright_browser_press_key({
  key: "Enter"
})

// 按 Tab 键
mcp_playwright_browser_press_key({
  key: "Tab"
})

// 按方向键
mcp_playwright_browser_press_key({
  key: "ArrowDown"
})

// 按 Escape 键
mcp_playwright_browser_press_key({
  key: "Escape"
})
```

#### mcp_playwright_browser_close
关闭当前浏览器页面。

```javascript
mcp_playwright_browser_close()
```

### 高级表单操作

#### mcp_playwright_browser_fill_form
批量填写多个表单字段。这是比逐个 type 更高效的方式。

```javascript
mcp_playwright_browser_fill_form({
  fields: [
    {
      target: "@e101",
      text: "1003854"
    },
    {
      target: "@e102",
      text: "zxjqwe@621"
    }
  ]
})
```

#### mcp_playwright_browser_select_option
在下拉菜单中选择选项。

```javascript
// 选择单个值
mcp_playwright_browser_select_option({
  target: "@e200",
  values: ["选项1"]
})

// 选择多个值（多选下拉）
mcp_playwright_browser_select_option({
  target: "@e201",
  values: ["选项1", "选项2", "选项3"]
})
```

#### mcp_playwright_browser_file_upload
上传文件。

```javascript
// 上传单个文件
mcp_playwright_browser_file_upload({
  paths: ["/path/to/file.pdf"]
})

// 上传多个文件
mcp_playwright_browser_file_upload({
  paths: ["/path/to/file1.pdf", "/path/to/file2.png"]
})

// 取消文件选择
mcp_playwright_browser_file_upload()
```

#### mcp_playwright_browser_handle_dialog
处理页面弹出的对话框（alert、confirm、prompt）。

```javascript
// 接受对话框
mcp_playwright_browser_handle_dialog({
  accept: true
})

// 拒绝对话框
mcp_playwright_browser_handle_dialog({
  accept: false
})

// 接受并输入提示文本（仅 prompt 类型）
mcp_playwright_browser_handle_dialog({
  accept: true,
  promptText: "输入的文本"
})
```

### 鼠标交互

#### mcp_playwright_browser_hover
悬停在元素上（触发 hover 效果或下拉菜单）。

```javascript
mcp_playwright_browser_hover({
  target: "@e301"
})
```

#### mcp_playwright_browser_drag
拖拽操作。

```javascript
mcp_playwright_browser_drag({
  startTarget: "@e401",
  endTarget: "@e402"
})
```

### 调试与监控

#### mcp_playwright_browser_take_screenshot
截图用于视觉调试。注意：截图不能用于自动化操作，应使用 snapshot。

```javascript
// 截取整个页面
mcp_playwright_browser_take_screenshot()

// 截取完整可滚动页面
mcp_playwright_browser_take_screenshot({
  fullPage: true
})

// 截取特定元素
mcp_playwright_browser_take_screenshot({
  target: "@e501"
})

// 指定文件名和格式
mcp_playwright_browser_take_screenshot({
  filename: "debug.png",
  type: "png"
})
```

#### mcp_playwright_browser_console_messages
获取浏览器控制台消息（console.log、error、warning 等）。

```javascript
// 获取 info 及以上级别的消息
mcp_playwright_browser_console_messages({
  level: "info"
})

// 获取所有消息
mcp_playwright_browser_console_messages({
  all: true
})

// 保存到文件
mcp_playwright_browser_console_messages({
  filename: "console.log"
})
```

#### mcp_playwright_browser_network_requests
获取网络请求列表。

```javascript
// 获取所有网络请求（不包括静态资源）
mcp_playwright_browser_network_requests()

// 包括静态资源（图片、字体、脚本等）
mcp_playwright_browser_network_requests({
  static: true
})

// 包含请求体和响应头
mcp_playwright_browser_network_requests({
  requestBody: true,
  requestHeaders: true
})

// 过滤特定 URL 模式
mcp_playwright_browser_network_requests({
  filter: "/api/.*user"
})

// 保存到文件
mcp_playwright_browser_network_requests({
  filename: "requests.json"
})
```

### 高级功能

#### mcp_playwright_browser_evaluate
在页面上下文中执行 JavaScript。

```javascript
// 执行简单表达式
mcp_playwright_browser_evaluate({
  target: "@e601",
  function: "element => element.innerText"
})

// 滚动页面
mcp_playwright_browser_evaluate({
  function: "() => window.scrollTo(0, 500)"
})

// 获取页面标题
mcp_playwright_browser_evaluate({
  function: "() => document.title"
})

// 保存结果到文件
mcp_playwright_browser_evaluate({
  function: "() => document.title",
  filename: "title.txt"
})
```

#### mcp_playwright_browser_run_code
运行完整的 Playwright 代码片段。适合复杂操作。

```javascript
// 直接使用 Playwright API
mcp_playwright_browser_run_code({
  code: "async (page) => { await page.getByRole('button', { name: 'Submit' }).click(); return await page.title(); }"
})

// 从文件加载代码
mcp_playwright_browser_run_code({
  filename: "./playwright-script.js"
})
```

#### mcp_playwright_browser_resize
调整浏览器窗口大小。

```javascript
mcp_playwright_browser_resize({
  width: 1920,
  height: 1080
})
```

## A8 系统典型工作流程

### 1. 登录 A8 系统

```javascript
// 打开登录页面
mcp_playwright_browser_navigate({
  url: "http://120.35.0.67:28101/seeyon/main.do?method=main"
})

// 获取页面快照
mcp_playwright_browser_snapshot()

// 填写登录表单
mcp_playwright_browser_fill_form({
  fields: [
    { target: "@e101", text: "1003854" },
    { target: "@e102", text: "zxjqwe@621" }
  ]
})

// 点击登录按钮
mcp_playwright_browser_click({
  target: "@e103"
})

// 等待页面加载后获取快照
mcp_playwright_browser_snapshot()
```

### 2. 查找工单

```javascript
// 进入待办中心
mcp_playwright_browser_click({
  target: "@e200"  // 待办中心入口
})

// 获取待办列表快照
mcp_playwright_browser_snapshot()

// 使用搜索框查找工单
mcp_playwright_browser_type({
  target: "@e201",
  text: "KFXQ-CX-2026032700129"
})

mcp_playwright_browser_press_key({
  key: "Enter"
})

// 点击工单
mcp_playwright_browser_click({
  target: "@e300"
})
```

### 3. 提取工单信息

```javascript
// 获取工单详情快照
mcp_playwright_browser_snapshot()

// 如需要，截取完整页面用于人工核对
mcp_playwright_browser_take_screenshot({
  fullPage: true,
  filename: "workorder-detail.png"
})
```

### 4. 处理文件上传

```javascript
// 如果工单需要上传附件
mcp_playwright_browser_click({
  target: "@e400"  // 文件上传按钮
})

// 等待文件选择器出现
// 然后上传文件
mcp_playwright_browser_file_upload({
  paths: ["/tmp/attachment.pdf"]
})
```

### 5. 处理弹窗

```javascript
// 如果出现确认对话框
mcp_playwright_browser_handle_dialog({
  accept: true
})

// 如果需要输入
mcp_playwright_browser_handle_dialog({
  accept: true,
  promptText: "备注信息"
})
```

## 元素引用（ref）

Playwright MCP 使用页面快照中的元素引用（如 `@e1`、`@e2`）来精确定位元素。

- 获取快照后，每个交互元素都会被分配唯一的 ref ID
- ref ID 在页面刷新后会改变，需要重新获取快照
- 也可以使用 CSS 选择器或 XPath 作为 target

## 调试技巧

### 保存快照到文件

```javascript
mcp_playwright_browser_snapshot({
  filename: "page-snapshot.md"
})
```

### 截图调试

```javascript
mcp_playwright_browser_take_screenshot({
  fullPage: true,
  filename: "debug-screenshot.png"
})
```

### 查看控制台错误

```javascript
mcp_playwright_browser_console_messages({
  level: "error",
  all: true
})
```

### 查看网络请求

```javascript
mcp_playwright_browser_network_requests({
  static: true,
  requestBody: true,
  requestHeaders: true
})
```

## 注意事项

1. **隔离浏览器**：使用 MCP 服务器时，每次会话会创建独立的浏览器实例，不会影响用户正在使用的浏览器
2. **页面加载等待**：导航后等待页面完全加载再获取快照
3. **ref 失效**：页面刷新或导航后，之前的 ref ID 会失效，需要重新获取快照
4. **iframe 处理**：A8 系统常用 iframe，可能需要逐级进入
5. **快照优于截图**：自动化操作始终使用 snapshot，截图仅用于调试
6. **网络限制**：某些网站可能有反爬机制，可能需要设置延迟或调整用户代理

## 与旧 browser.md 的差异

| 旧 API | 新 API | 说明 |
|---------|--------|------|
| `browser(action="navigate", ...)` | `mcp_playwright_browser_navigate()` | 功能相同 |
| `browser(action="snapshot", ...)` | `mcp_playwright_browser_snapshot()` | 功能相同 |
| `browser(action="act", kind="click", ...)` | `mcp_playwright_browser_click()` | 独立函数 |
| `browser(action="act", kind="fill", ...)` | `mcp_playwright_browser_fill_form()` | 批量填写更高效 |
| - | `mcp_playwright_browser_select_option()` | **新增**：下拉选择 |
| - | `mcp_playwright_browser_file_upload()` | **新增**：文件上传 |
| - | `mcp_playwright_browser_handle_dialog()` | **新增**：对话框处理 |
| - | `mcp_playwright_browser_drag()` | **新增**：拖拽 |
| - | `mcp_playwright_browser_hover()` | **新增**：悬停 |
| - | `mcp_playwright_browser_network_requests()` | **新增**：网络监控 |
| `profile` 参数 | 无需 | MCP 自动管理会话 |

## 资源

- Playwright 官方文档：https://playwright.dev
- Playwright MCP GitHub：https://github.com/microsoft/playwright-mcp
- MCP 协议规范：https://modelcontextprotocol.io
