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
- 登录凭据由用户或私有配置提供，**不要**写入公开仓库
- 浏览器自动化使用隔离 profile，避免影响用户日常浏览器会话
- 已具备腾讯文档相关能力，用于创建文档并设置分享权限

## A8 实际环境信息

| 项目 | 值 |
|------|-----|
| A8 系统地址 | `http://120.35.0.67:28101/seeyon` |
| 系统类型 | JavaScript-heavy SPA (单页应用)，需要浏览器自动化 |
| 登录凭据 | 用户私有配置 |

## 认证方式（DES-ECB）

A8 使用 DES-ECB 加密（经反复试验验证）：

1. 从首页 HTML 提取 `_SecuritySeed`（正则：`/_SecuritySeed\s*=\s*["\']*(-?\d+)["\']*/`）
2. 从首次响应 Set-Cookie 提取 JSESSIONID
3. DES-ECB 加密参数（**注意：模块名是 `pyDes`，不是 `pydes`**）：
   - Key：seed 前8字符，null-padded 至 8 字节：`seed[:8].encode('utf8').ljust(8, b'\x00')[:8]`
   - 模式：`pyDes.ECB`（常量，**不是函数**，直接用 `pyDes.ECB`）
   - Padding：`padmode=pyDes.PAD_PKCS5`
   - 输出：大写十六进制
4. POST 到 `main.do?method=login`，表单字段：`method=login`, `login_username=<user>`, `login_password=<hex>`, `encryption=true`

**关键注意**：`pyDes` 模块 pip 安装后 import 名是 `pyDes`（区分大小写），不是 `pydes`。

```python
import pyDes
key = str(seed)[:8].encode('utf8').ljust(8, b'\x00')[:8]
k = pyDes.des(key, pyDes.ECB, padmode=pyDes.PAD_PKCS5)
pwd_enc = k.encrypt(password.encode('utf8')).hex().upper()
```

## Session 维护问题

- **纯 HTTP session 访问受保护页面会被强制下线**（返回 `被迫下线，原因：与服务器失去连接`）
- A8 是 JavaScript-heavy SPA，服务器通过 JS 运行时检测浏览器环境
- **必须使用浏览器自动化（Playwright/Chromium）**，HTTP 请求无法维持会话
- Snap Chromium 可用：`/snap/bin/chromium`（版本 147.0.7727.55）
- browser_navigate 工具在此环境网络不通，需直接用 Playwright API 调用 snap Chromium

## 环境问题记录

| 问题 | 状态 |
|------|------|
| `import pydes` → No module | ⚠️ 正确名是 `import pyDes` |
| Playwright Chromium CDN 下载 | ⛔ 超时（网络问题） |
| Snap Chromium (`/snap/bin/chromium`) | ✅ 可用（需 `--no-sandbox`） |
| browser_navigate 工具 | ⛔ 超时（环境网络不通） |
| GitHub HTTPS 访问 | ⛔ 超时 |
| GitHub SSH 访问 | ✅ 正常 |

## 工作流

1. **DES 登录**（纯 Python/requests 可完成认证，但无法维持 session）：
   ```python
   import pyDes, re, http.client
   from urllib.parse import urlencode, urlparse
   # 获取 seed → DES-ECB 加密密码 → POST login → 获取 JSESSIONID
   # 注意：此时 session 几秒内会被强制下线
   ```
2. **浏览器自动化**（必须！用 Playwright + snap Chromium）：
   ```python
   from playwright.sync_api import sync_playwright
   with sync_playwright() as p:
       browser = p.chromium.launch(
           executable_path='/snap/bin/chromium',
           headless=True,
           args=['--no-sandbox', '--disable-dev-shm-usage', '--disable-gpu']
       )
       # 然后正常 navigate/fill/click 登录流程
   ```
3. 登录后通过 Playwright 提取工单数据
4. 使用 tencent-docs 能力生成报告
| 登录账号 | `<A8_USERNAME>` |
| 登录密码 | `<A8_PASSWORD>` |
| 浏览器 profile | `<ISOLATED_BROWSER_PROFILE>`（例如 `openclaw`） |
| 腾讯文档权限策略 | `policy=2`（所有人可查看，默认示例） |

## 工作流程（按顺序执行）

### 步骤 1：浏览器自动化

使用隔离浏览器 profile（例如 `openclaw`），不要复用用户自己的浏览器会话。

```javascript
// 1. 启动隔离浏览器
browser(action="start", profile="<ISOLATED_BROWSER_PROFILE>")

// 2. 打开登录页并抓取快照
browser(action="navigate", profile="<ISOLATED_BROWSER_PROFILE>", url="<A8_LOGIN_URL>")
browser(action="snapshot", profile="<ISOLATED_BROWSER_PROFILE>", compact=true)

// 3. 根据当前快照中的实际 ref 填写登录信息
browser(action="act", profile="<ISOLATED_BROWSER_PROFILE>", kind="fill",
  fields=[
    {"ref":"<USERNAME_REF>","text":"<A8_USERNAME>"},
    {"ref":"<PASSWORD_REF>","text":"<A8_PASSWORD>"}
  ])
browser(action="act", profile="<ISOLATED_BROWSER_PROFILE>", ref="<SUBMIT_REF>", kind="click")

// 4. 跳转完成后再次抓取快照
browser(action="snapshot", profile="<ISOLATED_BROWSER_PROFILE>", compact=true)
```

> 浏览器操作细节见 `references/browser.md`。

### 步骤 2：定位目标工单

在待办列表中查找工单（常见编号格式：`KFXQ-CX-YYYYMMDDNNNN`、`QXWT-CX-YYYYMMDDNNNN`）。

关键经验：
- `KFXQ` 类工单在很多 A8 部署中常见于“运维管理”等分类下，但实际分类名称以目标环境为准
- 快照中的标题文本通常比页面视觉更完整，应优先以快照为准
- 工单列表加载较慢，建议等待页面稳定后再操作

### 步骤 3：提取工单信息

从快照和截图提取：
- 工单编号、标题、类型
- 所属项目、产品名称、产品版本
- 发起人、发起部门、发起时间
- 当前状态、紧急程度、需求负责人
- 需求描述（背景、期望、效果）
- 处理记录 / 意见历史

### 步骤 4：生成腾讯文档分析报告

使用 tencent-docs 能力创建智能文档：

```bash
mcporter call "tencent-docs" "create_smartcanvas_by_mdx" --args '{
  "title": "<工单编号>-<简短标题（限36字）>",
  "content_format": "mdx",
  "mdx": "<MDX内容>"
}'
```

> MDX 内容需符合五章结构模板，详见 `references/mdx-template.md`。

### 步骤 5：设置文档权限

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

### 步骤 6：返回结果

返回给用户：
1. 腾讯文档链接
2. 工单摘要（编号、项目、需求、状态等）
3. 若存在不确定字段，明确标记需人工复核的部分

## 报告模板结构（五章）

1. **工单基本信息** — 两列表格
2. **需求详解** — 2.1 背景 / 2.2 用户期望 / 2.3 预期效果
3. **涉及业务场景及测试** — 3.1 单据 / 3.2 测试要点
4. **实现逻辑** — 4.1 前端 / 4.2 后端 / 4.3 数据字典 / 枚举
5. **参考信息** — 两列表格

## 已知限制

| 问题 | 方案 |
|------|------|
| A8 iframe 跨域无法直接读取内容 | 使用截图或快照提取关键信息，必要时提示人工补充 |
| 快照标题与页面视觉不一致 | 以快照文本为准，页面视觉截断不影响实际提取 |
| 浏览器可能干扰用户现有页面 | 必须使用隔离 browser profile |
| 腾讯文档创建后无法分享 | 立即调用 `manage.set_privilege` 设置权限 |
