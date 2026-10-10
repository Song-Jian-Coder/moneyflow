---
name: finishing-a-development-branch
description: "Use when all tasks in a development branch are complete and you need to merge, create a PR, or discard the branch."
---

# Finishing a Development Branch — 分支收尾

本技能处理开发分支完成后的收尾工作：合并、创建 PR、或丢弃。

## 核心原则

> 分支开发的最后一步和第一步同样重要。干净的收尾确保代码历史可追溯、可审计。

## Iron Law — 铁律

> **不留下无主的开发分支。每个分支必须有明确的结局：合并、PR、或丢弃。**

## When to Use — 使用时机

- 所有任务已完成并通过验证
- 代码审查已通过
- OpenSpec 变更的 apply 阶段完成
- 需要决定是否继续某个实验性分支

## Process — 核心流程

### 1. 最终验证

在进入收尾前，确认：

- [ ] 所有任务已完成（对照 tasks.md）
- [ ] 所有测试通过
- [ ] 代码审查已完成并通过
- [ ] verification-before-completion 已通过

### 2. 选择结局

| 结局 | 适用场景 | 操作 |
|------|----------|------|
| **合并到主分支** | 功能完成，审查通过 | 直接 merge 或 squash merge |
| **创建 Pull Request** | 需要他人审查或团队流程要求 | 创建 PR，等待审批 |
| **丢弃分支** | 方案被否决或不再需要 | 删除分支和 worktree |

### 3. 合并操作

```bash
# 切回主分支
git checkout main

# 合并开发分支（推荐 squash merge 保持历史整洁）
git merge --squash feature/feature-name
git commit -m "feat: 实现 XX 功能

- 实现了 XX 核心逻辑
- 添加了 XX 测试覆盖
- 关联 OpenSpec 变更: change-name"

# 清理 worktree
git worktree remove ../worktree-feature-name
git branch -d feature/feature-name
```

### 4. OpenSpec 同步

分支合并后，执行 OpenSpec 的收尾操作：

- `openspec sync`：将变更中的规范合并到主规范库
- `openspec archive`：归档已完成的变更

### 5. 更新状态

- 更新 tasks.md 中所有任务为已完成
- 更新 CHANGELOG（如果项目有维护）
- 关闭相关的 Issue 或任务卡片

## Red Flags — 识别不良收尾的信号

| 想法 | 现实 |
|------|------|
| "先不合并，让我再改改" | 如果验证已通过，就应该合并 |
| "分支留着以后参考" | 用 git log 或 archive 代替保留分支 |
| "直接 push 到 main 吧" | 跳过审查和验证的直接 push 是危险的 |

## Verification Checklist

- [ ] 是否已完成最终验证（所有测试通过、审查通过）？
- [ ] 是否已选择明确的结局（合并/PR/丢弃）？
- [ ] 合并后是否清理了 worktree 和分支？
- [ ] 是否已执行 OpenSpec sync + archive？
- [ ] 是否已更新相关状态文档？
