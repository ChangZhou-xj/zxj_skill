---
name: tencent-docs-workorder-sync
description: 源表本地工作记录写入目标复核表 - 支持工单过滤、去重、字段映射、MR信息补充和腾讯文档链接补充，最终同步至无纸化代码复核表。
version: 3.1.0

triggers:
  - 同步工单到支出表格
  - 腾讯文档工单登记
  - 周兴杰工单写入
---


# 源表本地工作记录写入目标复核表（高性能稳定版）

---

# 1. 技能目标

将腾讯文档工单（工作记录）中的符合条件记录：

* 过滤
* 去重
* 字段映射
* 写入目标复核表
* 补充 MR 信息
* 补充腾讯文档链接

最终同步至无纸化代码复核表。

目标表：

```txt
https://docs.qq.com/sheet/DQXpETkpFdGh3emtO?tab=gllec0
```

---

# 2. 总体执行架构（必须遵守）

技能必须采用：

```txt
两阶段执行模型
```

---

# 第一阶段（主同步，必须完成）

仅负责：

* 下载本地 Excel
* 读取本地数据
* 数据过滤
* 去重
* 查找可写入行
* 批量写入基础字段

要求：

```txt
优先保证同步速度与成功率
```

---

# 第二阶段（补偿同步，允许降级）

异步补充：

* MR 链接
* MR 合并状态
* 腾讯文档超链接
* set_link

要求：

```txt
失败不影响主流程成功
```

---

# 3. 强制执行流程

## 步骤1：下载本地文件（必须）

禁止直接读取源表 API。

必须：

```bash
cd /home/ubuntu/performance-tool && npm run download-work-record
```

原因：

源 SmartSheet API 高频限流：

* 400007
* 高频失败

因此：

```txt
必须：
先下载 → 再读取本地 Excel
```

禁止：

```txt
直接读取源表 API
```

---

# 4. 数据源定义

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

# 5. 源表字段索引（0-based）

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

# 6. 数据过滤规则（严格等于）

必须全部满足：

| 字段   | 条件                    |
| ---- | --------------------- |
| 登记人  | `周兴杰`                 |
| 类别   | `需求` 或 `程序缺陷`         |
| 产品标识 | `pcx` 或 `gwwy-uniapp` |
| 任务状态 | `开发完成`                |

禁止：

```txt
contains
模糊匹配
```

必须：

```txt
严格等于
```

---

# 7. 去重规则

目标表中：

```txt
登记人 + A8单号/月度任务编号
```

不能重复。

---

# 8. 目标表字段映射

| 目标列         | col | 来源                |
| ----------- | --- | ----------------- |
| 登记人         | 0   | 固定：周兴杰            |
| A8单号/月度任务编号 | 1   | A8单号，无则提交编号       |
| 涉及前后端编号     | 2   | build_fe_be_str() |
| 项目名称        | 3   | 项目名称              |
| 问题描述        | 4   | 提交信息全文（纯文本）       |
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

# 9. 主同步阶段（高性能模式）

---

# 9.1 主阶段允许写入的字段

主同步阶段：

必须写入：

| col |
| --- |
| 0   |
| 1   |
| 2   |
| 3   |
| 4   |
| 6   |
| 7   |
| 8   |
| 9   |
| 10  |
| 11  |

---

# 9.2 主阶段默认值

| 字段     | 值       |
| ------ | ------- |
| 地址     | `MR待解析` |
| 合并状态   | `MR待解析` |
| 提交信息说明 | 空       |

---

# 9.3 主阶段禁止操作

主同步阶段禁止：

* set_link
* 腾讯文档搜索
* MR 深度查询
* 高频 get_cell_data
* 写后校验
* 二次校验

---

# 10. 前后端编号提取

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

# 11. MR 查询规则（补偿阶段）

---

# 11.1 仓库映射

| 产品标识        | 仓库                        | repositoryId |
| ----------- | ------------------------- | ------------ |
| pcx         | fruits/orange/product/pcx | 4764536      |
| gwwy-uniapp | fruits/lemon/gwwy-uniapp  | -            |

---

# 11.2 MR URL 格式（唯一正确）

```txt
https://codeup.aliyun.com/fruits/orange/product/{product}/change/{localId}
```

必须：

```txt
/change/
```

禁止：

```txt
/changes/
```

---

# 11.3 MR 查询优先级

按顺序查询：

```txt
1. merged/develop
2. merged/dev_test
3. opened/develop
4. opened/dev_test
```

规则：

```txt
单条记录命中即停止后续查询
```

---

# 11.4 MR 查询性能规则（必须）

MR 查询阶段：

允许：

* 批量并发
* 请求缓存
* 命中提前停止

禁止：

* 同一 issue 重复查询
* 全量串行查询

---

# 11.5 MR 查询失败策略

若 MR 查询失败：

必须：

```txt
保留：
MR待解析
```

禁止：

```txt
主流程失败
```

---

# 11.6 MR 状态映射

| 条件                   | 写入值 |
| -------------------- | --- |
| MERGED               | 已合并 |
| TO_BE_MERGED         | 待合并 |
| OPENED               | 待合并 |
| opened 且 status=None | 待合并 |

---

# 11.7 MR 标题匹配规则

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

# 12. 腾讯文档匹配规则（补偿阶段）

---

# 12.1 匹配优先级

优先级1：

```txt
A8单号前缀匹配
```

优先级2：

```txt
前端编号前缀匹配
```

---

# 12.2 标题匹配规则

仅允许：

```python
title.startswith(prefix)
```

禁止：

```python
prefix in title
```

---

# 12.3 腾讯文档性能规则（必须）

允许：

* 搜索缓存
* 批量搜索
* 命中提前停止

禁止：

* 每条记录全目录扫描
* 重复搜索同一关键词

---

# 12.4 腾讯文档失败策略

匹配失败：

允许：

```txt
col[17] 保持为空
```

禁止：

```txt
主流程失败
```

---

# 13. 超链接写入规范

适用于：

* col[5]
* col[17]

---

# 13.1 写入顺序（必须）

步骤1：

```txt
set_range_value 写纯文本
```

步骤2：

```txt
set_link 写超链接
```

---

# 13.2 set_link 参数（必须）

必须：

```json
{
  "display_text": "原文本"
}
```

---

# 13.3 禁止行为

| 错误操作            | 后果    |
| --------------- | ----- |
| 先 set_link      | 内容丢失  |
| 不传 display_text | 单元格清空 |

---

# 14. 目标表读取规则

---

# 14.1 必须使用

```txt
get_cell_data
```

---

# 14.2 禁止依赖

```txt
row_count 判断写入成功
```

原因：

worksheet 空白行不会增加 row_count。

---

# 14.3 高性能读取策略（必须）

允许：

```txt
一次读取有效区域
本地内存扫描
```

禁止：

```txt
逐行 get_cell_data
```

---

# 15. 写入起始行规则

---

# 15.1 空白行定义

同时为空：

* col[0]
* col[1]

---

# 15.2 空白行扫描策略

规则：

```txt
首次命中空白行即停止扫描
```

禁止：

```txt
全表重复扫描
```

---

# 15.3 写入策略

优先：

```txt
复用空白行
```

否则：

```txt
使用 row_count 作为追加起点
```

---

# 16. set_range_value 写入规范

## 服务名（必须）

必须：

```txt
tencent-sheetengine
```

禁止：

```txt
tencent-docs
```

---

# 16.1 正确参数格式

```json
{
  "file_id": "DQXpETkpFdGh3emtO",
  "sheet_id": "gllec0",
  "values": [
    {
      "row": 16,
      "col": 0,
      "value_type": "STRING",
      "string_value": "周兴杰"
    }
  ]
}
```

---

# 16.2 写入性能规则（必须）

必须：

```txt
批量 set_range_value
```

禁止：

```txt
逐单元格单请求写入
```

---

# 16.3 关键规则

| 项      | 说明                  |
| ------ | ------------------- |
| 服务     | tencent-sheetengine |
| values | 单元格对象数组             |
| row    | 仅 cell 内有效          |
| 行索引    | 0-based             |

---

# 16.4 禁止方式

| 错误方式                   | 问题                |
| ---------------------- | ----------------- |
| tencent-docs 写入        | 无权限               |
| set_range_value_by_csv | 列偏移               |
| insert_dimension       | 产生空行              |
| NUMBER 日期              | 日期异常              |
| 顶层 row 参数              | unknown parameter |

---

# 17. 云效 MCP 组织标识

| 用途      | organizationId             |
| ------- | -------------------------- |
| MR 查询   | `5ea04ae0f89c9700014a57dc` |
| 旧组织（禁止） | `AeFcjhNsqZaD`             |

---

# 18. 提测日期规则

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

# 19. 最终返回

同步完成后：

必须返回：

```txt
https://docs.qq.com/sheet/DQXpETkpFdGh3emtO?tab=gllec0
```

---

# 20. 错误处理策略（必须）

---

# 20.1 主流程失败条件

仅以下情况允许主流程失败：

| 条件                   |
| -------------------- |
| 本地 Excel 下载失败        |
| 本地 Excel 无法读取        |
| 目标表不可写               |
| set_range_value 全量失败 |

---

# 20.2 非阻塞错误（必须降级）

以下失败：

```txt
不得中断主流程
```

包括：

| 场景          | 降级方案      |
| ----------- | --------- |
| MR 查询失败     | MR待解析     |
| 腾讯文档未匹配     | col[17] 空 |
| set_link 失败 | 保留纯文本     |
| 部分记录失败      | 跳过继续      |

---

# 21. 推荐执行顺序（高性能）

```txt
1. 下载 Excel
2. 读取本地数据
3. 数据过滤
4. 读取目标表有效区域
5. 内存去重
6. 查找空白行
7. 批量构造 values
8. 批量 set_range_value
9. 返回目标链接
10. 异步补 MR 与超链接
```

---

# 22. 核心性能原则（必须遵守）

允许：

* 批量读取
* 批量写入
* 内存去重
* 请求缓存
* 命中提前停止
* 异步补偿
* 降级处理

禁止：

* 高频 API 调用
* 逐行读取
* 重复 MR 查询
* 重复腾讯文档搜索
* 全量串行 IO
* 因补偿逻辑阻塞主流程
