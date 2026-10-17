# Proposal

## Why

项目当前只有 OpenSpec + Superpowers 的框架目录结构，没有任何可运行的代码骨架。所有后续业务功能开发都需要一个完整的前后端开发框架作为基座——包括 Maven 多模块后端、React 前端、统一的基础设施（响应封装、异常处理、API 文档等）。现在搭建这个基座，让后续所有功能开发都有统一的规范和约定可以遵循。

## What Changes

- 新建 `backend/` 目录，包含 Maven 多模块项目（parent + common + model + dao + service + web 共 6 个模块）
- 新建 `frontend/` 目录，包含 React + Vite + TypeScript 前端项目骨架
- 后端基于 Java 17 + Spring Boot 3.x，数据层使用 MySQL 8.0 + MyBatis-Plus
- 后端内置基础能力：统一响应封装（`Result<T>`）、全局异常处理、参数校验（Validation）、API 文档（SpringDoc OpenAPI）、CORS 跨域配置、多环境配置（dev/prod）、Lombok
- 前端内置基础能力：React Router 路由、Axios HTTP 客户端（含请求/响应拦截器）、API 对接基础配置
- 认证鉴权不在本次范围内，留待后续变更

## Capabilities

### New Capabilities

- `backend-framework`: 后端 Maven 多模块脚手架及基础设施——包含分层模块结构（common/model/dao/service/web）、统一响应封装、全局异常处理、参数校验、API 文档、CORS 配置、多环境配置等开箱即用的基础能力
- `frontend-framework`: 前端 React + Vite + TypeScript 脚手架及基础设施——包含项目骨架、路由配置、HTTP 客户端封装、API 对接基础配置等开箱即用的基础能力

### Modified Capabilities

（无已有能力被修改）

## Impact

- **代码结构**：项目根目录新增 `backend/` 和 `frontend/` 两个主要目录，原有 `openspec/`、`skills/`、`.qoder/` 不受影响
- **依赖**：引入 Spring Boot 3.x、MyBatis-Plus、SpringDoc、Lombok、MySQL Connector 等后端依赖；引入 React、Vite、TypeScript、React Router、Axios 等前端依赖
- **构建**：后端通过 `mvn clean package` 构建，前端通过 `npm run build` 构建，两者独立
- **运行环境**：需要 JDK 17+、Maven 3.6+、Node.js 18+、MySQL 8.0+
- **API**：后端启动后提供 RESTful API 基础结构，前端通过代理配置对接
- **回滚方案**：本次变更为纯新增（新建目录和文件），不修改任何已有文件。回滚只需删除 `backend/` 和 `frontend/` 目录即可完全恢复
