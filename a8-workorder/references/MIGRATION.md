# Playwright MCP 迁移指南

## 变更摘要

已从旧的 `browser()` 函数迁移到官方 Playwright MCP 工具。

### 文件变更

| 文件 | 状态 | 说明 |
|------|------|------|
| `references/browser.md` | 标记为废弃 | 添加废弃警告，保留作为参考 |
| `references/playwright-mcp.md` | 新建 | 完整的 Playwright MCP 文档 |
| `references/playwright-mcp-quicktest.md` | 新建 | 快速测试指南 |
| `SKILL.md` | 更新 | 工具名称从 `mcp_io_github_chr_*` 改为 `mcp_playwright_*` |
| `~/.hermes/config.yaml` | 更新 | 添加 Playwright MCP 服务器配置 |

### 配置变更

在 `~/.hermes/config.yaml` 中添加：

```yaml
mcp_servers:
  playwright:
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
```

## 新增功能

相比旧的 `browser()` 函数，Playwright MCP 提供了以下新功能：

### 表单增强
- `browser_fill_form` - 批量表单填写
- `browser_select_option` - 下拉选择
- `browser_file_upload` - 文件上传
- `browser_handle_dialog` - 处理对话框

### 鼠标交互
- `browser_hover` - 悬停
- `browser_drag` - 拖拽

### 调试功能
- `browser_network_requests` - 网络请求监控
- `browser_console_messages` - 控制台消息
- `browser_take_screenshot` - 截图
- `browser_evaluate` - 执行 JavaScript

### 高级功能
- `browser_run_code` - 运行完整 Playwright 代码
- `browser_resize` - 调整窗口大小

## 工具名称对照

| 旧 API | 新 API |
|--------|--------|
| `browser(action="navigate", ...)` | `mcp_playwright_browser_navigate()` |
| `browser(action="navigate_back", ...)` | `mcp_playwright_browser_navigate_back()` |
| `browser(action="snapshot", ...)` | `mcp_playwright_browser_snapshot()` |
| `browser(action="act", kind="click", ...)` | `mcp_playwright_browser_click()` |
| `browser(action="act", kind="press", ...)` | `mcp_playwright_browser_press_key()` |
| `browser(action="act", kind="fill", ...)` | `mcp_playwright_browser_fill_form()` 或 `browser_type()` |
| `browser(action="act", kind="evaluate", ...)` | `mcp_playwright_browser_evaluate()` |
| `browser(action="act", kind="wait", ...)` | 使用 JS 等待或显式等待 |

## 使用示例

### 旧代码

```javascript
// 导航
browser(action="navigate", profile="playwright", url="http://example.com")

// 快照
browser(action="snapshot", profile="playwright", compact=true)

// 填写表单
browser(action="act", profile="playwright", kind="fill",
  fields=[
    {ref: "@e1", text: "user"},
    {ref: "@e2", text: "pass"}
  ])

// 点击
browser(action="act", profile="playwright", ref: "@e3", kind="click")

// 按键
browser(action="act", profile="playwright", ref: "@e3", kind="press", key="Enter")
```

### 新代码

```javascript
// 导航
mcp_playwright_browser_navigate({ url: "http://example.com" })

// 快照
mcp_playwright_browser_snapshot()

// 批量表单填写
mcp_playwright_browser_fill_form({
  fields: [
    { target: "@e1", text: "user" },
    { target: "@e2", text: "pass" }
  ]
})

// 点击
mcp_playwright_browser_click({ target: "@e3" })

// 按键
mcp_playwright_browser_press_key({ key: "Enter" })
```

## A8 工单处理示例

```javascript
// 登录 A8
mcp_playwright_browser_navigate({
  url: "http://120.35.0.67:28101/seeyon/main.do?method=main"
})

mcp_playwright_browser_fill_form({
  fields: [
    { target: "@e101", text: "1003854" },
    { target: "@e102", text: "zxjqwe@621" }
  ]
})

mcp_playwright_browser_click({ target: "@e103" })

// 等待登录后进入待办中心
mcp_playwright_browser_snapshot()

mcp_playwright_browser_click({ target: "@e200" })

// 搜索工单
mcp_playwright_browser_type({
  target: "@e201",
  text: "KFXQ-CX-2026032700129"
})

mcp_playwright_browser_press_key({ key: "Enter" })

// 点击工单详情
mcp_playwright_browser_click({ target: "@e300" })

// 获取工单信息
mcp_playwright_browser_snapshot()
mcp_playwright_browser_take_screenshot({
  fullPage: true,
  filename: "workorder.png"
})
```

## 验证安装

运行以下命令验证 Playwright MCP 是否正确配置：

```bash
# 检查版本
npx -y @playwright/mcp@latest --version

# 查看帮助
npx -y @playwright/mcp@latest --help
```

## 验证工具可用性

在 Hermes 中运行：

```
请列出所有可用的 Playwright MCP 工具
```

预期输出应包含 `mcp_playwright_browser_navigate`、`mcp_playwright_browser_snapshot` 等工具。

## 故障排查

### 工具未出现

检查配置：
```bash
cat ~/.hermes/config.yaml | grep -A 3 mcp_servers
```

应包含：
```yaml
mcp_servers:
  playwright:
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
```

重启 Hermes Agent 以重新加载配置。

### 执行失败

检查 Playwright 安装：
```bash
npx -y @playwright/mcp@latest --version
```

如果失败，手动安装依赖：
```bash
npm install -g @playwright/mcp@latest
npx playwright install chromium
```

### 页面加载超时

增加超时配置：
```yaml
mcp_servers:
  playwright:
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
    timeout: 180
```

## 参考资料

- **完整文档**：`references/playwright-mcp.md`
- **快速测试**：`references/playwright-mcp-quicktest.md`
- **Playwright 官方文档**：https://playwright.dev
- **MCP 协议规范**：https://modelcontextprotocol.io
