---
name: using-superpowers
description: "Use when starting any new session, before any creative or implementation work. This is the bootstrap skill that activates all other skills."
---

# Using Superpowers — 引导技能

本技能是整个 Superpowers 体系的入口点。它在每次会话开始时自动激活，确保 AI 在执行任何任务前先检查并调用相关技能。

## 核心原则

> 在任何响应或操作之前，先检查是否有相关 Skill 可以调用。过程类 Skill 优先于实现类 Skill。

## 启动流程

1. **检查任务类型**：分析用户请求属于哪类工作
2. **匹配技能**：根据任务类型查找对应的 Skill
3. **按优先级调用**：过程类技能（brainstorming, writing-plans）优先于实现类技能
4. **遵循铁律**：每个被调用的技能的 Iron Law 不可违反

## 任务分类与技能路由

```
收到任务
├─ 涉及创建/修改功能？
│   ├─ 是 → 调用 brainstorming（需求梳理）
│   └─ 否 ↓
├─ 是 bug 修复？
│   ├─ 是 → 调用 systematic-debugging（系统化调试）
│   └─ 否 ↓
├─ 需要写/改生产代码？
│   ├─ 是 → 必须先有失败测试（TDD）
│   └─ 否 ↓
├─ 需要审查代码？
│   ├─ 是 → 调用 requesting-code-review
│   └─ 否 ↓
└─ 其他任务 → 直接执行
```

## Red Flags — 识别跳过技能的信号

| 想法 | 现实 |
|------|------|
| "这只是个简单问题，不需要技能" | 简单问题也可能需要 TDD 或调试技能 |
| "用户很着急，直接写代码吧" | 仓促导致更多 bug，反而更慢 |
| "我已经知道该怎么做了" | 知道怎么做不等于可以跳过流程 |
| "这次情况特殊" | 每次都觉得自己是特殊情况 |

## Iron Law — 铁律

> **没有检查可用技能之前，不开始任何实质性工作。**

违反字面意思就是违反精神。不存在"例外情况"。

## Verification Checklist

- [ ] 是否已分析任务类型？
- [ ] 是否已匹配并调用相关技能？
- [ ] 是否已阅读并理解被调用技能的流程？
- [ ] 是否在开始工作前完成了技能要求的前置步骤？
