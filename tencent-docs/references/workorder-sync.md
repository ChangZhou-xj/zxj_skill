# 优化版 Skill（结构化重构版）

```yaml
---
name: tencent-docs-workorder-sync
description: 腾讯文档工单同步写入技能（稳定版）— 从本地工单文件过滤、去重并同步至无纸化代码复核表
version: 2.0.0

triggers:
  - 同步工单到支出表格
  - 腾讯文档工单登记
  - 周兴杰工单写入
---
```

# 腾讯文档工单同步技能（稳定版）

---

# 1. 技能目标

将腾讯文档工单（工作记录）中的符合条件记录：

* 过滤
* 去重
* 字段映射
* 写入目标复核表

目标表：

```txt
https://docs.qq.com/sheet/DQXpETkpFdGh3emtO?tab=gllec0
```

---

# 2. 强制执行流程（必须遵守）

## 步骤1：先下载本地文件（禁止直接读取源表）

```bash
cd /home/ubuntu/performance-tool && npm run download-work-record
```

### 原因

源 SmartSheet API 存在：

* 400007 限流
* 高频失败

因此：

## 必须：

* 先下载
* 再读取本地 Excel

## 禁止：

* 直接读取源表 API

---

# 3. 数据源定义

## 源表

| 项          | 值                                                     |
| ---------- | ----------------------------------------------------- |
| file_id    | `300000000$AdQIfsCeAJaV`                              |
| sheet_name | `工作记录`                                                |
| sheet_id   | `BB08J2`                                              |
| 本地文件       | `/home/ubuntu/performance-tool/data/work-record.xlsx` |

---

## 目标表

| 项        | 值                   |
| -------- | ------------------- |
| file_id  | `DQXpETkpFdGh3emtO` |
| sheet_id | `gllec0`            |

---

# 4. 源表字段索引（0-based）

| col | 字段   |
| --- | ---- |
| 0   | 登记人  |
| 2   | 类别   |
| 3   | 任务内容 |
| 5   | 任务状态 |
| 9   | 项目名称 |
| 11  | 产品标识 |
| 12  | 提交编号 |
| 14  | 提交信息 |
| 16  | A8单号 |

---

# 5. 数据过滤规则（严格等于）

## 必须全部满足

| 字段   | 条件                    |
| ---- | --------------------- |
| 登记人  | `周兴杰`                 |
| 类别   | `需求` 或 `程序缺陷`         |
| 产品标识 | `pcx` 或 `gwwy-uniapp` |
| 任务状态 | `开发完成`                |

---

# 6. 去重规则

目标表中：

```txt
登记人 + A8单号/月度任务编号
```

不能重复。

---

# 7. 目标表字段映射

| 目标列         | col | 来源                |
| ----------- | --- | ----------------- |
| 登记人         | 0   | 固定：周兴杰            |
| A8单号/月度任务编号 | 1   | A8单号，无则提交编号       |
| 涉及前后端编号     | 2   | build_fe_be_str() |
| 项目名称        | 3   | 项目名称              |
| 问题描述        | 4   | 提交信息全文            |
| 地址          | 5   | MR 链接             |
| 初审状态        | 6   | 待初审               |
| 终审状态        | 7   | 待终审               |
| 复核状态        | 8   | 待复核               |
| 合并状态        | 9   | MR 状态映射           |
| 是否涉及前后端     | 10  | 默认：是              |
| 提测日期        | 11  | 同步日期+1天           |
| 测试人         | 12  | 空                 |
| 测试状态        | 13  | 空                 |
| 测试备注        | 14  | 空                 |
| 提交信息说明      | 17  | 腾讯文档超链接           |

---

# 8. 关键字段规则

---

## 8.1 问题描述（col[4]）

### 必须：

* 保存提交信息全文
* 纯文本

### 禁止：

* set_link
* 超链接
* 修改内容

---

## 8.2 提交信息说明（col[17]）

允许：

* 腾讯文档超链接

---

# 9. 前后端编号提取

## Python 实现

```python
import re

def extract_issue(info):
    m = re.match(r'#([A-Za-z0-9-]+)', info.strip())
    return m.group(1) if m else ''

def extract_be(info):
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

---

# 10. MR 查询规则（最终版）

---

# 10.1 仓库映射

| 产品标识        | 仓库                        | repositoryId |
| ----------- | ------------------------- | ------------ |
| pcx         | fruits/orange/product/pcx | 4764536      |
| gwwy-uniapp | fruits/lemon/gwwy-uniapp  | -            |

---

# 10.2 MR URL 格式（唯一正确格式）

```txt
https://codeup.aliyun.com/fruits/orange/product/{product}/change/{localId}
```

## 注意

### 正确：

```txt
/change/
```

### 错误：

```txt
/changes/
```

---

# 10.3 MR 查询优先级

按以下顺序查询：

```txt
1. merged/develop
2. merged/dev_test
3. opened/develop
4. opened/dev_test
```

命中即停止。

---

# 10.4 MR 状态映射

| 条件                   | 写入值 |
| -------------------- | --- |
| MERGED               | 已合并 |
| TO_BE_MERGED         | 待合并 |
| OPENED               | 待合并 |
| opened 且 status=None | 待合并 |

---

# 10.5 MR 标题匹配规则

仅匹配：

```txt
#号开头 + 【之前
```

示例：

```txt
#20260423002【需求】xxx
```

提取：

```txt
#20260423002
```

---

# 11. 腾讯文档匹配规则

---

# 11.1 匹配优先级

## 优先级1

A8单号前缀匹配。

---

## 优先级2

前端编号前缀匹配。

---

# 11.2 文档标题匹配

仅允许：

```python
title.startswith(prefix)
```

禁止：

```python
prefix in title
```

---

# 12. 超链接写入统一规范（必须遵守）

所有超链接字段：

* col[5]
* col[17]

必须按以下顺序：

---

## 步骤1：set_range_value 写文本

---

## 步骤2：set_link 写超链接

必须：

```json
{
  "display_text": "原文本"
}
```

---

# 禁止行为

| 错误操作            | 后果    |
| --------------- | ----- |
| 先 set_link      | 内容丢失  |
| 不传 display_text | 单元格清空 |

---

# 13. 目标表读取规则

---

# 13.1 读取方式

必须：

```txt
get_cell_data
```

---

# 13.2 禁止依赖 row_count

worksheet 类型：

```txt
写入空白行时 row_count 不增加
```

因此：

## 验证写入成功

必须：

```txt
get_cell_data
```

禁止：

```txt
仅依赖 row_count
```

---

# 14. 写入起始行规则

---

# 14.1 空白行定义

以下字段同时为空：

* col[0]
* col[1]

---

# 14.2 写入策略

## 优先：

复用空白行。

## 若不存在空白行：

使用：

```txt
row_count
```

作为追加起点。

---

# 15. set_range_value 写入规范

## 正确方式

使用：

```json
{
  "row": 14,
  "col": 0,
  "value_type": "STRING",
  "string_value": "周兴杰"
}
```

---

# 禁止方式

| 错误方式                   | 问题   |
| ---------------------- | ---- |
| set_range_value_by_csv | 列偏移  |
| insert_dimension       | 产生空行 |
| Excel NUMBER 日期        | 格式异常 |

---

# 16. 提测日期规则

格式：

```txt
YYYY年MM月DD日
```

示例：

```txt
2026年05月09日
```

禁止：

* Excel 序列号
* NUMBER 类型

---

# 17. 验证规则

写入后：

必须使用：

```txt
get_cell_data
```

验证：

| 列       | 验证内容   |
| ------- | ------ |
| col[0]  | 周兴杰    |
| col[1]  | A8单号   |
| col[2]  | 前后端编号  |
| col[4]  | 提交信息全文 |
| col[11] | 格式化日期  |

---

# 18. 最终返回

同步完成后：

必须返回：

```txt
https://docs.qq.com/sheet/DQXpETkpFdGh3emtO?tab=gllec0
```

---

# 19. 实现建议（强烈推荐）

同步流程建议拆为两阶段：

---

## 第一阶段（主同步）

仅同步：

* 基础字段
* issue号
* 提交信息

MR 状态：

```txt
MR待解析
```

---

## 第二阶段（异步补偿）

异步补充：

* MR 链接
* 合并状态

---

# 20. 禁止事项（非常重要）

| 禁止项                  | 原因            |
| -------------------- | ------------- |
| 直接读取源 SmartSheet API | 高频限流          |
| 使用 /changes/         | URL 无效        |
| 修改 col[4] 为超链接       | 会污染问题描述       |
| 依赖 row_count 判断成功    | worksheet 不可靠 |
| 用 contains 做业务过滤     | 会误匹配          |
| 不传 display_text      | set_link 清空内容 |
