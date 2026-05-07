# Hermes Agent Skills 汇总指南

> 共 88 个技能，17 个分类。生成时间：2026-05-06

---

## 快速索引

| 分类 | 数量 | 核心技能 |
|------|------|---------|
| [autonomous-ai-agents](#autonomous-ai-agents) | 4 | claude-code, codex, opencode, hermes-agent |
| [creative](#creative) | 15 | baoyu-infographic, excalidraw, p5js, pixel-art, sketch |
| [data-science](#data-science) | 1 | jupyter-live-kernel |
| [devops](#devops) | 2 | jenkins-build, webhook-subscriptions |
| [email](#email) | 1 | himalaya |
| [gaming](#gaming) | 2 | minecraft-modpack-server, pokemon-player |
| [github](#github) | 5 | github-pr-workflow, github-code-review, github-issues |
| [mcp](#mcp) | 1 | native-mcp |
| [media](#media) | 5 | spotify, youtube-content, gif-search |
| [mlops](#mlops) | 12 | llama-cpp, serving-llms-vllm, fine-tuning-with-trl, unsloth |
| [note-taking](#note-taking) | 1 | obsidian |
| [productivity](#productivity) | 7 | tencent-docs, airtable, google-workspace, notion |
| [red-teaming](#red-teaming) | 1 | godmode |
| [research](#research) | 5 | arxiv, blogwatcher, llm-wiki, polymarket |
| [smart-home](#smart-home) | 1 | openhue |
| [social-media](#social-media) | 1 | xurl |
| [software-development](#software-development) | 10 | systematic-debugging, test-driven-development, writing-plans |
| [.archive](#archive) | 2 | mcp-migration, tencent-docs-workorder-sync（已归档） |

---

## autonomous-ai-agents
*把编码任务委托给 AI 编程工具*

| 技能 | 说明 |
|------|------|
| **claude-code** | Claude Code CLI — 代理编码（PR、功能） |
| **codex** | OpenAI Codex CLI — 代理编码（PR、功能） |
| **opencode** | OpenCode CLI — PR 代码审查 |
| **hermes-agent** | 配置/扩展 Hermes Agent 自身 |

**使用示例：**
```
用户: 用 Claude Code 帮我给这个项目加单元测试
→ skill: claude-code
→ 加载 skill 后自动调用 claude-code CLI 执行
```

---

## creative
*创意内容生成 — 图像、视频、动画、设计*

| 技能 | 说明 | 典型命令 |
|------|------|---------|
| **baoyu-infographic** | 信息图（21种布局×21种风格） | 加载后生成可视化信息图 |
| **excalidraw** | 手绘风格图表（架构/流程/时序） | 输出 Excalidraw JSON |
| **p5js** | p5.js 创意编程（艺术/着色器/3D） | 生成 HTML 动画 |
| **pixel-art** | 像素艺术（NES/Game Boy/PICO-8 调色盘） | 生成像素图 |
| **sketch** | HTML 草图（2-3个设计方案对比） | 生成可交互原型 |
| **ascii-art** | ASCII 艺术（pyfiglet/cowsay） | 终端文字装饰 |
| **ascii-video** | 彩色 ASCII 视频/GIF | 音视频转 ASCII |
| **architecture-diagram** | 深色主题架构图（SVG/云/基础设施） | 输出 HTML |
| **baoyu-comic** | 知识漫画（科普/传记/教程） | 生成漫画 |
| **claude-design** | 一次性 HTML 设计（landing/deck/prototype） | 输出 HTML |
| **comfyui** | ComfyUI 图像/视频/音频生成 | 管理节点和工作流 |
| **design-md** | Google DESIGN.md 令牌规范 | 生成设计规范文档 |
| **humanizer** | 去除 AI 味文本 | 让人写得更自然 |
| **ideation** | 约束创意生成项目 idea | 头脑风暴 |
| **manim-video** | Manim 数学动画（3Blue1Brown 风格） | 生成数学视频 |
| **popular-web-designs** | 54个真实设计系统（Stripe/Linear/Vercel） | HTML/CSS 参考 |
| **pretext** | @chenglou/pretext 创意浏览器演示 | 单文件 HTML 文字艺术 |
| **songwriting-and-ai-music** | 歌词创作 + Suno AI 音乐提示 | 生成歌曲 |
| **touchdesigner-mcp** | TouchDesigner 实时视觉控制 | 36个原生工具 |

**使用示例：**
```
用户: 生成一个系统架构图
→ skill: architecture-diagram
→ 输出深色主题 SVG 架构图 HTML

用户: 用 ASCII 风格展示这首诗
→ skill: ascii-art
→ 输出终端可渲染的 ASCII 艺术
```

---

## data-science

| 技能 | 说明 |
|------|------|
| **jupyter-live-kernel** | 实时 Jupyter 内核迭代编程（hamelnb） |

**使用示例：**
```
用户: 帮我分析这个 CSV 文件并画图
→ skill: jupyter-live-kernel
→ 启动 live kernel，执行 Python 数据分析代码
```

---

## devops
*CI/CD、流水线、发布管理*

| 技能 | 说明 | 关键能力 |
|------|------|---------|
| **jenkins-build** | Jenkins 构建打包流水线 | 触发构建、下载制品、解析日志 |
| **webhook-subscriptions** | Webhook 订阅 — 事件驱动 agent 运行 | 自动化触发 |

---

### jenkins-build — 使用详解

#### 环境变量（已配置）
```bash
JENKINS_URL="http://223.223.178.68:2004/jenkins-122"
JENKINS_USER="develop"
JENKINS_TOKEN="dev1234"
```

#### 场景一：通用构建（任何 Job）

```bash
# 1. 触发无参数构建
curl -s -X POST \
  "$JENKINS_URL/job/JOB_NAME/build" \
  -u "$JENKINS_USER:$JENKINS_TOKEN"

# 2. 触发带参数构建
curl -s -X POST \
  "$JENKINS_URL/job/JOB_NAME/buildWithParameters" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data "BRANCH=develop&VERSION=1.2.3"

# 3. 查询最新构建状态
curl -s "$JENKINS_URL/job/JOB_NAME/lastBuild/api/json" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | jq '{number, result, duration}'

# 4. 查询队列（看构建是否在排队）
curl -s "$JENKINS_URL/queue/api/json" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | jq '.items[] | {job: .task.name, queueId, why}'

# 5. 轮询直到完成（最多 10 分钟）
BUILD_NUM=$(curl -s "$JENKINS_URL/job/JOB_NAME/lastBuild/buildNumber" \
  -u "$JENKINS_USER:$JENKINS_TOKEN")
for i in $(seq 1 60); do
  RESULT=$(curl -s "$JENKINS_URL/job/JOB_NAME/$BUILD_NUM/api/json" \
    -u "$JENKINS_USER:$JENKINS_TOKEN" | jq -r '.result')
  echo "[$i] #$BUILD_NUM: ${RESULT:-RUNNING}"
  case "$RESULT" in
    SUCCESS|FAILURE|ABORTED|UNSTABLE) break ;;
  esac
  sleep 10
done

# 6. 下载制品
curl -s -L -o /tmp/artifact.zip \
  "$JENKINS_URL/job/JOB_NAME/$BUILD_NUM/artifact/*zip*/**" \
  -u "$JENKINS_USER:$JENKINS_TOKEN"

# 7. 获取构建日志
curl -s "$JENKINS_URL/job/JOB_NAME/$BUILD_NUM/consoleText" \
  -u "$JENKINS_USER:$JENKINS_TOKEN" | tail -50
```

#### 场景二：前端 orange-aliyun 打包（实际业务）

```
Job 路径: job/web/job/orange-aliyun
必填参数: build_type=vOrange, version_file, options=update_code,npm_build,package,orange_patch
```

**version_file 格式（3列，空格分隔）：**
```
日期时间(YYYYMMDDHHMMSS)  项目名  分支名
```

示例：
```
20250825172125 orange Release_20250830-5eccbc93
20260304104815 home Feature_20260228_tianJingDaXueZhiHuiCaiWu-b1449c99
20260328165400 pex Feature_20250430_tianJin-HEAD
```

**完整流程：**
```bash
# Step 1: 写 version_file
cat > /tmp/version_file.txt << 'EOF'
20250825172125 orange Release_20250830-5eccbc93
EOF

# Step 2: 认证（登录获取 cookie+crumb，cookie要单独存文件）
rm -f /tmp/jenkinsCookies.txt
curl -s -X POST "$JENKINS_URL/j_spring_security_check" \
  -d "j_username=develop&j_password=dev1234&from=%2F" \
  -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -L > /dev/null

CRUMB=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" \
  -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" | python3 -c \
  "import sys,json; print(json.load(sys.stdin).get('crumb',''))")

# Step 3: 触发构建
curl -s -X POST \
  "$JENKINS_URL/job/web/job/orange-aliyun/buildWithParameters" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" -H "Jenkins-Crumb: $CRUMB" \
  -F "build_type=vOrange" \
  -F "options=update_code,npm_build,package,orange_patch" \
  -F "version_file=@/tmp/version_file.txt"
# 返回 302/201 表示成功入队

# Step 4: 获取构建号（稍等几秒再查）
sleep 5
BUILD_NUM=$(curl -s \
  "$JENKINS_URL/job/web/job/orange-aliyun/lastBuild/buildNumber" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234")
echo "构建号: #$BUILD_NUM"

# Step 5: 轮询监控（用户说"我的构建是 #30191"后，用这个号直接查）
for i in $(seq 1 60); do
  RESULT=$(curl -s \
    "$JENKINS_URL/job/web/job/orange-aliyun/30191/api/json" \
    -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
    -u "develop:dev1234" | python3 -c \
    "import sys,json; print(json.load(sys.stdin).get('result','RUNNING'))" 2>/dev/null)
  echo "[$i] #30191: ${RESULT}"
  if echo "$RESULT" | grep -qE "SUCCESS|FAILURE|ABORTED|UNSTABLE"; then break; fi
  sleep 15
done

# Step 6: 从日志提取产物
curl -s "$JENKINS_URL/job/web/job/orange-aliyun/30191/consoleText" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" | grep -E "vOrange_|orange_.*zip|外网下载地址"
# 典型产物:
#   vOrange_20260506150807
#   orange_20260506150807.zip
#   外网下载地址: http://223.223.178.68:2004/orange-file/orange_20260506150807.zip
```

#### 场景三：orange-patch 补丁包构建（依赖 Step1）

```
Job 路径: job/web/job/orange-patch
必填参数: orange_package（从Step1日志拿）, orange_module（需询问用户）
orange_module 可选: pex / home / orange 等
```

```bash
# Step 1: 认证（cookie可能已过期，需要重新登录）
rm -f /tmp/jenkinsCookies.txt
curl -s -X POST "$JENKINS_URL/j_spring_security_check" \
  -d "j_username=develop&j_password=dev1234&from=%2F" \
  -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt -L > /dev/null

CRUMB=$(curl -s "$JENKINS_URL/crumbIssuer/api/json" \
  -c /tmp/jenkinsCookies.txt -b /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" | python3 -c \
  "import sys,json; print(json.load(sys.stdin).get('crumb',''))")

# Step 2: 触发（orange_module 必须先问用户确认）
curl -s -X POST \
  "$JENKINS_URL/job/web/job/orange-patch/buildWithParameters" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" -H "Jenkins-Crumb: $CRUMB" \
  -F "orange_package=orange-patch-20260506150807.zip" \
  -F "orange_module=pex"

# Step 3: 产物获取
curl -s "$JENKINS_URL/job/web/job/orange-patch/BUILD_NUM/consoleText" \
  -b /tmp/jenkinsCookies.txt -c /tmp/jenkinsCookies.txt \
  -u "develop:dev1234" | grep -E "update_patch_|vOrange_|外网:"
# 典型产物:
#   文件名: update_patch_pex_20260506151923.zip
#   外网: http://223.223.178.68:2004/jenkins-orange-patch/update_patch_pex_20260506151923.zip
```

#### 场景四：Python 脚本（复杂工作流）

```python
import requests
import time

JENKINS_URL = "http://223.223.178.68:2004/jenkins-122"
AUTH = ("develop", "dev1234")

def trigger_build(job_name, parameters=None):
    url = f"{JENKINS_URL}/job/{job_name}/buildWithParameters"
    resp = requests.post(url, data=parameters, auth=AUTH, timeout=30)
    resp.raise_for_status()
    return resp.status_code  # 201=成功入队

def poll_build(job_name, build_number=None, interval=10, timeout=3600):
    if build_number is None:
        build_number = requests.get(
            f"{JENKINS_URL}/job/{job_name}/lastBuild/api/json",
            auth=AUTH, timeout=30
        ).json()["number"]
    start = time.time()
    while time.time() - start < timeout:
        info = requests.get(
            f"{JENKINS_URL}/job/{job_name}/{build_number}/api/json",
            auth=AUTH, timeout=30
        ).json()
        result = info.get("result")
        print(f"[{int(time.time()-start)}s] #{build_number}: {result}")
        if result is not None:
            return result
        time.sleep(interval)
    raise TimeoutError(f"构建 #{build_number} 超时")

def get_console(job_name, build_number):
    resp = requests.get(
        f"{JENKINS_URL}/job/{job_name}/{build_number}/consoleText",
        auth=AUTH, timeout=30
    )
    return resp.text

# 用例
trigger_build("job/web/job/orange-aliyun", {
    "build_type": "vOrange",
    "options": "update_code,npm_build,package,orange_patch"
})
result = poll_build("job/web/job/orange-aliyun")
print(f"构建结果: {result}")
```

#### 避坑提示

| 陷阱 | 解决 |
|------|------|
| 返回 403 CSRF | 先获取 crumb：`-H "Jenkins-Crumb: $CRUMB_VALUE"` |
| 参数化任务返回 400 | 用 `buildWithParameters` 而不是 `build` |
| 监控到别人的构建 | 用户给 build 号后直接查 `#号`，不用 `lastBuild` |
| 制品下载 302 重定向 | 加 `-L` 参数 |
| cookie 过期 | 每次构建前重新执行登录（`j_spring_security_check`） |

---

### webhook-subscriptions — 使用详解

#### 前置：启动 Webhook 服务

```bash
# 检查是否已启用
hermes webhook list

# 如果没启用，配置并启动
# 方法1: 修改 config.yaml
cat >> ~/.hermes/config.yaml << 'EOF'
platforms:
  webhook:
    enabled: true
    extra:
      host: "0.0.0.0"
      port: 8644
      secret: "your-strong-secret-here"
EOF

# 方法2: 环境变量
echo "WEBHOOK_ENABLED=true" >> ~/.hermes/.env
echo "WEBHOOK_PORT=8644" >> ~/.hermes/.env
echo "WEBHOOK_SECRET=your-strong-secret-here" >> ~/.hermes/.env

# 启动网关
hermes gateway run

# 验证运行
curl http://localhost:8644/health
# 返回 {"status": "ok"} 表示正常
```

#### 场景一：GitHub Issue 来时自动分类

```bash
hermes webhook subscribe github-issues \
  --events "issues" \
  --prompt "新 GitHub Issue #{issue.number}: {issue.title}
作者: {issue.user.login}
标签: {issue.labels[*].name}
正文:
{issue.body}
请进行分类（需要回复/已知的bug/功能需求）并给出处理建议。" \
  --deliver telegram \
  --deliver-chat-id "-100123456789"
# 返回 webhook URL 和 secret
# 在 GitHub Settings → Webhooks 添加，Payload URL 填返回的 URL，Secret 填返回的 secret
```

#### 场景二：GitHub PR 创建时自动代码审查

```bash
hermes webhook subscribe github-pr-review \
  --events "pull_request" \
  --prompt "PR #{pull_request.number}: {pull_request.title}
作者: {pull_request.user.login}
分支: {pull_request.head.ref} → {pull_request.base.ref}
正文:
{pull_request.body}" \
  --skills "github-code-review" \
  --deliver origin
```

#### 场景三：CI/CD 构建完成后推送通知

```bash
hermes webhook subscribe ci-notify \
  --events "build" \
  --prompt "构建 {object_attributes.status} | 项目: {project.name} | 分支: {object_attributes.ref}
提交: {commit.message}" \
  --deliver telegram \
  --deliver-chat-id "-100123456789"
```

#### 场景四：只转发通知（不启动 Agent，零 LLM 消耗）

```bash
hermes webhook subscribe ping-forward \
  --deliver telegram \
  --deliver-chat-id "-100123456789" \
  --deliver-only \
  --prompt "🔔 收到通知: {alert.name} | 等级: {alert.severity} | {alert.message}"
```

#### 查看和管理订阅

```bash
# 列出所有订阅
hermes webhook list

# 测试某个订阅
hermes webhook test github-issues --payload '{"key": "value"}'

# 删除订阅
hermes webhook remove github-issues
```

#### 常见问题排查

| 问题 | 检查方式 |
|------|---------|
| Webhook 没反应 | `curl http://localhost:8644/health` 确认服务运行 |
| GitHub 签名校验失败 | 确认 GitHub 传的 secret 和 `hermes webhook list` 的一致 |
| 外网无法访问 | 需内网穿透（ngrok/cloudflared），或确认防火墙/NAT 映射 |
| 消息没送到 | 检查 `hermes webhook list` 中订阅状态，查看网关日志 |


---

## email

| 技能 | 说明 |
|------|------|
| **himalaya** | Himalaya CLI — 终端 IMAP/SMTP 收发邮件 |

**使用示例：**
```
用户: 帮我查一下有没有来自 xxx 的邮件
→ skill: himalaya
→ 使用 himalaya search/display 命令操作邮件
```

---

## gaming

| 技能 | 说明 |
|------|------|
| **minecraft-modpack-server** | CurseForge/Modrinth 模组服务器托管 |
| **pokemon-player** | 模拟器 + RAM 读取玩 Pokemon |

**使用示例：**
```
用户: 架设一个 Minecraft 模组服务器
→ skill: minecraft-modpack-server
→ 加载后执行模组服搭建流程
```

---

## github
*GitHub 仓库、PR、Issues、代码审查*

| 技能 | 说明 | 关键能力 |
|------|------|---------|
| **github-pr-workflow** | PR 生命周期（分支→提交→打开→CI→合并） | 完整 PR 流程 |
| **github-code-review** | PR 审查 — diff + 行内评论 | gh/REST 评论 |
| **github-issues** | Issues 创建/分类/标签/分配 | gh issues |
| **github-repo-management** | 仓库克隆/创建/分叉/远程管理/releases | 仓库管理 |
| **codebase-inspection** | 代码库统计（LOC/语言/比率） | pygount 分析 |

**github-pr-workflow 使用示例：**
```bash
# 1. 创建分支
git checkout -b feat/add-login

# 2. 提交代码
git add src/auth.py
git commit -m "feat: add login endpoint"

# 3. 推送并创建 PR
git push -u origin HEAD
gh pr create --title "feat: add login" --body "## Summary..."

# 4. 监控 CI
gh pr checks --watch

# 5. 合并
gh pr merge --squash --delete-branch
```

---

## mcp

| 技能 | 说明 |
|------|------|
| **native-mcp** | MCP 客户端 — 连接 MCP 服务器、注册工具（stdio/HTTP） |

**使用示例：**
```
用户: 配置一个新的 MCP 服务器
→ skill: native-mcp
→ 按 skill 指引在 config.yaml 中注册 MCP 服务器
```

---

## media
*音乐、音频、YouTube、GIF*

| 技能 | 说明 |
|------|------|
| **spotify** | Spotify 播放/搜索/队列/播放列表管理 |
| **youtube-content** | YouTube 字幕→摘要/线程/博客 |
| **gif-search** | Tenor GIF 搜索下载（curl+jq） |
| **songsee** | 音频频谱/特征（mel/chroma/MFCC） |
| **heartmula** | Suno 类歌词生成 + AI 音乐提示 |

**使用示例：**
```
用户: 帮我找几个 GIF
→ skill: gif-search
→ curl Tenor API 搜索并下载 GIF

用户: 总结一下这个 YouTube 视频的内容
→ skill: youtube-content
→ 获取字幕并生成摘要
```

---

## mlops
*机器学习训练、推理、微调、部署*

| 技能 | 说明 | 场景 |
|------|------|------|
| **llama-cpp** | llama.cpp 本地 GGUF 推理 + HF Hub 模型发现 | 本地跑大模型 |
| **serving-llms-vllm** | vLLM 高吞吐推理服务（OpenAI API/量化） | 生产推理 |
| **fine-tuning-with-trl** | TRL 微调（SFT/DPO/PPO/GRPO/奖励模型） | RLHF 训练 |
| **unsloth** | Unsloth 加速 LoRA/QLoRA 微调（2-5x更快/更少显存） | 轻量微调 |
| **axolotl** | Axolotl YAML 微调配置（LoRA/DPO/GRPO） | 微调配置 |
| **dspy** | DSPy 声明式 LM 程序（自动优化提示/RAG） | RAG 系统 |
| **outlines** | 结构化 JSON/regex/Pydantic LLM 生成 | 可靠输出 |
| **obliteratus** | OBLITERATUS — 消除 LLM 拒绝（diff-in-means） | 对抗性测试 |
| **huggingface-hub** | HF hf CLI — 搜索/下载/上传模型数据集 | 模型管理 |
| **evaluating-llms-harness** | lm-eval-harness 基准测试（MMLU/GSM8K等） | 模型评估 |
| **weights-and-biases** | W&B 日志/实验追踪/模型注册/仪表盘 | 实验管理 |
| **segment-anything-model** | SAM 零样本图像分割 | 图像分割 |
| **audiocraft-audio-generation** | AudioCraft 音乐生成（MusicGen/AudioGen） | 音频生成 |

**使用示例：**
```
用户: 在本地跑一下这个模型
→ skill: llama-cpp
→ 使用 GGUF + llama.cpp 本地推理

用户: 帮我微调一个 7B 模型
→ skill: unsloth
→ 生成加速微调配置，显存占用更少

用户: 部署一个 vLLM 服务
→ skill: serving-llms-vllm
→ 完整的 vLLM 部署流程
```

---

## note-taking

| 技能 | 说明 |
|------|------|
| **obsidian** | Obsidian vault 笔记读取/搜索/创建 |

**使用示例：**
```
用户: 帮我搜一下 Obsidian 笔记库里有啥关于 x 的内容
→ skill: obsidian
→ 搜索 vault 并返回相关笔记
```

---

## productivity
*文档、表格、演示、PDF、OCR*

| 技能 | 说明 | 关键能力 |
|------|------|---------|
| **tencent-docs** | 腾讯文档（首选！）— 创建/编辑所有类型 | 文档/Excel/PPT/思维导图/流程图 |
| **airtable** | Airtable REST API — 记录 CRUD/过滤/upsert | 结构化数据库 |
| **google-workspace** | Gmail/Calendar/Drive/Docs/Sheets | 谷歌全家桶 |
| **notion** | Notion API — pages/databases/blocks/search | 笔记数据库 |
| **maps** | OpenStreetMap/OSRM — 地理编码/POI/路线 | 地图服务 |
| **nano-pdf** | PDF 文本编辑（自然语言提示） | 修改 PDF |
| **ocr-and-documents** | PDF/扫描件文字提取（pymupdf/marker-pdf） | 文字识别 |
| **powerpoint** | PPTX 创建/读取/编辑（幻灯片/备注/模板） | 演示文稿 |

**tencent-docs 使用示例：**
```bash
# 1. 列出可用工具
mcporter list tencent-docs

# 2. 创建智能文档（首选）
mcporter call "tencent-docs" "create_smartcanvas_by_mdx" --args '{
  "title": "报告标题",
  "mdx": "# 报告内容..."
}'

# 3. 搜索已有文档
mcporter call "tencent-docs" "manage.search_file" --args '{"keyword": "关键词"}'

# 4. 读取文档内容
mcporter call "tencent-docs" "get_content" --args '{"file_id": "xxx"}'
```

**tencent-docs 场景路由：**
| 场景 | 类型 | 工具 |
|------|------|------|
| 报告/笔记/文章 | smartcanvas | `create_smartcanvas_by_mdx` |
| 结构化数据管理 | smartsheet | `smartsheet.*` 系列 |
| Excel 计算/筛选 | sheet | `sheet.*` + `sheetengine` |
| Word 专业文档 | docengine | `create_with_markdown` |
| PPT 演示文稿 | slide | `create_slide` |
| 知识整理 | mind/flowchart | `create_mind_by_markdown` |
| 工单同步写入 | — | `workorder-sync.md` 工作流 |

---

## red-teaming

| 技能 | 说明 |
|------|------|
| **godmode** | LLM 越狱 — Parseltongue/GODMODE/ULTRAPLUS |

---

## research
*学术论文、博客监控、预测市场*

| 技能 | 说明 |
|------|------|
| **arxiv** | arXiv 论文搜索（关键词/作者/类别/ID） |
| **blogwatcher** | RSS/Atom 博客监控（blogwatcher-cli） |
| **llm-wiki** | Karpathy LLM Wiki — 交互式 Markdown 知识库 |
| **polymarket** | Polymarket 预测市场查询（价格/订单簿/历史） |
| **research-paper-writing** | ML 论文写作（NeurIPS/ICML/ICLR） |

**使用示例：**
```
用户: 搜一下最近有哪些 RAG 相关的 arXiv 论文
→ skill: arxiv
→ 按关键词搜索论文并返回摘要

用户: 帮我查一下某个预测市场的当前价格
→ skill: polymarket
→ 查询 markets/prices/orderbook
```

---

## smart-home

| 技能 | 说明 |
|------|------|
| **openhue** | Philips Hue 灯光/场景/房间控制（OpenHue CLI） |

**使用示例：**
```
用户: 把客厅灯调成暖白色
→ skill: openhue
→ 使用 openhue 命令控制灯光
```

---

## social-media

| 技能 | 说明 |
|------|------|
| **xurl** | X/Twitter — 发推/搜索/DM/媒体/v2 API |

**使用示例：**
```
用户: 帮我发一条推文
→ skill: xurl
→ 使用 xurl post 命令发送
```

---

## software-development
*软件开发流程 — 调试/计划/TDD/代码审查*

| 技能 | 说明 | 何时使用 |
|------|------|---------|
| **systematic-debugging** | 四阶段根因调试（理解→定位→验证→修复） | 任何 bug |
| **test-driven-development** | 红-绿-重构 TDD 流程 | 实现功能/bugfix 前 |
| **writing-plans** | 写实现计划（小型任务/路径/代码） | 复杂任务拆解 |
| **plan** | Plan 模式 — 写 Markdown 计划到 .hermes/plans/（不执行） | 需要先规划 |
| **spike** | 一次性实验验证想法 | 动手前先试探 |
| **subagent-driven-development** | 子 agent 执行计划（二阶段审查） | 多 agent 协作 |
| **requesting-code-review** | 提交前审查（安全扫描/质量门/自动修复） | PR 前 |
| **python-debugpy** | Python 调试（pdb REPL + debugpy DAP） | Python 调试 |
| **node-inspect-debugger** | Node.js 调试（--inspect + Chrome DevTools Protocol） | JS 调试 |
| **hermes-agent-skill-authoring** | 编写 SKILL.md（frontmatter/验证器/结构） | 创建新 skill |
| **debugging-hermes-tui-commands** | Hermes TUI 斜杠命令调试 | TUI 调试 |

**使用示例：**
```
用户: 这个接口报 500 错误，帮我排查
→ skill: systematic-debugging
→ 四阶段：理解现象→定位根因→验证假设→修复

用户: 帮我写一个用户认证模块
→ skill: test-driven-development
→ 先写测试用例，再实现代码（TDD 流程）

用户: 帮我规划一下这个功能的实现步骤
→ skill: writing-plans
→ 拆解为可执行的小任务，写出实施路径
```

---

## .archive（已归档）

| 技能 | 说明 |
|------|------|
| **mcp-migration** | 迁移 skill 到新 MCP 服务器 |
| **tencent-docs-workorder-sync** | 腾讯文档工单同步写入（周兴杰→代码复核表） |

> 这些技能已归档，可能不再维护。

---

## 按使用频率分类

### 🔴 高频（几乎每天用）
| 技能 | 用途 |
|------|------|
| **jenkins-build** | 前端打包、流水线触发 |
| **tencent-docs** | 腾讯文档读写 |
| **a8-workorder** | A8 工单处理（专属环境） |

### 🟡 中频（每周几次）
| 技能 | 用途 |
|------|------|
| **github-pr-workflow** | GitHub PR 全流程 |
| **systematic-debugging** | 排查问题 |
| **writing-plans** | 任务规划 |

### 🔵 低频（按需）
| 技能 | 用途 |
|------|------|
| mlops 系列 | 大模型训练/部署 |
| creative 系列 | 生成内容 |
| research 系列 | 论文/博客查询 |

---

## 如何使用 Skills

### 1. 自动触发
当你描述的任务匹配 skill 的关键词时，Agent 会自动加载对应 skill。

### 2. 手动指定
```
用户: 帮我处理一下这个 A8 工单，编号是 KFXQ-CX-2026032700129
→ Agent 自动识别 a8-workorder 并加载
```

### 3. 查看 skill 详情
```
skill_view(name="jenkins-build")     # 查看完整内容
skill_view(name="tencent-docs", file_path="references/workflows.md")  # 查看子文档
```

### 4. 查看所有可用 skills
```
skills_list()        # 列出所有
skills_list(category="devops")  # 只看某分类
```

---

*生成命令：skill_view / skills_list — 可随时刷新最新状态*
