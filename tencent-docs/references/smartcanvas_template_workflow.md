# SmartCanvas 模板文档编辑规范

> **何时使用**：编辑已有模板结构的腾讯文档（smartcanvas）时适用。
> 与 `smartcanvas/entry.md` 中的"工作流六"配合阅读。

---

## 核心原则

编辑已有模板文档前，**必须先判断文档结构**，区分两种场景：

| 场景 | 特征 | 正确操作 |
|------|------|---------|
| **模板文档**（有预设章节） | 文档含"一、二、三"或"1.1、1.2"等编号章节标题，章节下为空或只有占位符 | 用 `find` 定位各空标题，逐一 `INSERT_AFTER` 填充内容到对应章节 |
| **空白文档**（无预设结构） | 文档为空或仅有封面/标题，无章节框架 | 内容可直接 `INSERT_AFTER`（id为空）追加到末尾 |

---

## 判断流程

```
步骤 1：smartcanvas.read(file_id) 读取全文
步骤 2：扫描内容判断是否含编号章节标题（"一、", "1.1", "4.1" 等）
步骤 3A：发现有编号章节标题 → 进入「模板文档」流程
步骤 3B：未发现编号章节标题 → 进入「空白文档」流程
```

---

## 模板文档工作流（正确做法）

### 步骤 1：找到所有空章节标题的 Block ID

对每个待填充的章节标题执行 `smartcanvas.find`，或一次性读取后逐个查找：

```bash
mcporter call "tencent-docs" "smartcanvas.find" --args '{"file_id": "<file_id>", "query": "4.1 新增页面组件"}'
mcporter call "tencent-docs" "smartcanvas.find" --args '{"file_id": "<file_id>", "query": "4.2 接口说明"}'
mcporter call "tencent-docs" "smartcanvas.find" --args '{"file_id": "<file_id>", "query": "4.3 前端页面功能"}'
mcporter call "tencent-docs" "smartcanvas.find" --args '{"file_id": "<file_id>", "query": "4.4 逻辑处理"}'
```

记录每个标题的 `id`（Block ID）。

### 步骤 2：逐一填充各空章节

```bash
# 填入 4.1 内容（插到 4.1 标题后）
mcporter call "tencent-docs" "smartcanvas.edit" --args '{
  "action": "INSERT_AFTER",
  "content": "4.1 的实际内容...",
  "file_id": "<file_id>",
  "id": "<4.1标题的BlockID>"
}'

# 填入 4.2 内容（插到 4.2 标题后）
mcporter call "tencent-docs" "smartcanvas.edit" --args '{
  "action": "INSERT_AFTER",
  "content": "4.2 的实际内容...",
  "file_id": "<file_id>",
  "id": "<4.2标题的BlockID>"
}'

# ... 以此类推
```

### 步骤 3：检查重复内容（若之前已误追加到末尾）

如果之前已执行过追加操作导致文档末尾有重复内容，需先清理：

```bash
# 找到重复内容的 Block ID
mcporter call "tencent-docs" "smartcanvas.find" --args '{"file_id": "<file_id>", "query": "<重复的章节标题>"}'

# 删除重复的 Block
mcporter call "tencent-docs" "smartcanvas.edit" --args '{
  "action": "DELETE",
  "file_id": "<file_id>",
  "id": "<重复Block的ID>"
}'
```

---

## 反面案例

用户有一份模板文档，结构如下：
```
一、概述
二、...
三、...
四、实现逻辑
   4.1 新增页面组件       ← 空标题
   4.2 接口说明           ← 空标题
   4.3 前端页面功能       ← 空标题
   4.4 逻辑处理           ← 空标题
```

**错误做法**：直接 `INSERT_AFTER`（id为空）将全部内容追加到文档末尾 → 4.1/4.2/4.3/4.4 出现在末尾，与"四、实现逻辑"下的空标题重复。

**正确做法**：
1. `find` 找到 `4.1 新增页面组件` 的 Block ID → `INSERT_AFTER` 填入 4.1 内容
2. `find` 找到 `4.2 接口说明` 的 Block ID → `INSERT_AFTER` 填入 4.2 内容
3. `find` 找到 `4.3 前端页面功能` 的 Block ID → `INSERT_AFTER` 填入 4.3 内容
4. `find` 找到 `4.4 逻辑处理` 的 Block ID → `INSERT_AFTER` 填入 4.4 内容

---

## 限流（400007）处理

`smartcanvas.*` 工具频繁调用后会触发 400007 限流，持续数分钟。

**减少限流策略**：
- 在限流期间避免调用 `smartcanvas.read`（全文读取调用间隔至少 30 秒）
- 优先使用 `smartcanvas.find`（单次精确查找比全文扫描更轻量）
- 限流期间可用 `curl` 直接请求腾讯文档页面验证内容
- 等待时间：轻度限流约 1-2 分钟，重度限流可能持续 5-10 分钟

---

## 相关文件

- `smartcanvas/entry.md` — SmartCanvas 工具完整参考（含工作流一至工作流九）
