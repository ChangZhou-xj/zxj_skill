# 工单分析报告 MDX 模板

## 文档结构（五章）

```
---
title: <工单编号>-<简短标题>
icon: 📋
---

## 一、工单基本信息（两列表格）

## 二、需求详解
   2.1 背景说明
   2.2 用户期望
   2.3 预期效果

## 三、涉及业务场景及测试

### 3.1 涉及单据（两列表格）
### 3.2 测试要点（纯文本列表）

## 四、实现逻辑（待开发补充）

### 4.1 前端改动
### 4.2 后端改动
### 4.3 数据字典/枚举

## 五、参考信息（两列表格）
```

## 完整模板示例

```mdx
---
title: KFXQ-CX-202604210001-示例需求标题
icon: 📋
---

## 一、工单基本信息

<Table>
    <TableRow>
        <TableCell>工单编号</TableCell>
        <TableCell>KFXQ-CX-202604210001</TableCell>
    </TableRow>
    <TableRow>
        <TableCell>工单类型</TableCell>
        <TableCell>运维工单-开发需求</TableCell>
    </TableRow>
    <TableRow>
        <TableCell>所属项目</TableCell>
        <TableCell><项目名称></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>产品名称</TableCell>
        <TableCell><产品名称></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>发起人</TableCell>
        <TableCell><发起人></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>发起时间</TableCell>
        <TableCell>2026-04-21 09:30</TableCell>
    </TableRow>
    <TableRow>
        <TableCell>当前状态</TableCell>
        <TableCell><当前状态></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>紧急程度</TableCell>
        <TableCell><紧急程度></TableCell>
    </TableRow>
</Table>

---

## 二、需求详解

### 2.1 背景说明

<背景说明>

### 2.2 用户期望

- <用户期望1>
- <用户期望2>

### 2.3 预期效果

- <预期效果1>
- <预期效果2>

---

## 三、涉及业务场景及测试

### 3.1 涉及单据

<Table>
    <TableRow>
        <TableCell>单据类型</TableCell>
        <TableCell><单据类型></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>涉及模块</TableCell>
        <TableCell><模块名称></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>功能路径</TableCell>
        <TableCell><功能路径></TableCell>
    </TableRow>
</Table>

### 3.2 测试要点（纯文本列表）

- <测试要点1>
- <测试要点2>
- <测试要点3>

---

## 四、实现逻辑（待开发补充）

### 4.1 前端改动

- <前端改动说明>

### 4.2 逻辑处理

- <逻辑处理说明>


---

## 五、参考信息

<Table>
    <TableRow>
        <TableCell>系统平台</TableCell>
        <TableCell><系统平台></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>登录地址</TableCell>
        <TableCell><A8_BASE_URL></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>操作账号</TableCell>
        <TableCell><A8_USERNAME></TableCell>
    </TableRow>
    <TableRow>
        <TableCell>报告生成时间</TableCell>
        <TableCell>2026-04-21</TableCell>
    </TableRow>
</Table>

<Callout icon="ℹ️" blockColor="light_blue" borderColor="sky_blue">经过全面自测，前端页面展示正确,  新增加的功能可以正确使用, 在各关联业务场景中, 均能正确展示与操作，功能符合需求预期，未发现明显缺陷与异常，可正常上线使⽤。</Callout>

---

**— 报告结束 —**
```

## MDX 组件速查

| 组件 | 语法 | 说明 |
|------|------|------|
| 表格 | `<Table> <TableRow> <TableCell>...</TableCell> </TableRow> </Table>` | 两列对比时用两列表格 |
| 无序列表 | 纯文本 `- ` | 列表项直接使用 `- ` 前缀，**不使用** `<BulletedList><ListItem>` |
| 提示框 | `<Callout icon="ℹ️" blockColor="light_blue" borderColor="sky_blue">内容</Callout>` | 补充说明 |
| 分隔线 | `---` | 章节分隔 |
| 二级标题 | `## 标题` | 章节标题 |
| 三级标题 | `### 标题` | 子节标题 |

## 注意事项

- **禁止使用** `<BulletedList><ListItem>` 标签，腾讯文档可能无法正确渲染
- 列表统一使用纯文本 Markdown 格式：`- 内容`
- 公开仓库中的示例数据应使用占位符或脱敏后的通用示例
