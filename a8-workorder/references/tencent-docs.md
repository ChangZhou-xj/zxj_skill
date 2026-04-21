# 腾讯文档操作规范

## 创建智能文档（推荐）

使用 tencent-docs 的 `create_smartcanvas_by_mdx` 能力：

```bash
mcporter call "tencent-docs" "create_smartcanvas_by_mdx" --args '{
  "title": "<文档标题（限36字）>",
  "content_format": "mdx",
  "mdx": "<MDX格式内容>"
}'
```

返回结果通常包含：
- `file_id`：文档 ID，用于后续权限设置
- `url`：文档链接

### MDX 格式基础

```mdx
---
title: <文档标题>
icon: 📋
---

正文内容

<Table>
    <TableRow>
        <TableCell>单元格1</TableCell>
        <TableCell>单元格2</TableCell>
    </TableRow>
</Table>

- 列表项1
- 列表项2

<Callout icon="ℹ️" blockColor="light_blue" borderColor="sky_blue">提示内容</Callout>
```

完整 MDX 规范与列表写法以 `references/mdx-template.md` 为准。

## 设置文档权限（强制）

**创建文档后必须立即设置权限，否则文档可能保持私密状态而无法分享。**

```bash
mcporter call "tencent-docs" "manage.set_privilege" --args '{
  "file_id": "<文档ID>",
  "policy": 2
}'
```

| policy 值 | 含义 | 适用场景 |
|-----------|------|---------|
| `2` | 所有人可读 | 工单分析报告（推荐） |
| `3` | 所有人可编辑 | 需要多人协作编辑 |

## 重命名文档

```bash
mcporter call "tencent-docs" "manage.rename_file_title" --args '{
  "file_id": "<文档ID>",
  "title": "<新标题（限36字）>"
}'
```

## 查询文档信息

```bash
mcporter call "tencent-docs" "manage.query_file_info" --args '{
  "file_id": "<文档ID>"
}'
```

## 完整工作流示例

```bash
# 步骤1：创建文档
mcporter call "tencent-docs" "create_smartcanvas_by_mdx" --args '{
  "title": "<工单编号>-<简短标题>",
  "content_format": "mdx",
  "mdx": "---\ntitle: <工单编号>-<简短标题>\nicon: 📋\n---\n\n## 一、工单基本信息\n\n<Table>...</Table>\n..."
}'

# 步骤2：设置权限（从步骤1返回中获取 file_id）
mcporter call "tencent-docs" "manage.set_privilege" --args '{
  "file_id": "<文档ID>",
  "policy": 2
}'

# 步骤3：返回文档链接给用户
# <TENCENT_DOC_URL>
```

## 工具速查

| 任务 | 工具 |
|------|------|
| 创建智能文档 | `create_smartcanvas_by_mdx` |
| 创建 Word 文档 | `manage.create_file` (file_type=doc) |
| 设置权限 | `manage.set_privilege` |
| 重命名 | `manage.rename_file_title` |
| 查询信息 | `manage.query_file_info` |
| 搜索文档 | `manage.search_file` |
| 删除文档 | `manage.delete_file` |
