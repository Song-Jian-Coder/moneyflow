# Proposal

## Why

项目后期需要 SEO 界面（如公开的消费指南、理财知识等页面），当前 Vite + React 纯 SPA 架构无法支持服务端渲染（SSR）和静态生成（SSG）。后期从 Vite 迁移到 Next.js 的成本会随业务模块增多而显著增加（路由重构、构建配置、部署流程全变），不如在 MVP 早期一步到位迁移到 Next.js，为后续 SEO 场景和长期迭代奠定基础。

## What Changes

- **移除** Vite 构建工具链（`vite`、`@vitejs/plugin-react`、`vite.config.ts`、`index.html`）
- **移除** React Router（`react-router-dom`），替换为 Next.js 文件系统路由（App Router）
- **移除** Axios HTTP 客户端，替换为 `fetch` API（Next.js 内置，Server Components 不支持 Axios）
- **新增** Next.js 14+（App Router）作为前端框架，提供文件系统路由、SSR/SSG/ISR 能力
- **新增** Tailwind CSS 作为样式方案（替代当前内联 style）
- **新增** `next.config.ts` 配置 API 代理（rewrites 替代 Vite proxy）
- **重构** 目录结构：`src/pages/` → `app/`，`src/api/` → `lib/`，`src/components/` → `components/`
- **迁移** 所有现有页面组件（Home、Todo、NotFound）和 Todo 业务组件（TodoList、TodoItem、TodoForm、TodoFilter）到 Next.js 目录结构
- **迁移** TypeScript 类型定义（types/）保持不变

## Capabilities

### New Capabilities

（无新增能力）

### Modified Capabilities

- `frontend-framework`：框架核心从 Vite 迁移到 Next.js，影响以下需求：
  - "React + Vite + TypeScript 项目结构" → 替换为 "Next.js + TypeScript + Tailwind CSS 项目结构"，开发服务器端口从 5173 改为 3000，构建产物从 `dist/` 改为 `.next/`
  - "路由配置" → 从 React Router 手动配置替换为 Next.js 文件系统路由（App Router）
  - "HTTP 客户端封装" → 从 Axios 替换为基于 `fetch` 的封装（兼容 Server Components）
  - "API 代理配置" → 从 Vite proxy 替换为 Next.js rewrites

## Impact

- **前端子仓库**（`frontend/`）：全面重构，所有文件需要迁移或替换
- **后端 API**：零影响，后端 REST API 接口路径和请求/响应格式不变
- **根仓库**：AGENTS.md 和 config.yaml 中的前端技术栈描述已更新
- **依赖变更**：
  - 移除：`vite`、`@vitejs/plugin-react`、`react-router-dom`、`axios`
  - 新增：`next`、`tailwindcss`、`@tailwindcss/postcss`、`postcss`
  - 保留：`react`、`react-dom`、`typescript`
- **部署方式**：从纯静态文件部署变为需要 Node.js 运行时（或导出纯静态 `next export`）

## 回滚方案

迁移前在 `frontend/` 目录创建 Git 分支保留 Vite 版本。如果迁移后发现功能异常或团队无法适应 Next.js，可通过 Git 回退到迁移前的提交，恢复 Vite 构建链。后端 API 不受影响，无需回滚。
