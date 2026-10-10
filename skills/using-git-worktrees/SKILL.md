---
name: using-git-worktrees
description: "Use when starting implementation work after a design has been approved, before writing any production code."
---

# Using Git Worktrees — Git Worktree 隔离开发

本技能利用 Git Worktree 创建隔离的开发环境，确保 AI 的代码修改不会直接污染主分支。

## 核心原则

> 在隔离环境中开发，不污染主分支。每个功能或变更在独立的工作目录中进行。

## Iron Law — 铁律

> **不在主分支上直接开发。所有实现工作在独立分支/worktree 中进行。**

## When to Use — 使用时机

- 设计方案已获批准，准备开始实现
- 需要同时开发多个功能且互不干扰
- 需要一个干净的环境进行实验

## What is Git Worktree?

Git Worktree 允许在同一个仓库中同时检出多个分支到不同目录：

```
project/
├── main/              # 主分支（稳定代码）
├── worktree-feature-a/ # 功能 A 的独立工作目录
└── worktree-feature-b/ # 功能 B 的独立工作目录
```

## Process — 核心流程

### 1. 创建 Worktree

```bash
# 基于主分支创建新的工作分支和目录
git worktree add ../worktree-feature-name -b feature/feature-name

# 进入工作目录
cd ../worktree-feature-name
```

### 2. 在隔离环境中开发

- 所有代码修改在 worktree 目录中进行
- 正常进行 TDD 开发
- 主分支代码不受影响

### 3. 同步主分支变更（如需要）

```bash
# 如果主分支有更新，在 worktree 中合并
git merge main
```

### 4. 完成后处理

开发完成后，使用 `finishing-a-development-branch` 技能处理分支收尾：

- 合并到主分支
- 创建 Pull Request
- 或删除（如果方案被放弃）

### 5. 清理 Worktree

```bash
# 合并完成后清理 worktree
cd project/main
git worktree remove ../worktree-feature-name
```

## Worktree 命名规范

| 场景 | 分支名 | Worktree 目录名 |
|------|--------|-----------------|
| 新功能 | `feature/feature-name` | `worktree-feature-name` |
| Bug 修复 | `fix/bug-description` | `worktree-fix-description` |
| OpenSpec 变更 | `change/change-name` | `worktree-change-name` |

## Red Flags — 识别错误使用方式的信号

| 想法 | 现实 |
|------|------|
| "直接在主分支上改吧，反正就一个小功能" | 小功能也可能引入需要回滚的问题 |
| "worktree 太麻烦了" | 比 git stash + checkout 更干净 |
| "我会在合并前清理的" | 先建好隔离环境，后面不用记得清理 |

## Verification Checklist

- [ ] 是否已创建独立的 worktree？
- [ ] 是否在 worktree 目录中而非主分支上开发？
- [ ] 是否已确认 worktree 基于最新的主分支代码？
