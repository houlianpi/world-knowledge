# World Knowledge Schema

> 本文件定义知识库的结构规范、标签分类、进化规则。
> Agent 必须遵守这些约束。修改本文件需要人工审批。

## Domain

**macOS UI 自动化知识库**，为 LLM Planner 提供结构化的执行知识。

涵盖领域：
- **Apps** — 应用注册信息 (能力、快捷键、UI 结构)
- **Patterns** — 可复用的交互模式 (操作序列模板)
- **Elements** — UI 元素定位策略 (多策略、按优先级)
- **Cases** — 成功案例的执行路径 (few-shot 示例)
- **Failures** — 已知陷阱和解决方案 (避免重蹈覆辙)

## Directory Structure

```
world-knowledge/
├── SCHEMA.md          # 本文件 - 规范约束
├── INDEX.md           # 导航索引
├── LOG.md             # 变更日志 (append-only)
│
├── raw/               # Layer 1: 原始资产 (不可变)
│   ├── cases/         # 导入的测试用例 JSON
│   ├── traces/        # 成功执行的完整 trace
│   └── screenshots/   # UI 截图参考
│
├── apps/              # Layer 2: App 注册表
├── patterns/          # Layer 2: 交互模式
│   ├── browser/       # 浏览器相关
│   └── system/        # 系统级操作
├── elements/          # Layer 2: 元素定位
├── cases/             # Layer 2: 案例记忆
├── failures/          # Layer 2: 失败模式
│
└── _archive/          # 归档 (低置信度/废弃)
```

## Conventions

### File Naming

- 小写 + 连字符: `microsoft-edge.md`, `browser-navigate.md`
- 元素按上下文分组: `elements/github/login-button.md`
- 模式按类别分组: `patterns/browser/navigate.md`

### Frontmatter (必须)

每个知识页面必须有 YAML frontmatter:

```yaml
---
id: "unique-id"                       # 全局唯一，格式: type.category.name
type: app | pattern | element | case | failure
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: 0.95                      # 0.0 - 1.0
source: static | imported | discovered | user_confirmed
tags: [from taxonomy below]
links: [related-id-1, related-id-2]   # 至少 2 个交叉引用
---
```

### Cross-References

- 使用 `[[id]]` 语法链接其他页面
- **每个页面至少 2 个出站链接** (除非是首批创建的)
- 创建新页面时必须建立与现有页面的关联

### Index & Log Maintenance

- **每次**创建/更新/删除页面，必须同步更新 `INDEX.md`
- **每个**操作必须 append 到 `LOG.md`

---

## Tag Taxonomy

### App Categories
- `browser` — Safari, Chrome, Edge, Firefox, Arc
- `editor` — VS Code, Xcode, TextEdit
- `terminal` — Terminal, iTerm, Warp
- `finder` — Finder, file management
- `system` — System Settings, Spotlight
- `productivity` — Notes, Reminders, Calendar, Mail
- `media` — Photos, Music, QuickTime

### Action Types
- `navigation` — URL 导航, 页面跳转
- `input` — 文本输入, 表单填写
- `click` — 元素点击
- `shortcut` — 键盘快捷键
- `menu` — 菜单操作
- `drag` — 拖拽操作
- `scroll` — 滚动
- `wait` — 等待

### Verification Types
- `url` — URL 包含/匹配
- `element` — 元素存在/可见
- `text` — 文本内容
- `screenshot` — 截图对比
- `window` — 窗口状态

### Confidence Sources
- `static` — 人工编写 (confidence = 1.0)
- `imported` — 从 case 导入 (confidence = 0.8-0.9)
- `discovered` — 运行时发现 (confidence = 0.5-0.7)
- `user_confirmed` — 用户确认后 (+0.1-0.2)

**规则:** 标签必须来自此分类。需要新标签时，先在这里添加。

---

## Page Thresholds

何时创建页面，何时不创建：

| 类型 | 创建条件 | 不创建 |
|------|----------|--------|
| **App** | 需要自动化的应用 | 未安装/未使用的 App |
| **Pattern** | 2+ 次成功使用的操作序列 | 单次成功的特殊情况 |
| **Element** | 3+ 次成功定位 | 偶然出现的动态元素 |
| **Case** | 复杂目标 (3+ 步骤) 的成功路径 | 简单单步操作 |
| **Failure** | 2+ 次出现的失败模式 | 随机/偶发错误 |

---

## Confidence Evolution

置信度随执行反馈动态调整：

```
成功执行:     confidence += 0.05  (上限 1.0)
失败执行:     confidence -= 0.10  (下限 0.0)
用户确认:     confidence += 0.15
长期未验证:   confidence -= 0.02/month  (> 90 天)
```

**清理阈值:** `confidence < 0.3` 时，lint 标记为 "待清理"

**归档阈值:** `confidence < 0.2` 且 90 天未更新 → 移到 `_archive/`

---

## Update Policy

### 冲突处理

当新信息与现有内容冲突时：

1. **不删除**旧信息，而是记录两种说法
2. 标注各自的日期和来源
3. 在 frontmatter 添加 `contradictions: [page-id]`
4. lint 时提醒人工审查

### 拆分阈值

页面超过 **100 行**时，考虑拆分为子主题页面。

### 归档流程

1. 将页面移动到 `_archive/` (保持原路径结构)
2. 从 `INDEX.md` 移除
3. 更新指向该页面的链接为纯文本 + `(archived)`
4. 记录到 `LOG.md`

---

## Page Templates

### App Page

```markdown
---
id: "app.{name}"
type: app
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: 1.0
source: static
tags: [{category}]
links: [pattern.x, pattern.y]
---

# {App Name}

{简短描述}

## Basic Info

| 属性 | 值 |
|------|-----|
| Bundle ID | {bundle_id} |
| Category | {category} |
| Aliases | {aliases} |

## Capabilities

- {capability 1}
- {capability 2}

## Shortcuts

| 操作 | 按键 |
|------|------|
| {action} | {keys} |

## UI Hints

- {role/selector hints}

## Related

- [[pattern.x]] — {描述}
- [[pattern.y]] — {描述}
```

### Pattern Page

```markdown
---
id: "pattern.{category}.{name}"
type: pattern
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: 0.9
source: imported
tags: [{category}, {action_type}]
links: [app.x, app.y, pattern.z]
verified_count: 0
last_verified: null
---

# {Pattern Name}

{简短描述}

## Applies To

| App | Confidence | Last Verified |
|-----|------------|---------------|
| [[app.x]] | 0.9 | YYYY-MM-DD |

## Parameters

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| {name} | {type} | ✓/○ | {description} |

## Semantic Steps

1. {step 1} — {描述}
2. {step 2} — {描述}

## Compiled Steps

```yaml
steps:
  - command: {cmd}
    params: {params}
```

## Success Criteria

```yaml
criteria:
  - type: {type}
    value: {value}
```

## Known Issues

- {issue 1}

## Related

- [[app.x]]
- [[pattern.y]]
```

---

## Lint Rules

定期运行 lint 检查：

1. **孤立页面** — 无入站 `[[links]]` 的页面
2. **断开链接** — 指向不存在页面的 `[[links]]`
3. **INDEX 完整性** — 所有页面都在 INDEX.md 中
4. **Frontmatter 验证** — 必填字段、有效标签
5. **低置信度** — confidence < 0.3
6. **过期内容** — 90+ 天未验证
7. **页面过长** — 超过 100 行
8. **标签合规** — 只使用 Taxonomy 中定义的标签

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-04-18 | Initial schema |
