# Playwright MCP 快速测试

## 测试 Playwright MCP 是否可用

运行以下命令检查 Playwright MCP 服务器是否正常工作：

```bash
# 检查版本
npx -y @playwright/mcp@latest --version

# 查看帮助
npx -y @playwright/mcp@latest --help
```

预期输出：
- 版本号（如 `Version 0.0.70`）
- 帮助信息显示可用选项

## 在 Hermes 中测试

Playwright MCP 工具会以 `mcp_playwright_*` 前缀自动注册。

### 测试 1：打开网页并获取快照

```
请打开 https://example.com 并获取页面快照
```

预期行为：
- 调用 `mcp_playwright_browser_navigate`
- 调用 `mcp_playwright_browser_snapshot`
- 返回页面结构和可交互元素列表

### 测试 2：表单填写测试

```
请打开 https://example.com/form 并填写表单：
- 用户名：test@example.com
- 密码：password123
```

预期行为：
- 导航到表单页面
- 获取快照找到输入框
- 使用 `mcp_playwright_browser_fill_form` 或 `mcp_playwright_browser_type` 填写字段

### 测试 3：按钮点击

```
请打开 https://example.com 并点击第一个按钮
```

预期行为：
- 导航到页面
- 获取快照
- 使用 `mcp_playwright_browser_click` 点击按钮

## 可用的 Playwright MCP 工具

检查以下工具是否可用：

| 工具名 | 功能 |
|--------|------|
| `mcp_playwright_browser_navigate` | 导航到 URL |
| `mcp_playwright_browser_navigate_back` | 返回上一页 |
| `mcp_playwright_browser_snapshot` | 获取页面快照 |
| `mcp_playwright_browser_click` | 点击元素 |
| `mcp_playwright_browser_type` | 输入文本 |
| `mcp_playwright_browser_press_key` | 按键盘键 |
| `mcp_playwright_browser_fill_form` | 批量表单填写 |
| `mcp_playwright_browser_select_option` | 下拉选择 |
| `mcp_playwright_browser_file_upload` | 文件上传 |
| `mcp_playwright_browser_handle_dialog` | 处理对话框 |
| `mcp_playwright_browser_hover` | 悬停 |
| `mcp_playwright_browser_drag` | 拖拽 |
| `mcp_playwright_browser_take_screenshot` | 截图 |
| `mcp_playwright_browser_console_messages` | 获取控制台消息 |
| `mcp_playwright_browser_network_requests` | 获取网络请求 |
| `mcp_playwright_browser_evaluate` | 执行 JavaScript |
| `mcp_playwright_browser_run_code` | 运行 Playwright 代码 |
| `mcp_playwright_browser_resize` | 调整窗口大小 |
| `mcp_playwright_browser_close` | 关闭浏览器 |

## 常见问题

### Q: 工具未出现

**原因**：Hermes 未重新加载 MCP 服务器配置。

**解决**：
```bash
# 重启 Hermes Agent
# 或如果使用后台模式，重启服务
```

### Q: 执行超时

**原因**：网络问题或页面加载缓慢。

**解决**：
- 检查网络连接
- 增加配置中的 timeout 值：
```yaml
mcp_servers:
  playwright:
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
    timeout: 180  # 增加到 180 秒
```

### Q: 页面快照为空

**原因**：页面尚未加载完成或使用 iframe。

**解决**：
- 在获取快照前等待一段时间
- 对于 iframe，需要逐级进入
- 检查控制台错误：`mcp_playwright_browser_console_messages({ level: "error" })`

## A8 系统测试示例

```
请访问 http://120.35.0.67:28101/seeyon/main.do?method=main 并截取登录页面
```

预期行为：
1. 调用 `mcp_playwright_browser_navigate` 打开 A8 登录页
2. 调用 `mcp_playwright_browser_snapshot` 获取页面快照
3. 返回可交互元素列表（用户名框、密码框、登录按钮等）
4. 可选择调用 `mcp_playwright_browser_take_screenshot` 保存截图
