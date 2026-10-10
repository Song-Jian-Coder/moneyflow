# AGENTS.md — AI 编码助手项目约定

本文件为所有 AI 编码助手提供项目级上下文。当你首次进入此项目时，请先阅读本文件。

## 项目概述

first_moneyflow 是一个采用 **OpenSpec + Superpowers** 双框架体系的 AI 协作开发项目。

## 项目结构

```
first_moneyflow/
├── AGENTS.md              # 本文件 — AI 助手项目约定
├── README.md              # 项目说明
├── openspec/              # OpenSpec 规范层
│   ├── config.yaml        # 项目上下文配置（技术栈、规范、规则）
│   ├── commands.md        # OPSX 工作流命令速查
│   ├── specs/             # 系统规范（Source of Truth）
│   └── changes/           # 活跃变更工作区
└── skills/                # Superpowers 技能层
    ├── using-superpowers/ # 引导技能（入口）
    ├── brainstorming/     # 需求梳理
    ├── writing-plans/     # 计划编写
    └── ...                # 其他技能
```

## 工作流约定

### 1. 规范先行（OpenSpec）

- **任何功能开发前**，先在 `openspec/changes/` 下创建变更目录
- 变更目录包含完整的工件链：`proposal.md` → `specs/` → `design.md` → `tasks.md`
- 使用 `openspec/config.yaml` 中的 `context` 了解项目技术栈和约定
- 使用 `openspec/config.yaml` 中的 `rules` 遵守各工件的编写规则

### 2. 流程纪律（Superpowers）

- **任何创意工作前**，先调用 `brainstorming` 技能梳理需求
- **编写生产代码前**，必须有已批准的设计方案
- **编写生产代码时**，遵循 TDD（测试驱动开发）铁律：无失败测试不写代码
- **遇到 bug 时**，使用 `systematic-debugging` 技能，先定位根因再修复
- **完成任务前**，使用 `verification-before-completion` 技能做最终验证

### 3. 完整开发流程

```
用户需求 → brainstorming（需求梳理，人类批准）
         → openspec propose（生成 proposal/specs/design/tasks）
         → writing-plans（拆解为可执行任务）
         → executing-plans + TDD（逐一执行任务）
         → code-review（代码审查）
         → verification（完成验证）
         → openspec sync + archive（同步规范，归档变更）
```

## 核心规则

1. **不要跳过需求梳理**：即使需求看起来很简单，也先确认理解是否正确
2. **不要跳过测试**：TDD 铁律，没有失败的测试就不能写生产代码
3. **不要直接修改主规范**：所有变更通过 `openspec/changes/` 进行，完成后 sync 合并
4. **保持工件一致**：修改任何工件后，检查上下游工件是否需要同步更新
5. **小步验证**：每个任务完成后立即验证，不要积累未验证的变更

## 技术栈

> 待确定。确定后请同步更新 `openspec/config.yaml` 中的 `context` 字段。

## 代码规范

> 待确定。确定后请同步更新 `openspec/config.yaml` 中的 `rules` 字段。
