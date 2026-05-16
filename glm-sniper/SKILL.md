---
name: glm-sniper
description: "智谱 GLM Coding Plan 自动抢购工具 — Agent 在每天 10:00 自动执行 Node.js 脚本抢购套餐，无需浏览器工具"
---

# GLM 狙击手

> 自动抢购智谱 GLM Coding Plan。**直接执行 Node.js 脚本**，不走浏览器工具，最可靠。

## 抢购脚本

**路径：** `/home/ubuntu/glm-sniper.js`

**用法：**
```bash
node /home/ubuntu/glm-sniper.js              # 交互模式
node /home/ubuntu/glm-sniper.js check       # 登录 + 检查状态
node /home.ubuntu/glm-sniper.js buy          # 完整抢购流程
```

**playwright-core 路径：**
```
/home/ubuntu/.hermes/hermes-agent/node_modules/playwright-core
```

## 完整购买流程（分析）

1. **登录** → 账号密码登录 `https://open.bigmodel.cn/login`
2. **进入购买页** → `https://open.bigmodel.cn/glm-coding`
3. **点击"即刻订阅"** → 弹出订阅对话框
4. **点击"继续订阅"**（如果有）
5. **选择 Lite 套餐**（¥49/月）
6. **点击"立即支付"** → 跳转支付页

## 设置定时抢购

用 cron job：每天 09:50 触发（比 10:00 提前 10 分钟）：

```
cronjob(action="create", name="GLM-狙击手-Lite", schedule="50 9 * * *")
prompt: 见下方
```

## cron 任务提示词

创建时指定 `deliver: origin`，确保结果汇报给用户：

```
[任务：智谱 GLM Coding Plan 抢购]

执行以下步骤：

1. 用 Node.js 执行完整抢购流程：
   /home/ubuntu/.hermes/hermes-agent/node_modules/.bin/node /home/ubuntu/glm-sniper.js buy

2. 抢购脚本会：
   - 自动读取 /tmp/glm-session.json 的登录状态
   - 如果无 session 则自动用账号密码登录
   - 自动执行 subscribe -> continue -> select plan -> pay 流程

3. 截图保存到 /tmp/glm-buy-result.png

4. 把执行结果（成功/失败/状态）汇报给用户

注意：
- 禁止使用 browser_navigate 或任何 browser_ 开头的工具（会超时导致任务失败）
- 只用 terminal 执行 node 命令
```

## 当前状态

- **账号：** 19903348351
- **目标套餐：** Lite ¥49/月（折后￥44.1）
- **下次补货：** 每天 10:00（明日 05月15日 10:00）
- **Session 文件：** `/tmp/glm-session.json`
- **Cron Job：** GLM-狙击手-Lite（每天 09:50）

## 浏览器工具说明

**重要：本 skill 不使用 browser_navigate 等浏览器工具。**

原因：内置 browser 工具依赖 agent-browser daemon，daemon 会卡死导致超时。

正确方案：直接用 Node.js 执行 playwright-core 脚本，绕过所有 daemon 通信层。

**已知陷阱：** 禁用内置 browser 后，`browser_navigate` 会走 Playwright MCP 路由，但 Playwright MCP 在 cron 任务中通信会超时（30s gateway 限制）。所有自动化必须在 terminal 中用 `node .../glm-sniper.js` 执行。

## 取消狙击

```
cronjob(action="list")  → 找到 GLM-狙击手-Lite 的 job_id
cronjob(action="remove", job_id="xxx")
```
