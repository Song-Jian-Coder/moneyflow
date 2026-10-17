# AGENTS.md — AI 编码助手项目约定

本文件为所有 AI 编码助手提供项目级上下文。当你首次进入此项目时，请先阅读本文件。

## 项目概述

MoneyFlow 是一个采用 **OpenSpec + Superpowers** 双框架体系的 AI 协作开发项目。根仓库作为 **AIWorkSpace**，通过 Git Submodule 管理前后端子仓库。

## 项目结构

```
moneyflow/                     # 根仓库（AIWorkSpace）
├── AGENTS.md                  # 本文件 — AI 助手项目约定
├── README.md                  # 项目说明
├── .gitmodules                # Submodule 注册表
├── backend/                   # [submodule] 后端仓库
│   └── moneyflow_backend.git  # https://github.com/Song-Jian-Coder/moneyflow_backend.git
├── frontend/                  # [submodule] 前端仓库
│   └── moneyflow_frontend.git # https://github.com/Song-Jian-Coder/moneyflow_frontend.git
├── openspec/                  # OpenSpec 规范层
│   ├── config.yaml            # 项目上下文配置（技术栈、规范、规则）
│   ├── commands.md            # OPSX 工作流命令速查
│   ├── specs/                 # 系统规范（Source of Truth）
│   └── changes/               # 活跃变更工作区
└── skills/                    # Superpowers 技能层
    ├── using-superpowers/     # 引导技能（入口）
    ├── brainstorming/         # 需求梳理
    ├── writing-plans/         # 计划编写
    └── ...                    # 其他技能
```

## 子仓库说明

### `backend/` — 后端服务

- **仓库地址**：https://github.com/Song-Jian-Coder/moneyflow_backend.git
- **技术栈**：Java 17 + Spring Boot 3.2.5 + Maven 多模块 + Spring Data JPA
- **模块结构**：
  - `moneyflow-common` — 通用工具（异常类、统一响应 Result<T>）
  - `moneyflow-model` — 数据模型（Entity、DTO、VO）
  - `moneyflow-dao` — 数据访问层（Repository）
  - `moneyflow-service` — 业务逻辑层（Service 接口 + Impl）
  - `moneyflow-web` — 控制层（Controller、配置类、启动入口）
- **启动方式**：`cd backend && mvn spring-boot:run '-Dspring-boot.run.profiles=dev' -pl moneyflow-web`
- **默认端口**：8080
- **数据库**：开发环境使用 H2 内存数据库，生产环境使用 MySQL 8.0

### `frontend/` — 前端应用

- **仓库地址**：https://github.com/Song-Jian-Coder/moneyflow_frontend.git
- **技术栈**：Next.js (App Router) + TypeScript + Tailwind CSS
- **目录结构**：
  - `app/` — 文件系统路由（页面 + 布局 + 加载状态）
  - `components/` — 可复用 UI 组件
  - `lib/` — API 请求封装、工具函数
  - `types/` — TypeScript 类型定义
- **启动方式**：`cd frontend && npm install && npm run dev`
- **默认端口**：3000
- **代理配置**：Next.js rewrites 将 `/api` 请求代理到 `http://localhost:8080`

### Submodule 管理

```bash
# 首次克隆后初始化子仓库
git submodule update --init --recursive

# 拉取子仓库最新代码
git submodule update --remote --merge

# 进入子仓库提交后，在根仓库记录变更
cd backend && git add . && git commit -m "feat: xxx"
cd .. && git add backend && git commit -m "chore: update backend submodule"
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

- **后端**：Java 17 + Spring Boot 3.2.5 + Maven 多模块 + Spring Data JPA + H2/MySQL
- **前端**：Next.js (App Router) + TypeScript + Tailwind CSS
- **规范**：OpenSpec + Superpowers 双框架
- **接口文档**：SpringDoc OpenAPI（Swagger UI 自动生成）

## 技术选型决策记录

以下选型基于项目约束条件做出，所有后续开发必须遵循。

### 项目约束条件

| 约束 | 说明 |
|------|------|
| 团队规模 | 3 人，全员熟悉 React，有充足 Spring Boot 开发经验 |
| 项目周期 | 8 周 MVP，必须上线 |
| 部署环境 | Vercel 或 Docker |
| 开发模式 | AI 辅助开发为主，选择 AI 生成质量最高的方案 |
| 项目定位 | 线上可持续运行迭代的长期项目，非一次性交付 |

### 后端选型：Spring Boot（保持）

**对比方案**：Spring Boot vs Next.js API Routes

| 维度 | Spring Boot | Next.js API |
|------|-------------|-------------|
| AI 生成质量 | ★★★★☆ | ★★★★★ |
| 框架约定强度 | ★★★★★ | ★★★☆☆ |
| 生态成熟度 | ★★★★★ | ★★★★☆ |
| 部署敏捷度 | ★★★☆☆ | ★★★★★ |
| 团队学习成本 | ★★★★★（已有经验） | ★★★★★ |
| 长期可维护性 | ★★★★★ | ★★☆☆☆ |
| 复杂业务承载力 | ★★★★★ | ★★★☆☆ |

**决策理由**：
- 团队有充足 Spring Boot 经验，学习成本为零，可直接产出
- 长期迭代项目需要分层架构保护代码质量（100+ API 时依然清晰）
- Spring Boot 生态（事务管理、安全框架、消息队列、缓存）支撑复杂业务演进
- 已有后端骨架和 Todo 模块实现，有效资产不浪费
- Next.js API 在简单 CRUD 场景开发快，但长期迭代会积累技术债

### 前端选型：Next.js（从 Vite 迁移）

**对比方案**：Vite + React vs Next.js

| 维度 | Vite + React | Next.js |
|------|-------------|--------|
| AI 生成质量 | ★★★★☆ | ★★★★★ |
| 框架约定强度 | ★★★★☆ | ★★★★★ |
| 生态成熟度 | ★★★★★ | ★★★★★ |
| 部署敏捷度 | ★★★★☆ | ★★★★★ |
| 团队学习成本 | ★★★★★（已有经验） | ★★★★☆ |
| SSR/SEO | ★★☆☆☆（纯 SPA） | ★★★★★ |

**决策理由**：
- 项目后期需要 SEO 界面（如公开的消费指南、理财知识等页面）
- Next.js 内置 SSR/SSG/ISR，天然支持 SEO 场景
- 后期从 Vite 迁移到 Next.js 成本较高（路由重构、构建配置、部署流程全变），不如早期一步到位
- 文件系统路由在页面增多时比手动配置 React Router 更易维护
- 团队全员熟悉 React，Next.js 是 React 生态的自然延伸，学习成本低
- 与 Spring Boot 后端保持前后端分离架构，API 代理配置方式相近

### 数据访问层：Spring Data JPA（已迁移）

**对比方案**：Spring Data JPA vs MyBatis-Plus

**决策理由**：
- 项目后续会有多表关联查询密集的业务模块（如账单-分类-标签-用户）
- Spring Data JPA 提供对象关系映射、级联查询、JOIN FETCH 等能力
- MyBatis-Plus 优势在单表 CRUD，多表查询需退化到手写 SQL
- 已完成从 MyBatis-Plus 到 Spring Data JPA 的迁移

### 部署方案（待定）

```
方案 A：纯 Docker（完全控制、成本敏感）
  Next.js Container (:3000) + Spring Boot Container (:8080) + PostgreSQL

方案 B：Vercel + 云服务（少运维、快速上线）
  Vercel (前端) + Railway/Fly.io (Spring Boot JAR) + PostgreSQL (托管)
```

> 部署方案待确定后补充具体配置。

### 禁止动作（除非走 propose 流程推翻）

- ❌ 把后端切到 Next.js API Routes / Server Actions（仅适用于短周期 MVP，不适用本项目长期场景）
- ❌ 把前端切到 Vue / Nuxt / Svelte 等非 React 系（团队学习成本归零原则）
- ❌ 用“AI 生成质量更高”单一理由推翻已锁定栈（须同时满足长期工程性维度）

### 何时需要重新评估

触发任一条件，必须发起 `/openspec-propose` 重新评审本节：

- 业务定位变更（如转向短周期试错型项目）
- 团队人员结构剧烈变动（如 Spring Boot / React 经验流失）
- 出现明确性能 / 可维护性瓶颈，且现有栈无法通过常规手段解决

## 代码规范

- 前后端字段命名统一使用驼峰法（camelCase）
- 后端统一响应格式：`Result<T>`（code / message / data）
- 业务异常通过 `BusinessException` 抛出，HTTP 200 + 自定义 code
- 详细规范参见 `openspec/specs/` 下各模块 Spec
