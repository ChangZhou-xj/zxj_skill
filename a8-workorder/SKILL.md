---
name: a8-workorder
description: "Use when: 处理 A8 / 致远 OA 工单、根据工单编号或截图提取工单信息、生成腾讯文档分析报告、输出 MDX 报告草稿。关键词：A8、Seeyon、致远OA、工单、KFXQ、QXWT、腾讯文档、分析报告。"
---

# A8 工单分析报告自动化

## 适用场景

当用户需要处理 A8 / 致远 OA 工单，并将工单内容整理为标准化分析报告时使用本技能。

典型输入包括：
- 工单编号（如 `KFXQ-CX-*`、`QXWT-CX-*`）
- 工单截图或页面快照
- 用户要求生成腾讯文档分析报告

## 前置条件

- 运行环境可访问目标 A8 系统
- 登录凭据已配置在 SKILL.md（本地使用），公开仓库应使用占位符
- 已配置 Playwright MCP 工具（`mcp_playwright_*`），用于浏览器自动化
- 已具备腾讯文档相关能力，用于创建文档并设置分享权限
- 参见 `references/playwright-mcp.md` 了解完整的浏览器工具使用方法

## 关键配置（本地环境 - 需填写实际值）

| 项目 | 值 |
|------|-----|
| A8 系统地址 | `<A8_BASE_URL>`（例如：`http://120.35.0.67:28101/`） |
| 登录页地址 | `<A8_LOGIN_URL>`（例如：`http://120.35.0.67:28101/seeyon/main.do?method=main`） |
| 登录账号 | `<A8_USERNAME>`（例如：`1003854`） |
| 登录密码 | `<A8_PASSWORD>`（例如：`zxjqwe@621`） |
| 浏览器 profile | `playwright`（隔离 profile，由 Playwright MCP 自动管理） |
| 腾讯文档权限策略 | `policy=2`（所有人可查看） |

> **⚠️ 安全提示**：登录凭据（账号、密码）和 IP 地址为敏感信息，仅用于本地环境。如需提交到公共 Git 仓库，请使用占位符。本地配置请在私有环境或 `.env` 文件中管理。

## 工作流程（按顺序执行）

### 步骤 1：登录 A8 系统

使用 Playwright MCP 工具进行自动化操作：

1. 使用 `mcp_playwright_browser_navigate` 打开登录页面
2. 等待页面加载完成，使用 `mcp_playwright_browser_snapshot` 获取页面快照
3. 根据快照中的元素 ID 找到用户名输入框，使用 `mcp_playwright_browser_fill_form` 或 `mcp_playwright_browser_type` 填入账号
4. 找到密码输入框，填入密码
5. 点击登录按钮（使用 `mcp_playwright_browser_click`）
6. 等待登录成功，确认进入主页面

推荐使用批量表单填写以提高效率：
```javascript
mcp_playwright_browser_fill_form({
  fields: [
    { target: "@e101", text: "<A8_USERNAME>" },
    { target: "@e102", text: "<A8_PASSWORD>" }
  ]
})
mcp_playwright_browser_click({ target: "@e103" })  // 登录按钮
```

### 步骤 2：进入待办中心

1. 登录成功后，使用 `mcp_playwright_browser_snapshot` 获取主页快照
2. 在快照中找到并点击「待办中心」或「待办工作」入口
3. 确认进入待办列表页面

### 步骤 3：根据单号查找工单

1. 在待办列表中查找用户输入的工单单号（常见格式：`KFXQ-CX-YYYYMMDDNNNN`、`QXWT-CX-YYYYMMDDNNNN`）
2. 如果有搜索框，使用搜索功能查找对应单号
3. 点击对应工单进入工单详情页面
4. 使用 `mcp_playwright_browser_snapshot` 获取工单详情快照

搜索示例：
```javascript
// 在搜索框输入工单号
mcp_playwright_browser_type({ target: "@e201", text: "KFXQ-CX-2026032700129" })
mcp_playwright_browser_press_key({ key: "Enter" })
```

关键经验：
- `KFXQ-CX-*`、`QXWT-CX-*` 类工单在很多 A8 部署中常见于「待办工作」等分类下，但实际分类名称以目标环境为准
- 快照中的标题文本通常比页面视觉更完整，应优先以快照为准
- 工单列表加载较慢，建议等待页面稳定后再操作

### 步骤 4：提取工单信息

从快照和截图提取：
- 工单编号、标题、类型
- 所属项目、产品名称、产品版本
- 发起人、发起部门、发起时间
- 当前状态、紧急程度、需求负责人
- 需求描述（背景、期望、效果）
- 处理记录 / 意见历史

### 步骤 5：生成腾讯文档分析报告

使用 tencent-docs 能力创建智能文档：

```bash
mcporter call "tencent-docs" "create_smartcanvas_by_mdx" --args '{
  "title": "<工单编号>-<简短标题（限36字）>",
  "content_format": "mdx",
  "mdx": "<MDX内容>"
}'
```

> MDX 内容需符合五章结构模板，详见 `references/mdx-template.md`。

### 步骤 6：设置文档权限

**强制规则：生成文档后必须立即设置权限，否则默认可能无法分享。**

```bash
mcporter call "tencent-docs" "manage.set_privilege" --args '{
  "file_id": "<文档ID>",
  "policy": 2
}'
```

- `policy=2`：所有人可读
- `policy=3`：所有人可编辑

> 权限与文档操作细节见 `references/tencent-docs.md`。

### 步骤 7：返回结果

返回给用户：
1. 腾讯文档链接
2. 工单摘要（编号、项目、需求、状态等）
3. 若存在不确定字段，明确标记需人工复核的部分

## 报告模板结构（五章）

1. **工单基本信息** — 两列表格
2. **需求详解** — 2.1 背景 / 2.2 用户期望 / 2.3 预期效果
3. **涉及业务场景及测试** — 3.1 单据 / 3.2 测试要点
4. **实现逻辑** — 4.1 前端展示 / 4.2 逻辑处理
5. **参考信息** — 两列表格

## 已知限制

| 问题 | 方案 |
|------|------|
| A8 iframe 跨域无法直接读取内容 | 使用截图或快照提取关键信息，必要时提示人工补充 |
| 快照标题与页面视觉不一致 | 以快照文本为准，页面视觉截断不影响实际提取 |
| 浏览器可能干扰用户现有页面 | Playwright MCP 自动使用隔离浏览器实例 |
| 腾讯文档创建后无法分享 | 立即调用 `manage.set_privilege` 设置权限 |

## 依赖工具

- **Playwright MCP**：`mcp_playwright_browser_navigate`、`mcp_playwright_browser_snapshot`、`mcp_playwright_browser_click`、`mcp_playwright_browser_type`、`mcp_playwright_browser_fill_form`、`mcp_playwright_browser_press_key`、`mcp_playwright_browser_take_screenshot`
- **腾讯文档**：`mcporter call "tencent-docs" "create_smartcanvas_by_mdx"`、`mcporter call "tencent-docs" "manage.set_privilege"`

## 参考资料

- `references/playwright-mcp.md` - Playwright MCP 完整工具文档
- `references/playwright-mcp-quicktest.md` - 快速测试指南
- `references/MIGRATION.md` - 旧 API 迁移指南
- `references/mdx-template.md` - MDX 报告模板
- `references/tencent-docs.md` - 腾讯文档操作指南
