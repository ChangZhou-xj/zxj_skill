# 浏览器操作规范

## 核心原则

- 始终使用**隔离浏览器 profile**，不要在用户日常使用的浏览器会话中执行自动化。
- 公开仓库中的 profile 名称使用占位符或示例值；若你的团队约定使用 `openclaw`，可以在私有配置中落地。

## 启动与导航

```javascript
// 启动隔离浏览器
browser(action="start", profile="<ISOLATED_BROWSER_PROFILE>")

// 导航到目标页面
browser(action="navigate", profile="<ISOLATED_BROWSER_PROFILE>", url="<A8_LOGIN_URL>")

// 等待页面加载后获取快照
browser(action="snapshot", profile="<ISOLATED_BROWSER_PROFILE>", compact=true)
```

## 填写表单

```javascript
// 使用 fields 数组，ref 不带 @ 前缀
browser(action="act", profile="<ISOLATED_BROWSER_PROFILE>", kind="fill",
  fields=[
    {"ref":"<USERNAME_REF>","text":"<A8_USERNAME>"},
    {"ref":"<PASSWORD_REF>","text":"<A8_PASSWORD>"}
  ])

// 点击按钮
browser(action="act", profile="<ISOLATED_BROWSER_PROFILE>", ref="<SUBMIT_REF>", kind="click")

// 按回车
browser(action="act", profile="<ISOLATED_BROWSER_PROFILE>", ref="<PASSWORD_REF>", kind="press", key="Enter")
```

## A8 系统常见特点

### iframe 结构

A8 系统常见大量 iframe 嵌套，点击工单链接后不一定改变主 frame URL。建议：
1. 先 `snapshot` 获取当前页面的可点击元素
2. 在 refs 中找到目标工单的 ref
3. 直接在当前可操作上下文中点击

### 工单列表定位

常见分类路径示例：
- **运维管理**：部分环境中包含 `运维工单-开发需求--KFXQ-CX-...` 格式的 KFXQ 类工单
- **缺陷管理**：部分环境中包含 QXWT 类工单
- **需求管理**：部分环境中包含需求类工单

经验：
- 首页的“待办中心”卡片通常比顶部入口更稳定，可优先尝试
- 工单标题在快照中通常比页面视觉更完整，以快照为准
- 若快照中标题完整而页面显示截断，这是正常现象

### 兜底 URL

若首页入口持续不可点击，可以尝试使用目标环境中的完整待办列表 URL，例如：
- `<A8_HOME_URL>`
- `<A8_TODO_URL>`

## 调试技巧

```javascript
// 截图（用于人工查看当前页面状态）
browser(action="screenshot", profile="<ISOLATED_BROWSER_PROFILE>", fullPage=true)

// 截图到文件（示例路径请替换为本地可写目录）
browser(action="screenshot", profile="<ISOLATED_BROWSER_PROFILE>", path="<DEBUG_SCREENSHOT_PATH>")

// 执行 JavaScript（滚动、获取元素内容等）
browser(action="act", profile="<ISOLATED_BROWSER_PROFILE>", kind="evaluate",
  fn="window.scrollTo(0, 500)")

// 等待元素出现
browser(action="act", profile="<ISOLATED_BROWSER_PROFILE>", kind="wait",
  selector="#someId", timeoutMs=10000)
```

## 注意事项

1. **不要操作用户自己的浏览器** — 使用隔离 profile 避免冲突
2. **等待页面加载** — 列表页加载较慢，建议等待页面稳定后再继续
3. **刷新 refs** — 若点击失败，先重新 `snapshot` 再尝试，不要重复使用旧 ref
4. **跨域限制** — iframe 内嵌页面内容未必能直接读取，必要时使用截图辅助提取
