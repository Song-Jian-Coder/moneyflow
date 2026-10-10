---
name: writing-skills
description: "Use when creating a new Agent skill, editing an existing skill, or deploying a skill before verification."
---

# Writing Skills — 元技能：编写新技能

本技能指导如何编写高质量的 SKILL.md 文件。当你需要为项目创建自定义技能时，使用本技能。

## 核心原则

> 好的技能文件改变 AI 的行为方式，而非执行计算。每个技能应该独立可理解，包含完整的流程指引。

## Iron Law — 铁律

> **description 字段只写触发条件，不概括工作流内容。否则 AI 会走捷径只看描述而不读全文。**

## SKILL.md 文件结构

### 必需部分

```markdown
---
name: skill-name-here
description: "Use when [触发条件描述]. Use before [时机说明]."
---

# Skill Name — 技能中文名

## 核心原则
> [一句话核心原则]

## Iron Law — 铁律
> [一行绝对规则]

## Process — 核心流程
[逐步工作流]

## Verification Checklist
- [ ] [检查项 1]
- [ ] [检查项 2]
```

### 可选但推荐的部分

| 部分 | 用途 | 何时需要 |
|------|------|----------|
| When to Use | 详细使用时机 | 触发条件复杂时 |
| Red Flags 表 | 识别违规信号 | 纪律执行类技能 |
| Common Rationalizations 表 | 反驳常见借口 | 纪律执行类技能 |
| Good/Bad 示例 | 对比示例 | 有明确输出格式要求时 |

## YAML Frontmatter 规范

### name 字段

- 只允许字母、数字、连字符
- 示例：`test-driven-development`、`writing-plans`

### description 字段

```
❌ 错误：概括了工作流
description: "A skill that helps write tests using TDD methodology by following RED-GREEN-REFACTOR..."

✅ 正确：只描述触发条件
description: "Use when writing or modifying production code that needs tests."
```

## 设计原则

| 原则 | 说明 |
|------|------|
| **描述只写触发条件** | description 绝不概括工作流，否则 AI 走捷径 |
| **用人类语言** | 技能是给人类和 AI 共同阅读的文档 |
| **包含反理性化机制** | 纪律类技能必须有 Iron Law + Red Flags 表 |
| **附带可执行验证** | Checklist 比"请确保..."更可靠 |
| **用 Good/Bad 示例** | 对比示例比抽象描述更有效 |
| **独立可理解** | 每个技能应该能在不读其他技能的情况下被理解 |

## 最小可行技能模板

```markdown
---
name: my-new-skill
description: "Use when [具体触发条件]."
---

# My New Skill — 技能名称

## 核心原则

> [一句话核心原则]

## Iron Law — 铁律

> [一行绝对规则，不允许例外]

## Process — 核心流程

1. [步骤一：做什么，怎么做]
2. [步骤二：做什么，怎么做]
3. [步骤三：做什么，怎么做]

## Red Flags — 识别跳过流程的信号

| 想法 | 现实 |
|------|------|
| "[常见借口]" | "[反驳]" |

## Verification Checklist

- [ ] [检查项 1]
- [ ] [检查项 2]
- [ ] [检查项 3]
```

## 文件组织

每个技能一个目录：

```
skills/
└── my-new-skill/
    ├── SKILL.md              # 主技能文件（必需）
    ├── references/           # 补充参考文档（可选）
    │   └── related-info.md
    └── scripts/              # 辅助脚本（可选）
        └── helper.sh
```

## Verification Checklist

- [ ] name 是否只包含字母、数字、连字符？
- [ ] description 是否只描述了触发条件（而非工作流内容）？
- [ ] 是否包含 Iron Law（铁律）？
- [ ] 是否包含完整的 Process（核心流程）？
- [ ] 是否包含 Verification Checklist？
- [ ] 技能是否可以独立理解（不依赖其他技能）？
