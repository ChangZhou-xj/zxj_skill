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

## 数据源与目标
- 源文档 file_id: `DQWRRSWZzQ2VBSmFW`（工单文档）
- 目标文档 file_id: `DQXpETkpFdGh3emtO`，sheet_id: `gllec0`（无纸化代码复核表，URL tab 参数）
- 源数据优先从本地文件读取（API 频繁触发 400007 限流）：`/home/ubuntu/performance-tool/data/work-record.xlsx`（Sheet: 工作记录）
- 写入账号：周兴杰

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
| 项目名称 | col[3] | 源表 `项目名称`（col[9]），可留空 |
| 问题描述 | col[4] | 源表 `提交信息`（col[14]）完整文本，**不是任务内容** |
| 地址 | col[5] | 留空 |
| 初审状态 | col[6] | 固定写 `待初审` |
| 终审状态 | col[7] | 固定写 `待终审` |
| 复核状态 | col[8] | 固定写 `待复核` |
| 合并状态 | col[9] | 固定写 `已合并` |
| 是否涉及前/后端 | col[10] | 有后端编号或前端编号（issue号）→ `是`，否则留空 |
| 提测日期 | col[11] | **格式化字符串** `YYYY年MM月DD日`（如 `2026年04月23日`），**不要用 Excel 序列号** |
| 测试人 | col[12] | 留空 |
| 测试状态 | col[13] | 留空 |
| 测试备注 | col[14] | 留空 |

## 前后端编号提取规则

### 提取逻辑（Python）

```python
import re

def extract_issue(info):
    # 提取提交信息开头的 issue 号（前端编号来源）
    # 例如：#20260420002【需求】... → 20260420002
    m = re.match(r'#(\S+)', info.strip())
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

## 操作流程

### 步骤1：读取源数据
```bash
python3 -c "
import openpyxl, re
from datetime import date
wb = openpyxl.load_workbook('/home/ubuntu/performance-tool/data/work-record.xlsx', data_only=True)
ws = wb['工作记录']
# ... 过滤写入 ...
"
```

### 步骤2：读取目标表已登记单号
```bash
# 用 get_cell_data return_csv=true 读取目标表已有数据
~/.hermes/node/bin/mcporter call tencent-sheetengine get_cell_data --args '{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0",
  "start_row": 0,
  "end_row": 20,
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
  { row: 6, col: 4, value_type: 'STRING', string_value: '#20260420002 【需求】...' },
  { row: 6, col: 6, value_type: 'STRING', string_value: '待初审' },
  { row: 6, col: 7, value_type: 'STRING', string_value: '待终审' },
  { row: 6, col: 8, value_type: 'STRING', string_value: '待复核' },
  { row: 6, col: 9, value_type: 'STRING', string_value: '已合并' },
  { row: 6, col: 10, value_type: 'STRING', string_value: '是' },
  { row: 6, col: 11, value_type: 'STRING', string_value: '2026年04月23日' },
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

### 写入起始行确定
先用 `get_sheet_info` 拿到 `row_count`，写入起始行 = `row_count`（0-based 索引）。

## 验证步骤

1. `get_sheet_info` 查看 `row_count` 是否增加（追加写入后应增加 N 行）
2. `get_cell_data` 查询末尾 N 行，`return_csv: true`，确认关键列：
   - col[0] = `周兴杰`
   - col[1] = A8单号
   - col[2] = `【后端：XXX】【前端：XXX】`
   - col[4] = 提交信息全文
   - col[11] = `YYYY年MM月DD日` 格式日期字符串
3. 如有残留数据，用空字符串 `string_value: ''` 覆盖清除

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
