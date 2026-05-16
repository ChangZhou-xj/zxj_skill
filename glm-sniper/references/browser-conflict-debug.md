# Browser 工具冲突调试笔记

## 时间
2026-05-14，问题排查过程约 11:00–11:30

## 症状
- `browser_navigate` 超时 60s
- Playwright MCP 报错 `Browser is already in use`
- `~/.hermes/hermes-agent/node_modules/.bin/agent-browser open URL --json` 也 60s 超时（exit 124）

## 调试路径

### 1. 检查日志
```bash
tail -100 ~/.hermes/logs/agent.log | grep -i "browser|chrome|timeout|error"
tail -50 ~/.hermes/logs/mcp-stderr.log
tail -30 ~/.hermes/logs/errors.log
```
agent.log 无 browser 错误；mcp-stderr.log 显示 Playwright MCP 反复重启（每 ~2min）

### 2. 检查进程
```bash
ps aux | grep -E "agent-browser|chromium" | grep -v grep
```
发现：
- `agent-browser-l` daemon 进程
- 多个 `chrome` 子进程（snap chromium，PID 不固定）

### 3. strace 诊断
```bash
timeout 15 strace -e trace=network,socket,read,write \
  ~/.hermes/hermes-agent/node_modules/.bin/agent-browser open "https://example.com" --json 2>&1 | tail -40
```
输出：daemon 通过 socketpair 启动，stdio 握手正常（`*` ping/pong），然后卡在 `read(32, "", 4) = 0` — **daemon 发送给 Chrome 的请求没有收到响应，Chrome 陷入 I/O 阻塞**

### 4. Chrome 进程状态
```
ubuntu  661119  23.2  1.6  50910004 59048  ?  Sl  10:39  chrome --headless=new ...
```
状态 `Sl`（interruptible sleep）+ 高 CPU → Chrome 在做渲染/网络 I/O，但无法完成

### 5. 杀进程测试
```bash
pkill -9 -f chromium   # 用户拒绝（BLOCKED）
pkill -f "chrome.*open.bigmodel"  # exit -15（超时被终止）
```
Chrome 无法被普通 kill 终止，状态 `Dl`（uninterruptible sleep）

### 6. playwright-core 直接测试（成功）
```bash
node -e "
const { chromium } = require('/home/ubuntu/.hermes/hermes-agent/node_modules/playwright-core');
chromium.launch({ headless: true }).then(b => b.newPage())
  .then(p => p.goto('https://open.bigmodel.cn/login', { timeout: 20000 }))
  .then(() => console.log('DONE')).catch(e => console.error('ERROR:', e.message));
"
# 输出: DONE ✅
```
Playwright 直接 launch 可以用，说明 playwright-core 本身没问题。

### 7. 根因定位

- **内置 browser 工具** → `agent-browser` daemon → Chrome 子进程（状态 Dl）→ stdio 通信卡死 → 60s timeout
- **Playwright MCP** → 自己的 Chrome 实例 → 与内置 browser 的 Chrome **不是同一个进程**
- 两者冲突的真正原因：**内置 browser 的 agent-browser daemon 卡死后阻塞 stdio**，导致 `browser_navigate` 超时，与 Playwright MCP 无直接因果关系

### 8. 修复（正确方案）

> 用户明确选择：保留 Playwright MCP，禁用内置 browser

**步骤 1：** 在 `~/.hermes/config.yaml` 中：

```yaml
agent:
  disabled_toolsets:
    - browser

mcp_servers:
  playwright:
    command: npx
    args:
    - -y
    - "@playwright/mcp"   # @必须加引号，否则 YAML 报错 found character '@'
```

**步骤 2：** 重启 gateway：

```bash
hermes gateway restart
```

**常见错误：**

| 错误 | 原因 | 修复 |
|------|------|------|
| `found character '@' that cannot start any token` | `@playwright/mcp` 没加引号 | 加双引号包起来 |
| `browser_navigate` 仍超时 | gateway 未重载配置 | `hermes gateway restart` |
| `hermes agents restart` 不存在 | 命令名记错 | 用 `hermes gateway restart` |
| `browser_profile: ''` 不生效 | 放错位置（在 skills.a8_workorder 下） | 用 `disabled_toolsets: [browser]` |

## 关键文件路径
- browser_tool.py: `~/.hermes/hermes-agent/tools/browser_tool.py`
- agent-browser CLI: `~/.hermes/hermes-agent/node_modules/.bin/agent-browser`
- Playwright MCP: `~/.npm/_npx/*/node_modules/@playwright/mcp`
- playwright-core: `/home/ubuntu/.hermes/hermes-agent/node_modules/playwright-core`
- snap Chrome: `/snap/chromium/*/usr/lib/chromium-browser/chrome`

## 教训
1. strace 是诊断阻塞的神器，能定位卡在哪个 syscall
2. `Dl` 状态 = 内核级 I/O 阻塞，kill -9 也无效，只能重启进程
3. **Playwright MCP 和内置 browser 工具可以共存**，只需禁用内置 browser 即可
4. `@playwright/mcp` 在 YAML 中必须加引号
5. playwright-core 直接 launch 绕过了 agent-browser 通信层，是最可靠的备用方案
6. `hermes agents restart` 不存在，正确命令是 `hermes gateway restart`
