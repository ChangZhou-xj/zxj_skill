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
- 已安装 Playwright（Python 版：`pip install playwright && playwright install chromium`）
- 已具备腾讯文档相关能力，用于创建文档并设置分享权限
- mcporter 已安装：`/home/ubuntu/.hermes/node/bin/mcporter`

## 关键配置

### 本地环境配置

在 `~/.hermes/config.yaml` 中配置：

```yaml
skills:
  a8_workorder:
    base_url: http://120.35.0.67:28101/
    login_url: http://120.35.0.67:28101/seeyon/main.do?method=main
    username_env: A8_USERNAME
    password_env: A8_PASSWORD
    doc_policy: 2
    browser_profile: playwright
```

在 `~/.hermes/.env` 中配置敏感信息：

```bash
# A8 登录凭据
A8_USERNAME=1003854
A8_PASSWORD=zxjqwe@621

# 腾讯文档权限策略（2=所有人可读，3=所有人可编辑）
TENCENT_DOC_POLICY=2
```

## 工作流程

### 步骤 1：登录 A8 系统

**重要：使用 Python Playwright（而非 MCP 工具）**

技能文档中的 `mcp_playwright_*` MCP 工具在此环境下不可用。实际使用 Python 脚本通过 Playwright API 直接操作浏览器。

登录页面 DOM 元素：
- 用户名输入框：`#login_username`（`id="login_username"`）
- 密码输入框：`#login_password1`（`id="login_password1"`）
- 登录按钮：`#login_button`（`id="login_button"`，type="button"）

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.set_default_timeout(30000)

    page.goto('http://120.35.0.67:28101/seeyon/main.do?method=main')
    page.wait_for_load_state('networkidle', timeout=20000)
    page.fill('#login_username', '1003854')
    page.fill('#login_password1', 'zxjqwe@621')
    page.click('#login_button')
    page.wait_for_load_state('networkidle', timeout=15000)
    print('LOGGED_IN')
```

### 步骤 2：查找工单

工单列表在主页的"运维管理"或"待办工作"区域。工单链接文字包含完整标题，格式如：

`运维工单-开发需求--KFXQ-CX-2026032700129--项目：xxx`

**关键经验：必须使用双击（dblclick）打开工单链接，单击不会跳转到详情页。**

```python
# 双击工单链接，打开新标签页
links = page.query_selector_all('a')
for l in links:
    text = l.inner_text() or ''
    if 'KFXQ-CX-2026032700129' in text:
        l.dblclick()
        break

page.wait_for_timeout(3000)
# 切换到新标签页
if len(browser.contexts[0].pages) > 1:
    page2 = browser.contexts[0].pages[-1]
    page2.wait_for_load_state('networkidle', timeout=15000)
```

### 步骤 3：提取工单详情

工单详情页URL格式：
```
http://120.35.0.67:28101/seeyon/collaboration/collaboration.do?method=summary&openFrom=listPending&affairId=xxx&showTab=true
```

详情页包含大量 iframe，实际工单表单在 `#zwIframe` 中：

```python
# 获取 zwIframe 的 src
iframe_url = page2.evaluate("""() => document.querySelector('#zwIframe').src""")
# 打印所有 iframe 信息
iframes = page2.query_selector_all('iframe')
for i, f in enumerate(iframes):
    info = f.evaluate("""fr => ({src: fr.src, id: fr.id, name: fr.name})""")
    print(f'IFRAME_{i}:' + json.dumps(info))
```

关键 iframe 说明：

| iframe id | 说明 |
|-----------|------|
| `zwIframe` | 工单表单主体内容，src 包含 `form/dist/index.html` |
| `thisIframe` | 审批意见历史区域 |
| `formRelativeDiv` | 表单关联信息 |

**提取表单内容**：直接导航到 zwIframe 的 src URL 加载表单页面：

```python
page3 = browser.contexts[0].pages[0]
page3.goto(iframe_url)
page3.wait_for_load_state('networkidle', timeout=20000)
body = page3.inner_text('body')
print(body)  # 工单表单完整文本
```

### 步骤 4：提取审批意见历史

在详情页（URL 含 `collaboration.do?method=summary`）获取审批意见历史：

```python
# 查找"处理人意见区"段落
opinion_area = page2.inner_text('body')
idx = opinion_area.find('处理人意见区')
if idx >= 0:
    print(opinion_area[idx:idx+3000])
```

### 步骤 5：生成腾讯文档分析报告

使用 mcporter 创建智能文档：

```bash
/home/ubuntu/.hermes/node/bin/mcporter call "tencent-docs" "create_smartcanvas_by_mdx" --args '{
  "title": "KFXQ-CX-2026032700129-简短描述",
  "mdx": "---..."
}'
```

**⚠️ 标题限制：最多 36 字符。** 工单编号 `KFXQ-CX-2026032700129` 约 24 字符，留给描述的字符有限。超长报错：`code:400001, msg:title length exceeds 36 characters`

### 步骤 6：设置文档权限

创建后**立即**设置权限：

```bash
/home/ubuntu/.hermes/node/bin/mcporter call "tencent-docs" "manage.set_privilege" --args '{
  "file_id": "<返回的file_id>",
  "policy": 2
}'
```

- `policy=2`：所有人可读
- `policy=3`：所有人可编辑

### 步骤 7：返回结果

返回给用户：
1. 腾讯文档链接
2. 工单摘要（编号、项目、需求、状态等）
3. 若存在不确定字段，明确标记需人工复核的部分

## 报告模板结构（五章）

1. **工单基本信息** — 两列表格（工单编号、类型、所属项目、产品名称、产品版本、提出人、提出时间、当前节点状态、紧急程度、需求类型、运维工单号、支持人员、需求负责人、研发负责人、开发人员、希望/计划完成时间等）
2. **需求详解** — 2.1 背景说明 / 2.2 用户期望 / 2.3 预期效果
3. **涉及业务场景及测试** — 3.1 涉及单据（表格）/ 3.2 测试要点（列表）
4. **实现逻辑** — 4.1 前端改动 / 4.2 逻辑处理 / 4.3 数据字典（均标注"待开发补充"）
5. **参考信息** — 两列表格（系统平台、登录地址、操作账号、报告生成时间、需求方案、处理意见）

MDX 模板详见 `references/mdx-template.md`。

## 已知限制与避坑

| 问题 | 避坑方案 |
|------|---------|
| `mcp_playwright_*` 工具不可用（超时） | 使用 Python `playwright.sync_api` 直接编写脚本 |
| 单击工单链接 URL 不变化 | **必须双击（dblclick）**打开新标签页 |
| 工单详情 iframe 跨域无法直接读 | 从 `zwIframe` 的 `src` 属性提取表单页面 URL，直接导航加载 |
| 腾讯文档标题超 36 字符报错 | 工单编号+简短描述，总长 ≤ 36 |
| 审批意见历史在主页正文而非表单 iframe | 在 `collaboration.do?method=summary` 页面提取 |
| `mcporter` 命令找不到 | 使用绝对路径：`/home/ubuntu/.hermes/node/bin/mcporter` |

## 依赖

- **Playwright**（Python）：`pip install playwright && playwright install chromium`
- **mcporter**：`/home/ubuntu/.hermes/node/bin/mcporter`
- **腾讯文档 MCP**：tencent-docs skill

## 参考资料

- `references/mdx-template.md` - MDX 报告模板
- `references/tencent-docs.md` - 腾讯文档操作指南
