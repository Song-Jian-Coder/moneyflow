# OPSX 工作流命令速查

OpenSpec 的 OPSX 工作流提供了一组斜杠命令，用于管理开发全流程。
本文档为命令速查参考，实际使用时通过 AI 编码助手的斜杠命令触发。

## 核心命令（6 个）

| 命令 | 阶段 | 功能 | 说明 |
|------|------|------|------|
| `/opsx:explore` | 探索 | 只讨论、不写文件 | 梳理需求与选型，适合需求模糊时 |
| `/opsx:propose` | 规划 | 一次生成全部工件 | 生成 proposal.md / specs/ / design.md / tasks.md |
| `/opsx:apply` | 执行 | 按 tasks 清单实施代码 | 逐任务执行，内部强制 TDD |
| `/opsx:update` | 修订 | 修订已有规划工件 | 保持工件间一致性 |
| `/opsx:sync` | 同步 | 合并变更规范到主规范库 | 将 delta spec 合并到 specs/ |
| `/opsx:archive` | 归档 | 归档已完成变更 | 自动先 sync，然后封存到 changes/archive/ |

## 扩展命令（6 个）

需要切换到 Expanded profile 后使用：

| 命令 | 功能 |
|------|------|
| `/opsx:new` | 只 scaffold 变更目录，不生成规划内容 |
| `/opsx:continue` | 一次只生成一个工件，边生成边审阅 |
| `/opsx:ff` | 快进：一次生成所有规划工件 |
| `/opsx:verify` | 校验代码是否符合规范 |
| `/opsx:bulk-archive` | 批量归档多个已完成的变更 |
| `/opsx:onboard` | 交互式 15 分钟教程 |

## 工件依赖关系（DAG）

```
proposal.md → specs/ → design.md → tasks.md
     ↑           ↑          ↑           ↑
   BLOCKED → BLOCKED → BLOCKED → BLOCKED
     ↓           ↓          ↓           ↓
   READY → READY → READY → READY
     ↓           ↓          ↓           ↓
    DONE → DONE → DONE → DONE
```

每个工件的三态：
- **BLOCKED**：缺少依赖，无法开始
- **READY**：依赖齐备，可以开始
- **DONE**：文件已存在于磁盘上

关键规则：
- 依赖是 enabler（使能器），不是 gate（门控）
- 可以任意顺序、任意时机组合动作
- 修改任何工件后，检查下游工件是否需要同步更新

## 典型使用场景

### 场景一：需求模糊

```
1. /opsx:explore — 和 AI 讨论技术选型和边界
2. /opsx:propose "一句话需求" — 四份工件一次生成
3. /opsx:update — 对不满意的工件局部修订
4. /opsx:apply — 按 tasks 清单实施代码
5. /opsx:sync — 沉淀通用规范
6. /opsx:archive — 归档已完成变更
```

### 场景二：需求明确

```
1. /opsx:propose "一句话需求" — 直接生成全部工件
2. /opsx:apply — 实施代码
3. /opsx:archive — 归档完成
```

### 场景三：中途发现设计偏差

```
1. /opsx:apply — 执行到一半
2. 发现 design 有遗漏
3. /opsx:update — 修订 design.md
4. /opsx:apply — 继续执行（自动基于新 design）
```

## 与 Superpowers 技能的对应关系

| OPSX 命令 | 对应 Superpowers 技能 | 协作方式 |
|-----------|----------------------|----------|
| `/opsx:explore` | brainstorming | 需求梳理阶段互补 |
| `/opsx:propose` | writing-plans | 生成 proposal + tasks |
| `/opsx:apply` | executing-plans + TDD | 按任务执行，强制测试 |
| `/opsx:update` | — | OpenSpec 独有，修订工件 |
| `/opsx:sync` | finishing-a-development-branch | 合并规范 + 分支收尾 |
| `/opsx:archive` | verification-before-completion | 归档前最终验证 |

## 项目配置

项目上下文配置在 `openspec/config.yaml` 中：

- `context`：注入到所有工件的通用背景（技术栈、代码规范等）
- `rules`：按工件类型设置的硬性约束

配置示例：

```yaml
schema: spec-driven

context: |
  技术栈：TypeScript、React、Node.js
  接口约定：RESTful，JSON 响应
  测试：Vitest 跑单测，Playwright 跑 e2e

rules:
  proposal:
    - 必须包含回滚方案
  specs:
    - Scenario 用 Given/When/Then 格式
  design:
    - 必须包含架构图
  tasks:
    - 每个任务 2-5 分钟粒度
```
