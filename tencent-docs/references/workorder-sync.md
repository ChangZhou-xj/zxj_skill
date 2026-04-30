---
name: tencent-docs-workorder-sync
description: 腾讯文档工单同步写入技能 — 从源工单表过滤、去重、字段映射后写入无纸化代码复核表
triggers:
  - 同步工单到支出表格
  - 腾讯文档工单登记
  - 周兴杰工单写入
---

# 腾讯文档工单同步写入技能

## 触发条件
将腾讯文档工单（源表：https://docs.qq.com/smartsheet/DQWRRSWZzQ2VBSmFW?tab=BB08J2&_t=1769997401113&nlc=1&needShowTips=1&viewId=vE4ueT）中的记录同步写入无纸化代码复核表（https://docs.qq.com/sheet/DQXpETkpFdGh3emtO?tab=gllec0），需严格过滤、去重、字段映射。

## 源表字段列索引（work-record.xlsx Sheet: 工作记录）

| 列索引 | 字段名 | 说明 |
|--------|--------|------|
| col[0] | 登记人 | |
| col[2] | 类别 | 需求/程序缺陷 等 |
| col[3] | 任务内容 | |
| col[5] | 任务状态 | 开发完成/任务完成/测试中 等 |
| col[9] | 项目名称 | |
| col[11] | 产品标识 | pcx/gwwy-uniapp/pex 等 |
| col[12] | 提交编号 | |
| col[14] | 提交信息 | 前端编号（issue号）和后端编号提取来源 |
| col[16] | A8单号 | |

> ⚠️ 注意：openpyxl 读取后 col 索引即 0-based 列序号，无需额外偏移。

## ⚠️ 强制前置步骤（必须严格遵守）

**在开始任何同步操作之前，必须首先执行：**

```bash
cd /home/ubuntu/performance-tool && npm run download-work-record
```

**原因**：SmartSheet API 频繁触发 400007 限流，直接调用 API 读取源表会导致后续所有操作失败。
**正确流程**：先下载更新本地文件，再从本地文件读取数据，完全规避 API 限流。

**本地源数据文件**：`/home/ubuntu/performance-tool/data/work-record.xlsx`（Sheet: 工作记录）

## 数据源与目标
- 源文档 file_id: `DQWRRSWZzQ2VBSmFW`（工单文档）
- 目标文档 file_id: `DQXpETkpFdGh3emtO`，sheet_id: `gllec0`（无纸化代码复核表，URL tab 参数）
- 写入账号需与登记人一致：周兴杰

## 写入条件（全部满足）

| 字段 | 条件 | 说明 |
|------|------|------|
| 类别 | `需求` 或 `程序缺陷` | 严格等于，非包含 |
| 产品标识 | `pcx` 或 `gwwy-uniapp` | 严格等于，非包含 |
| 登记人 | `周兴杰` | 固定 |
| 任务状态 | `开发完成` | 固定 |
| A8单号/月度任务编号 | 目标表格中不存在相同登记人的相同单号 | 去重检查 |

## 目标表格字段映射

| 目标列 | 列索引 | 来源/逻辑 |
|--------|--------|-----------|
| 登记人 | col[0] | 固定写 `周兴杰` |
| A8单号/月度任务编号 | col[1] | 源表 `A8单号`（col[16]），无则用 `提交编号`（col[12]） |
| 涉及前/后端编号 | col[2] | 格式：`【后端：XXX】【前端：XXX】`（见下方提取规则） |
| 项目名称 | col[3] | 源表 `项目名称`（col[9]），必填 |
| 问题描述 | col[4] | 源表 `提交信息`（col[14]）完整文本，**不是任务内容**，不要修改此列 |

> ⚠️ **提交信息截断问题**：腾讯文档导出的 Excel 可能对长文本做了列宽截断渲染（显示 `...`），openpyxl 直接读取时可能拿到截断值。**正确做法**：用 `openpyxl.load_workbook(path, data_only=True)` 读取时，应确保读取完整单元格内容（.value），如果发现文本以 `...` 结尾，需从源表原始数据重新提取或直接用 read_file 检查实际内容。|

### 地址列（col[5]）— 超链接格式

**地址列写入分两步：先写 URL 文本，再用 `set_link` 转换为可点击超链接。**

MR 链接 URL，格式：`https://codeup.aliyun.com/fruits/orange/product/{product}/change/{localId}`（注意路径是 `/change/` 不是 `/changes/`）：通过产品标识匹配仓库，查找对应 develop 分支的合并请求。

**写入步骤：**

1. **步骤1**：`set_range_value` 写入 URL 文本到 col[5]
2. **步骤2**：`set_link` 将 col[5] 转换为可点击超链接（必须传 `display_text` 为原 URL 文本）

```bash
# 步骤1：写入 URL 文本
~/.hermes/node/bin/mcporter call tencent-sheetengine set_range_value --args '{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0",
  "values": [
    {"row": <row>, "col": 5, "value_type": "STRING", "string_value": "https://codeup.aliyun.com/fruits/orange/product/pcx/change/2713"}
  ]
}'

# 步骤2：set_link 将 col[5] 转换为超链接
~/.hermes/node/bin/mcporter call tencent-sheetengine set_link --args '{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0",
  "row": <row>,
  "col": 5,
  "url": "https://codeup.aliyun.com/fruits/orange/product/pcx/change/2713",
  "display_text": "https://codeup.aliyun.com/fruits/orange/product/pcx/change/2713"
}'
```

> ⚠️ **必须先写文本再设置链接**，否则单元格内容会被清空。`display_text` 必须等于 URL 文本，否则显示异常。

### 地址列查询步骤（关键经验）

#### ⚠️ MCP 工具核心限制

`list_change_requests` 存在两个严重限制，**导致搜索结果不完整**：

1. **`projectIds` 和 `search` 参数不生效**：API 忽略这些服务端过滤参数，直接返回全组织所有项目 MR
2. **结果按 `updated_at` 降序排列 + 100 条限制**：较旧的 MR（如 localId 2729/2713）会被截断在 100 条之外

**典型症状**：20260424001 和 20260423002 在 pcx 项目存在，但全量搜索 100 条结果中找不到——因为这两个 MR 较旧，排序靠后未被返回。

#### 正确匹配流程（三步）

**第一步：Python 过滤（必要）**

```python
import json

with open('/tmp/mrs_result.txt') as f:
    raw = json.load(f).get('result', '[]')
    mrs = json.loads(raw)

targets = {'20260424001', '20260423002'}
found = {}
for mr in mrs:
    url = mr.get('detailUrl', '') or mr.get('webUrl', '')
    title = mr.get('title', '')
    # 过滤：product=pcx 且 targetBranch=develop
    if '/product/pcx/' in url and mr.get('targetBranch') == 'develop':
        for t in targets:
            if t in title:
                found[t] = mr
for t, mr in found.items():
    print(f'{t}: {mr.get("detailUrl")}')
```

**第二步：直接查 localId（兜底方案）**

若第一步过滤后未命中，且已知 localId（如 2729、2713），直接用 `get_change_request` 查询：

```bash
~/.hermes/node/bin/mcporter call alibabacloud-devops-mcp-server get_change_request --args '{
  "organizationId": "5ea04ae0f89c9700014a57dc",
  "repositoryId": "4764536",
  "localId": "2729"
}'
```

此方法绕过搜索直接命中，适用于：搜索过滤无果但你知道该 MR 确实存在于 pcx develop 分支的情况。

**第三步：确认状态字段**

只要匹配到 MR（不管合并状态），地址列均填入 MR 链接，状态列按实际状态填写：

| status | 状态列写法 |
|--------|-----------|
| `MERGED` | 已合并 |
| `TO_BE_MERGED` | 待合并 |
| `OPENED` | 待合并 |
| 其他 | 按实际填写 |

**⚠️ 不要把 TO_BE_MERGED 写成"已合并"**，只有 MERGED 才是已合并。
**⚠️ MR 标题格式注意**：标题可能带状态后缀，如 `#20260423002【需求】...-ljw-复核通过-...`，搜索时仍以 `#提交编号` 开头匹配即可。

#### 仓库映射（注意产品标识 ≠ 仓库名）
| 产品标识 | 仓库地址 | 仓库 ID |
|----------|----------|----------|
| pcx | https://codeup.aliyun.com/fruits/orange/product/pcx | 4764536 |
| gwwy-uniapp | https://codeup.aliyun.com/fruits/lemon/gwwy-uniapp | — |

> ⚠️ pcx ≠ pex，两者仓库不同。地址列写错仓库会导致链接无效。

| 字段 | 列索引 | 来源/逻辑 |
|------|--------|-----------|
| 初审状态 | col[6] | 固定写 `待初审` |
| 终审状态 | col[7] | 固定写 `待终审` |
| 复核状态 | col[8] | 固定写 `待复核` |
| 合并状态 | col[9] | 根据 MR 的 `status` 字段填写：<br>`MERGED` → `已合并`<br>`TO_BE_MERGED` → `待合并`<br>`OPENED` → `待合并`<br>其他 → 按实际填写 |
| 是否涉及前/后端 | col[10] | 默认写 `是`（按当前数据约束 issue 号必存在）；仅当提交信息异常导致前后端编号都未提取到时留空 |
| 提测日期 | col[11] | 同步当天 **+1 天**的格式化字符串 `YYYY年MM月DD日`（如同步日期为 2026-04-23，则写 `2026年04月24日`），**不要用 Excel 序列号** |
| 测试人 | col[12] | 留空 |
| 测试状态 | col[13] | 留空 |
| 测试备注 | col[14] | 留空 |
| 提交信息说明 | col[17] | 腾讯文档超链接，匹配规则见下方"腾讯文档文件夹匹配规则" |

## 前后端编号提取规则

### 提取逻辑（Python）

```python
import re

def extract_issue(info):
    # 提取提交信息开头的 issue 号（前端编号来源）
    # 例如：#20260420002【需求】... → 20260420002
    m = re.match(r'#([A-Za-z0-9-]+)', info.strip())
    return m.group(1) if m else ''

def extract_be(info):
    # 提取【后端编号：XXX】或【后端编码：XXX】
    m = re.search(r'后端编号[：:]\s*([A-Z0-9-]+)', info)
    if not m:
        m = re.search(r'后端编码[：:]\s*([A-Z0-9-]+)', info)
    return m.group(1) if m else ''

def build_fe_be_str(info):
    issue = extract_issue(info)
    be = extract_be(info)
    be_str = f'【后端：{be}】' if be else ''
    fe_str = f'【前端：{issue}】' if issue else ''
    return be_str + fe_str
```

### ⚠️ 重要注意事项

1. **后端编号正则**：必须用 `后端编号`（两个汉字），`后端[编号码]` 匹配单个字符会漏掉，正确写法：
   - ✅ `r'后端编号[：:]\s*([A-Z0-9-]+)'` 或 `r'后端编码[：:]\s*([A-Z0-9-]+)'`
   - ❌ `r'后端[编号码][：:]\s*...'`（漏匹配）
2. **冒号兼容**：中文字符冒号 `：` 和英文 `:` 均需兼容
3. **前端编号**：取提交信息开头的 issue 号（如 `#20260420002` → `20260420002`），**不要**取「关联前端编码」
4. **issue 号一定存在**：即使是纯前端改动，也会以 issue 号开头（如 `#20260416003`）
5. **无后端编号时**：只写【前端：XXX】，不写【后端：】（空内容不写入）
6. **地址列**：后续通过产品标识匹配云效仓库，根据前端提交编号匹配查询 develop 目标分支的合并请求链接 URL，填写到地址列（col[5]）。
   - 产品标识 `pcx` → 仓库地址：`https://codeup.aliyun.com/fruits/orange/product/pcx`
   - 产品标识 `gwwy-uniapp` → 仓库地址：`https://codeup.aliyun.com/fruits/lemon/gwwy-uniapp`
   - ⚠️ 产品标识与仓库名不同（如 pcx ≠ pex），须严格按上述映射填写，不可混淆

## 操作流程

### 步骤1：读取源数据
```bash
python3 -c "
import openpyxl, re
from datetime import date, timedelta
today = date.today()
test_date = (today + timedelta(days=1)).strftime('%Y年%m月%d日')  # 同步当天+1天

wb = openpyxl.load_workbook('/home/ubuntu/performance-tool/data/work-record.xlsx', data_only=True)
ws = wb['工作记录']
# ... 过滤写入 ...
"
```

### 步骤2：读取目标表已登记单号
```bash
# 先取行数，再全量读取已存在登记数据（避免只读前20行造成重复写入）
~/.hermes/node/bin/mcporter call tencent-sheetengine get_sheet_info --args '{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0"
}'

# 用 get_cell_data return_csv=true 读取 0 到 row_count-1 行
~/.hermes/node/bin/mcporter call tencent-sheetengine get_cell_data --args '{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0",
  "start_row": 0,
  "end_row": <row_count-1>,
  "start_col": 0,
  "end_col": 2,
  "return_csv": true
}'
# 提取 col[0]=登记人 和 col[1]=A8单号，构建已登记单号 Set
```

### 步骤3：过滤、去重、字段映射
在 Python 中完成全部过滤逻辑，生成 cell 数组。

### 步骤4：写入目标表
使用 `set_range_value` 一次性写入。

## 写入方式（必须严格遵守）

### ✅ 正确方式：set_range_value（cell 对象绝对索引）

**tencent-sheetengine 的 `set_range_value` 使用 cell 对象数组，每个 cell 包含绝对行/列索引（0-based）：**

```javascript
const values = [
  { row: 6, col: 0, value_type: 'STRING', string_value: '周兴杰' },
  { row: 6, col: 1, value_type: 'STRING', string_value: 'KFXQ-CX-xxx' },
  { row: 6, col: 2, value_type: 'STRING', string_value: '【后端：20260421002】【前端：20260420002】' },
  { row: 6, col: 3, value_type: 'STRING', string_value: 'XXX项目' }, // 项目名称（必填）
  { row: 6, col: 4, value_type: 'STRING', string_value: '#20260420002 【需求】...' },
  { row: 6, col: 5, value_type: 'STRING', string_value: 'https://codeup.aliyun.com/xxx/changes/123' }, // 地址列
  { row: 6, col: 6, value_type: 'STRING', string_value: '待初审' },
  { row: 6, col: 7, value_type: 'STRING', string_value: '待终审' },
  { row: 6, col: 8, value_type: 'STRING', string_value: '待复核' },
  { row: 6, col: 9, value_type: 'STRING', string_value: '已合并' },
  { row: 6, col: 10, value_type: 'STRING', string_value: '是' },
  { row: 6, col: 11, value_type: 'STRING', string_value: test_date },  // 同步当天+1天
  // row 7, col 0-14 ...
];

mcporter call tencent-sheetengine set_range_value --args '{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0",
  "values": <JSON数组>
}'
```

### ❌ 错误方式

| 方式 | 问题 |
|------|------|
| `insert_dimension` + `set_range_value` 分开调用 | 会在表头附近产生多余空行 |
| `set_range_value_by_csv` start_col=1 | 列全部右移一位 |
| `set_range_value` 带 start_col/start_row 参数 | 不支持该参数，报错 |
| 提测日期用 Excel 序列号 NUMBER 类型 | 新行显示格式与原表不一致，应用格式化字符串 |

### 写入起始行确定（优先空白行，再考虑追加）

**核心原则：先找现有空白行填充，写满后再追加新行。**

**判断逻辑：**
1. 用 `get_cell_data` 从第14行（数据起始行）起，每行读取 col[0]（登记人）和 col[1]（A8单号）
2. 若 col[0] 和 col[1] **均为空**，则该行为空白行，可填充
3. 若所有数据行均有内容，再追加新行

**实现步骤：**

```python
# 步骤1：从第14行起扫描，找第一行 col[0] 且 col[1] 都为空的行
start_row = None
for row in range(14, max_row):
    # 读取 col[0] 和 col[1]
    ...
    if col0_empty and col1_empty:
        start_row = row  # 找到空白行
        break

# 步骤2：若没有空白行，则 start_row = max_row（追加模式）
if start_row is None:
    start_row = max_row  # 追加到末位
```

**⚠️ 注意：**
- 目标表前13行（0-12）是表头，从第14行（13，0-based）起才是数据
- 表头行禁止写入
- 写入前必须确认该行确实空白，避免覆盖已有数据

## 腾讯文档文件夹匹配（提交信息说明列超链接）

目标表 **col[17]（提交信息说明）** 可写入腾讯文档超链接，使点击后跳转到腾讯文档。

**⚠️ 重要：提交信息说明是 col[17]，不是 col[4]（col[4] 是问题描述，禁止修改）**

**文件夹信息：**
- 文件夹ID: `AeFcjhNsqZaD`
- 文件夹URL: https://docs.qq.com/desktop/mydoc/folder/AeFcjhNsqZaD

**匹配规则（两段匹配，优先级依次）**：

| 优先级 | 待匹配字段 | 匹配方式 | 示例 |
|--------|-----------|---------|------|
| 1 | A8单号（col[1]） | 前缀匹配文档标题 | `KFXQ-CX-2026033100074` → `KFXQ-CX-2026033100074-报销关联合同穿透查询` |
| 2 | 前端编号（col[2]中提取，如`【前端：20260423002】`） | 前缀匹配文档标题 | `20260423002` → `20260423002-入参新增是否细化（isRefine）` |

**文件夹现有文档列表（按更新时间，offset=0）：**

| 文档标题 | A8单号/编号匹配 |
|---------|----------------|
| KFXQ-CX-2026012100106-移动端首页调整 | KFXQ-CX-2026012100106 |
| KFXQ-CX-2026032700129-回避制度开发需求 | KFXQ-CX-2026032700129 |
| KFXQ-CX-2026031300005-指标按项目分类展示 | KFXQ-CX-2026031300005 |
| KFXQ-CX-2026032600159-支付密码权限优化 | KFXQ-CX-2026032600159 |
| KFXQ-CX-2026033100074-报销关联合同穿透查询 | KFXQ-CX-2026033100074 |
| KFXQ-CX-2026011600136-云南财经无票报销 | KFXQ-CX-2026011600136 |
| 20260423002-入参新增是否细化（isRefine） | 20260423002 |
| KFXQ-CX-2026041500007-出纳支付，统一处理单位代码变化 | KFXQ-CX-2026041500007 |

**匹配示例：**
- A8单号=`KFXQ-CX-2026033100074` → 匹配第5行（优先级1，A8单号前缀匹配）
- A8单号=`无`，前端编号=`20260423002` → 匹配第7行（优先级2，前端编号前缀匹配）

**⚠️ 注意**：A8单号=`无`时无法用优先级1匹配，必须用优先级2的前端编号匹配。

**查询文件夹文档：**
```bash
~/.hermes/node/bin/mcporter call tencent-docs manage.folder_list --args '{
  "folder_id": "AeFcjhNsqZaD",
  "offset": 0
}'
```

## set_link 超链接写入方法

**⚠️ 关键经验（必须严格遵守）：**

`set_link` 不传 `display_text` 会清空单元格内容！正确流程：

### 步骤1：先用 set_range_value 写入文本内容（col[17] 提交信息说明）
```bash
~/.hermes/node/bin/mcporter call tencent-sheetengine set_range_value --args '{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0",
  "values": [
    {"row": 7, "col": 17, "value_type": "STRING", "string_value": "KFXQ-CX-2026033100074-报销关联合同穿透查询"}
  ]
}'
```

### 步骤2：再用 set_link 添加超链接（col[17]，必须带 display_text）
```bash
~/.hermes/node/bin/mcporter call tencent-sheetengine set_link --args '{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0",
  "row": 7,
  "col": 17,
  "url": "https://docs.qq.com/aio/DQWtPVnRXY1V3akRD",
  "display_text": "KFXQ-CX-2026033100074-报销关联合同穿透查询"
}'
```

### set_link 参数说明
| 参数 | 必填 | 说明 |
|------|------|------|
| file_id | 是 | 文档ID |
| sheet_id | 是 | 工作表ID |
| row | 是 | 行号（0-based，col[17]对应提交信息说明） |
| col | 是 | 列号（0-based，提交信息说明为17） |
| url | 是 | 链接URL |
| display_text | 是 | 显示文本（设为原文本内容即可保留显示） |

### ❌ 错误做法
- 先 set_link 再 set_range_value（link 会清除内容）
- set_link 不传 display_text（单元格内容被清空）

## 验证步骤

1. `get_sheet_info` 查看 `row_count` 是否增加（追加写入后应增加 N 行）
2. `get_cell_data` 查询末尾 N 行，`return_csv: true`，确认关键列：
   - col[0] = `周兴杰`
   - col[1] = A8单号
   - col[2] = `【后端：XXX】【前端：XXX】`
   - col[4] = 提交信息全文
   - col[11] = `YYYY年MM月DD日` 格式日期字符串
3. 如有残留数据，用空字符串 `string_value: ''` 覆盖清除

## 写入成功后

同步完成后，**必须**将目标表链接告知用户：

```
https://docs.qq.com/sheet/DQXpETkpFdGh3emtO?tab=gllec0
```

## ⚠️ SmartSheet 与 Sheet 文档服务差异

| 文档类型 | MCP 服务 | 工具 | 备注 |
|----------|----------|------|------|
| SmartSheet（源工单表 `DQWRRSWZzQ2VBSmFW`） | `tencent-docs` | `smartsheet.list_records` | 频繁触发 400007 限流，**优先读本地文件** |
| Sheet（目标无纸化代码复核表） | `tencent-sheetengine` | `set_range_value` / `get_cell_data` | 稳定 |

**本地文件读取**：
```python
import openpyxl
wb = openpyxl.load_workbook('/home/ubuntu/performance-tool/data/work-record.xlsx', data_only=True)
ws = wb['工作记录']
# openpyxl 自动处理 richText 格式，直接 .value 即可
```

**mcporter 路径**：`~/.hermes/node/bin/mcporter`（which 查不到，需用完整路径）
