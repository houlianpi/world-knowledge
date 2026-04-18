# 自动化测试框架研究总结

> 研究日期: 2026-04-18
> 目的: 为 goal-driven-automation 项目提供业界最佳实践参考

---

## 1. 研究概览

| 框架 | 平台 | Star | 核心特点 | 对 goal-driven 的借鉴价值 |
|------|------|------|---------|-------------------------|
| **Playwright** | Web | 75k+ | Locator 延迟求值、自动等待、链式选择器 | Locator 模式、选择器语法 |
| **Cypress** | Web | 47k+ | 命令队列、自动重试、稳定性等待 | 重试机制、可操作性检查 |
| **Appium** | 多平台 | 21k+ | Driver 机制、W3C 协议、插件化 | 协议设计、扩展架构 |
| **Robot Framework** | 通用 | 10k+ | 关键字驱动、嵌入式参数、DSL | 关键字抽象、数据驱动 |
| **EarlGrey** | iOS | 5.7k | 同步机制、Matcher 组合、白盒测试 | 同步等待、Matcher 设计 |
| **Detox** | RN | 11k+ | 灰盒测试、IdlingResource、三进程架构 | 同步机制、状态追踪 |

---

## 2. Playwright 架构精要

### 2.1 三层架构
```
Client (ChannelOwner) → Protocol/Dispatcher → Server (SdkObject)
```

### 2.2 Locator 模式 (核心)
- **延迟求值**: 选择器字符串，执行时才查询元素
- **自动等待**: 内置 actionability 检查
- **链式组合**: `role=button >> text=Submit`

### 2.3 选择器引擎
```
role, text, css, xpath, id, data-testid, 
nth, has, has-not, visible, enabled, ...
```

### 2.4 借鉴建议
- 采用 Locator 模式延迟元素查询
- 设计 macOS 选择器语法 (参考链式模式)
- 文档驱动类型生成

---

## 3. Cypress 架构精要

### 3.1 命令队列
- 双向链表管理命令 (`prev`/`next` 引用)
- 支持嵌套命令插入
- 状态机: `queued` → `pending` → `passed`/`failed`/`skipped`

### 3.2 重试机制
```typescript
retry(fn, options) {
  const total = Date.now() - options._start
  if ((total + interval) >= options._runnableTimeout) {
    throwErr()
  }
  return Promise.delay(interval).then(() => whenStable(fn))
}
```

### 3.3 稳定性等待 (Stability)
```typescript
whenStable(fn) {
  if (state('isStable') !== false) return Promise.try(fn)
  whenStableQueue.push({ fn, resolve, reject })
}
```

### 3.4 可操作性检查 (Actionability)
1. 元素必须可见
2. 元素不能被禁用
3. 元素不能被其他元素覆盖
4. 元素必须停止动画
5. 自动滚动元素到可见区域

### 3.5 借鉴建议
- 实现命令队列管理操作序列
- 采用稳定性队列机制
- 可操作性检查适配 AXUIElement

---

## 4. Appium 架构精要

### 4.1 Driver 机制
```
Appium Server
├── BaseDriver (核心抽象层)
│   ├── UiAutomator2Driver (Android)
│   ├── XCUITestDriver (iOS)
│   ├── Mac2Driver (macOS)  ← 重点参考
│   └── 自定义 Driver...
└── Plugin System
```

### 4.2 W3C WebDriver 协议
- HTTP + JSON 通信
- Session 管理 (Capabilities 协商)
- 命令路由表 (METHOD_MAP)

### 4.3 Mac2Driver 结构 (重点)
```
WebDriverAgentMac/
├── WebDriverAgentLib/
│   ├── Commands/      # 命令实现
│   ├── Categories/    # ObjC 扩展
│   ├── Routing/       # 请求路由
│   └── Utilities/     # 工具类
└── WebDriverAgent/    # 主程序
```

### 4.4 借鉴建议
- 采用 HTTP + JSON 协议
- 实现 Session 生命周期管理
- 路由表映射 URL → 命令
- 支持扩展命令 (mobile: 前缀)

---

## 5. Robot Framework 架构精要

### 5.1 关键字类型体系
```python
KeywordImplementation (基类)
├── LibraryKeyword    # 库关键字
├── UserKeyword       # 用户定义关键字
└── InvalidKeyword    # 错误处理
```

### 5.2 嵌入式参数 (独特 DSL)
```robot
*** Keywords ***
User ${user} Selects ${item} From Webshop
    Log    ${user} selected ${item}
```

正则表达式约束:
```robot
Result of ${a:\d+} ${operator:[+-]} ${b:\d+} is ${result}
```

### 5.3 借鉴建议
- 关键字匹配器实现目标匹配
- 嵌入式参数用于 Goal 模板
- Visitor 模式遍历执行计划

---

## 6. EarlGrey 架构精要

### 6.1 同步机制
- **GREYUIThreadExecutor**: `drainUntilIdle` 管理 UI 线程
- **GREYAppStateTracker**: 位掩码追踪多种状态
- **IdlingResource 协议**: 可扩展的空闲资源接口

### 6.2 Matcher 设计
```objc
@protocol GREYMatcher
- (BOOL)matches:(id)item;
- (void)describeTo:(id<GREYDescription>)description;
@end
```

组合器:
- `grey_allOf()` - AND
- `grey_anyOf()` - OR  
- `grey_not()` - NOT

### 6.3 借鉴建议
- IdlingResource 模式和状态追踪
- Matcher 组合与描述能力
- Accessibility 属性优先定位

---

## 7. Detox 架构精要

### 7.1 三进程架构
```
Tester ◄──WebSocket──► Detox Server ◄──WebSocket──► App (DetoxSync)
```

### 7.2 灰盒测试
- 应用内注入监控代码
- 感知内部状态 (不是黑盒猜测)
- 智能等待 idle 而非 sleep

### 7.3 DetoxSync 追踪资源
| 资源类型 | 监控内容 |
|---------|---------|
| Dispatch Queues | 队列工作项 |
| Run Loops | 主线程空闲状态 |
| Timers | NSTimer, CADisplayLink |
| Network | 网络请求状态 |
| Animations | UIView/CA 动画 |
| React Native | JS setTimeout, 桥接通信 |

### 7.4 借鉴建议
- 监控 NSRunLoop idle 状态
- 追踪 NSAnimation/CAAnimation
- 监控 GCD dispatch queues
- 三进程分离架构

---

## 8. 知识库组织最佳实践

### 8.1 Diátaxis 四象限框架
```
┌─────────────────┬─────────────────┐
│   Tutorials     │    How-to       │
│   (学习导向)     │    (目标导向)    │
├─────────────────┼─────────────────┤
│   Explanation   │   Reference     │
│   (理解导向)     │    (信息导向)    │
└─────────────────┴─────────────────┘
```

### 8.2 推荐结构
```
world-knowledge/
├── INDEX.md           # 导航入口
├── SCHEMA.md          # 约束和模板
├── LOG.md             # 变更记录
├── apps/              # 应用知识 (bundle_id, shortcuts)
├── patterns/          # 设计模式
│   ├── element-patterns/   # 元素定位模式
│   ├── flow-patterns/      # 流程模式
│   ├── resilience-patterns/# 容错模式
│   └── ai-patterns/        # AI 相关模式
├── elements/          # 元素类型参考
├── cases/             # 测试用例
├── failures/          # 失败案例
└── raw/               # 原始资料
```

### 8.3 元数据 Frontmatter
```yaml
---
title: "元素定位 - 按钮"
tags: [element, button, click]
confidence: 0.85
platform: macos
last_verified: 2026-04-18
related: [[element-click]], [[button-types]]
---
```

---

## 9. 对 goal-driven-automation 的综合建议

### 9.1 架构层面
1. **采用 Locator 模式** - 延迟求值，执行时才查询 AXUIElement
2. **实现命令队列** - 管理操作序列，支持重试和回滚
3. **设计同步机制** - 借鉴 Detox 的 IdlingResource 模式
4. **协议分离** - HTTP + JSON 接口，便于远程测试

### 9.2 执行层面
1. **可操作性检查** - 元素可见、可点击、未被遮挡
2. **智能等待** - 监控 RunLoop、动画、网络状态
3. **自动重试** - 带超时和退避策略
4. **稳定性队列** - 等待系统稳定后执行

### 9.3 选择器设计
```
# 链式选择器语法提案
role=button >> name="Submit"
app=Safari >> window=main >> role=textfield[title*=URL]
```

### 9.4 知识库结构
- 分离 patterns/ 收集设计模式
- 采用 Diátaxis 四象限组织文档
- Frontmatter 元数据支持搜索和关联

---

## 10. 参考资源

### 官方文档
- [Playwright Docs](https://playwright.dev/docs/intro)
- [Cypress Docs](https://docs.cypress.io)
- [Appium Docs](https://appium.io/docs)
- [Robot Framework User Guide](https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html)
- [EarlGrey API](https://github.com/google/EarlGrey)
- [Detox Docs](https://wix.github.io/Detox/)

### 源码仓库
- microsoft/playwright
- cypress-io/cypress
- appium/appium
- appium/appium-mac2-driver (macOS 专用)
- robotframework/robotframework
- google/EarlGrey
- wix/Detox

### 文档架构
- [Diátaxis Framework](https://diataxis.fr/)
- [LangChain Docs](https://python.langchain.com/docs/)
- [Anthropic Claude Cookbooks](https://github.com/anthropics/anthropic-cookbook)

---

*此文档由 Hermes Agent 自动生成，作为 world-knowledge 知识库的原始输入*
