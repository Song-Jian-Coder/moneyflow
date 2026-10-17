# Proposal

## Why

框架基座已搭建完成（后端 Maven 多模块 + 前端 React+Vite），但尚无业务功能。Todo 待办清单是 MoneyFlow 的第一个业务模块，用于验证框架基础设施的完整能力（统一响应、异常处理、参数校验、MyBatis-Plus ORM、前后端联调），同时为用户提供核心的待办管理能力。

## What Changes

- **新增** Todo 待办 CRUD 全链路：实体、Mapper、Service、Controller 共 8 个 REST API
- **新增** 数据库表 `todo_item`（H2 dev / MySQL prod）
- **新增** 前端 Todo 页面：列表、创建、编辑、删除、状态切换、批量操作
- **新增** 前端 API 层：`api/todo.ts` 封装所有 Todo 相关请求
- **新增** 前端路由 `/todos` 指向 Todo 页面

## Capabilities

### New Capabilities

无。Todo 能力规范已存在于主规范库 `openspec/specs/todo/spec.md`（8 项需求、36 个场景）。本次变更为实现该规范，不修改规范本身，设置 `skip_specs: true`。

### Modified Capabilities

无。

## Impact

| 影响范围 | 说明 |
|----------|------|
| **后端模块** | moneyflow-model（实体/DTO/VO）、moneyflow-dao（Mapper）、moneyflow-service（Service）、moneyflow-web（Controller） |
| **前端目录** | `frontend/src/pages/Todo.tsx`、`frontend/src/api/todo.ts`、`frontend/src/types/todo.ts`、`App.tsx`（新增路由） |
| **数据库** | 新增 `todo_item` 表，dev 环境 H2 自动建表，prod 环境需提供 DDL 脚本 |
| **API 路径** | 新增 `/api/todos/**` 共 8 个端点 |
| **依赖** | 无新增第三方依赖（MyBatis-Plus、Validation 已在框架阶段引入） |

## 回滚方案

本次变更为纯新增，不涉及已有代码修改（仅 `App.tsx` 新增一条路由）。回滚方式：
- 后端：删除 `todo` 相关的 Java 文件和数据库表
- 前端：删除 `Todo.tsx`、`todo.ts`、`types/todo.ts`，移除 `App.tsx` 中的路由
