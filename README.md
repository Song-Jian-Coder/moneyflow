# first_moneyflow

基于 **OpenSpec + Superpowers** 体系构建的 AI 协作开发项目。

## 开发体系

本项目采用两套互补的 AI 编码框架：

### OpenSpec — 规范驱动开发（SDD）

> 先约定，后编码。

在代码和需求之间建立一层可版本化、可审阅的规范，让人和 AI 先对齐再动手。

- `openspec/config.yaml` — 项目上下文配置（技术栈、代码规范、工件规则）
- `openspec/specs/` — 系统规范（Source of Truth）
- `openspec/changes/` — 活跃变更工作区，每个变更一个独立目录
- `openspec/commands.md` — OPSX 工作流命令速查

**核心工作流（OPSX）：**

```
explore（探索需求）→ propose（生成提案）→ apply（执行实施）
    ↕ update（修订工件）    ↕ sync（同步规范）    ↕ archive（归档变更）
```

### Superpowers — AI 技能框架

> 给 AI 装上开发方法论，让它按流程干活。

将资深工程师的工作习惯固化为可调用的技能文件，AI 在对应场景下自动遵循工程纪律。

- `skills/` — 技能目录，每个子目录包含一个 `SKILL.md` 技能定义
- 12 个核心技能覆盖：需求梳理、计划编写、TDD、调试、代码审查、分支管理

**核心工作流：**

```
brainstorming → writing-plans → executing-plans → test-driven-development
    → code-review → verification → finishing
```

## 快速开始

1. 阅读 `AGENTS.md` 了解 AI 编码助手的项目约定
2. 阅读 `openspec/config.yaml` 了解项目上下文
3. 新功能开发从 `/opsx:explore` 或 `/opsx:propose` 开始
4. AI 助手会自动加载 `skills/` 中的技能来指导工作流

## 项目状态

- 技术栈：待确定
- 开发框架已初始化，可开始使用 OpenSpec + Superpowers 工作流
