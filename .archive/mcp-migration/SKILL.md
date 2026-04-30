---
name: mcp-migration
description: "Use when: 迁移技能使用新的 MCP 服务器、评估新旧 API 差异、创建完整的 MCP 工具文档、处理敏感信息、发布技能到仓库。关键词：MCP、迁移、配置、文档化、Playwright、GitHub、发布。"
---

# MCP 服务器迁移与发布工作流

## 适用场景

当需要：
1. 将现有技能从旧版 API 迁移到新的 MCP 服务器
2. 评估新旧工具的差异和新增功能
3. 为 MCP 工具创建完整文档
4. 处理公开仓库中的敏感信息
5. 将更新的技能发布到 Git 仓库

## 工作流程

### 步骤 1：评估新旧 API 差异

#### 1.1 分析当前使用的工具
- 查看当前技能中使用的工具调用
- 识别需要替换的旧 API
- 记录当前的工作流程

#### 1.2 分析新 MCP 服务器的工具
- 查看新 MCP 服务器的官方文档
- 了解可用的工具列表和参数
- 识别新增功能

#### 1.3 创建对照表
创建功能对比表格，识别：
- 功能相同的工具
- 新增的工具
- 已废弃的工具
- 参数命名变化

示例：
| 旧 API | 新 API | 说明 |
|--------|--------|------|
| `browser(action="navigate", ...)` | `mcp_playwright_browser_navigate()` | 功能相同 |
| - | `mcp_playwright_browser_file_upload()` | 新增：文件上传 |

### 步骤 2：配置 MCP 服务器

#### 2.1 添加服务器配置
在 Hermes 配置中添加新的 MCP 服务器。

配置示例（需通过 Hermes 配置方式添加）：
```yaml
mcp_servers:
  playwright:
    command: npx
    args: ["-y", "@playwright/mcp@latest"]
    timeout: 180  # 可选：增加超时时间
```

> **注意**：具体配置方式取决于你的 Hermes 安装方式。请参考 Hermes 文档或使用配置工具。

#### 2.2 验证安装
- 检查 MCP 服务器版本
- 重启 Hermes Agent 以加载新配置
- 查看日志确认服务器已连接

#### 2.3 验证工具可用性
在 Hermes 中列出所有可用的 MCP 工具。

预期工具命名格式：`mcp_<server-name>_<tool_name>`

### 步骤 3：创建完整文档

#### 3.1 主文档结构
创建 `references/<server-name>-mcp.md`，包含：

```markdown
# <Server Name> MCP <功能说明>

## 概述
- MCP 服务器简介
- 核心特性
- 使用场景

## 配置
- config.yaml 配置示例
- 环境要求

## 工具列表

### 核心导航与操作
（为每个工具提供：用途、参数、示例代码）

### 高级功能
（如文件上传、下拉选择等）

### 调试与监控
（如网络请求、控制台日志等）

## 典型工作流程
- 分步骤的实际使用示例
- 完整的代码片段

## 注意事项
- 限制和注意事项
- 最佳实践

## 资源
- 官方文档链接
- GitHub 仓库
```

#### 3.2 快速测试文档
创建 `references/<server-name>-mcp-quicktest.md`，包含：

- 安装验证步骤
- 基本功能测试用例
- 可用工具列表
- 常见问题 Q&A

#### 3.3 迁移指南
创建 `references/MIGRATION.md`，包含：

- 变更摘要
- 新增功能说明
- 完整的工具名称对照表
- 旧代码 vs 新代码对比示例
- 验证安装步骤
- 故障排查指南

### 步骤 4：更新技能文档

#### 4.1 更新 SKILL.md
- 更新前置条件，添加 MCP 服务器依赖
- 引用新的文档
- 更新工作流程，使用新 API
- 添加批量操作示例（如果支持）
- 添加搜索/高级功能示例

#### 4.2 处理旧文档
```markdown
# 旧文档名称

> **⚠️ 已废弃**：本文档已过时，请使用 **[新文档](./<new-doc>.md)**。
>
> 旧版 <old-api> 已被 <new-api> 替代。新工具提供更完整的功能和更好的性能。

## 核心原则
（保留核心原则，更新过时内容）
```

### 步骤 5：处理敏感信息

#### 5.1 识别敏感信息
常见敏感信息：
- IP 地址和端口号
- 登录账号和密码
- API 密钥和 Token
- 数据库连接字符串
- 内部系统 URL
- 邮箱地址和手机号

#### 5.2 替换为占位符
| 实际值类型 | 占位符示例 |
|------------|------------|
| 系统地址 | `<A8_BASE_URL>` |
| 登录账号 | `<A8_USERNAME>` |
| 登录密码 | `<A8_PASSWORD>` |
| API 密钥 | `<API_KEY>` |
| 内部邮箱 | `<ADMIN_EMAIL>` |

#### 5.3 添加安全提示
```markdown
> **⚠️ 安全提示**：登录凭据和系统地址为敏感信息，仅用于本地环境。
> 如需提交到公共 Git 仓库，请使用占位符。
> 本地配置请在私有环境或 `.env` 文件中管理。
```

### 步骤 6：Git 工作流

#### 6.1 查看变更
查看技能目录的变更状态

#### 6.2 暂存文件
暂存所有修改和新增的文档

#### 6.3 创建提交
使用清晰的提交信息格式：
```
feat: 升级到 <New MCP Server> <功能说明>

变更内容：
- 替换旧版 <old-api> 为 <new-mcp-server>
- 添加完整的 <new-mcp-server> 工具文档（N 个工具）
- 新增快速测试指南和迁移指南
- 更新 SKILL.md 工作流程使用新 API
- 标记旧文档为废弃，添加迁移提示
- 将敏感信息替换为占位符

新增功能：
- <功能1>
- <功能2>

文档结构：
- SKILL.md - 主技能文档（含占位符）
- references/<new-doc>.md - 完整工具文档
- references/<quicktest>.md - 快速测试指南
- references/MIGRATION.md - 旧 API 迁移指南
```

#### 6.4 推送到远程仓库
将变更推送到远程仓库

#### 6.5 验证发布
- 查看最近的提交历史
- 在 GitHub 上查看变更详情

## 常见 MCP 服务器

### Playwright MCP
- 包名：`@playwright/mcp`
- 工具前缀：`mcp_playwright_`
- 功能：浏览器自动化
- 特点：18 个工具，支持表单、文件上传、网络监控

### GitHub MCP
- 包名：`@modelcontextprotocol/server-github`
- 工具前缀：`mcp_github_`
- 功能：GitHub 仓库操作
- 特点：需要 PAT 认证

### Filesystem MCP
- 包名：`@modelcontextprotocol/server-filesystem`
- 工具前缀：`mcp_filesystem_`
- 功能：文件系统访问
- 特点：可限制访问路径

## 最佳实践

### 1. 文档结构
- **主文档**：完整的参考文档，包含所有工具
- **快速测试**：验证和调试指南
- **迁移指南**：帮助平滑过渡
- **旧文档**：标记废弃，但保留参考

### 2. 代码示例
- 提供完整可运行的示例
- 使用批量操作提高效率
- 包含必要的注释
- 展示错误处理

### 3. 安全性
- 始终使用占位符
- 添加明确的安全提示
- 说明本地配置方法
- 避免硬编码敏感信息

### 4. Git 提交
- 使用 Conventional Commits 格式
- 提供详细的变更说明
- 列出新增文件和修改的文件
- 包含文档结构说明

## 故障排查

### 工具未出现
- 检查配置是否正确
- 重启 Hermes Agent
- 查看日志中的连接错误

### 执行失败
- 检查 MCP 服务器是否安装
- 验证网络连接
- 增加超时配置

### 页面加载超时
- 增加 timeout 配置
- 检查目标网站可访问性
- 使用网络监控诊断问题

### 敏感信息泄露
- 审查所有新增和修改的文档
- 使用占位符搜索：`grep -r "<.*_.*>" .`
- 查看 git diff 确认无意外提交

## 参考资料

- MCP 协议规范：https://modelcontextprotocol.io
- Hermes MCP 客户端：`native-mcp` skill
- Playwright 官方文档：https://playwright.dev
- Conventional Commits：https://www.conventionalcommits.org
