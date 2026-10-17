# Tasks

## 1. 清理 Vite 构建链

- [x] 1.1 在 `frontend/` 目录卸载旧依赖：运行 `npm uninstall vite @vitejs/plugin-react react-router-dom axios`，验证 `package.json` 中不再包含这些包
- [x] 1.2 删除 Vite 配置文件：删除 `frontend/vite.config.ts` 和 `frontend/index.html`，验证文件不存在
- [x] 1.3 删除旧源码目录：删除 `frontend/src/` 整个目录（含 `src/main.tsx`、`src/App.tsx`、`src/pages/`、`src/api/`、`src/components/`、`src/types/`、`src/vite-env.d.ts`），验证 `frontend/src/` 不存在

## 2. 初始化 Next.js 项目骨架

- [x] 2.1 在 `frontend/` 目录安装 Next.js 依赖：运行 `npm install next@latest react@latest react-dom@latest`，验证 `package.json` 包含 `next`、`react`、`react-dom`
- [x] 2.2 安装 Tailwind CSS 依赖：运行 `npm install tailwindcss @tailwindcss/postcss postcss`，验证 `package.json` devDependencies 包含这些包
- [x] 2.3 创建 `frontend/next.config.ts`，配置 API rewrites 代理（`/api/*` → `http://localhost:8080/api/*`），验证文件存在且语法正确
- [x] 2.4 创建 `frontend/postcss.config.js`，配置 Tailwind CSS 插件，验证文件存在
- [x] 2.5 创建 `frontend/tailwind.config.ts`，配置 content 路径扫描 `app/` 和 `components/`，验证文件存在
- [x] 2.6 重写 `frontend/tsconfig.json`，适配 Next.js 配置（`moduleResolution: "bundler"`、`jsx: "preserve"`、`paths: {"@/*": ["./*"]}`、`include` 包含 `next-env.d.ts`），验证文件存在
- [x] 2.7 更新 `frontend/package.json` scripts：`dev` → `next dev`、`build` → `next build`、`start` → `next start`，验证 `npm run dev` 能启动 Next.js 开发服务器（显示 ready 消息）

## 3. 创建 App Router 目录结构和基础页面

- [x] 3.1 创建 `frontend/app/globals.css`，添加 `@import "tailwindcss"` 指令，验证文件存在
- [x] 3.2 创建 `frontend/app/layout.tsx`，引入 `globals.css`，设置 HTML lang="zh-CN"、title "MoneyFlow"，验证文件存在
- [x] 3.3 创建 `frontend/app/page.tsx`（首页），迁移原 `Home.tsx` 逻辑：显示标题、Health Check 请求、待办清单链接。使用 `'use client'` 指令，将 `Link` 从 `react-router-dom` 改为 `next/link`，将 API 调用改为新的 `lib/request.ts` 封装。验证首页渲染正常
- [x] 3.4 创建 `frontend/app/not-found.tsx`（404 页面），迁移原 `NotFound.tsx` 逻辑，添加返回首页链接。验证访问不存在路由时显示 404 页面

## 4. 迁移类型定义和 API 封装层

- [x] 4.1 将 `frontend/src/types/api.ts` 复制到 `frontend/types/api.ts`（内容不变），验证文件存在且类型定义完整
- [x] 4.2 将 `frontend/src/types/todo.ts` 复制到 `frontend/types/todo.ts`（内容不变），验证文件存在且类型定义完整
- [x] 4.3 创建 `frontend/lib/request.ts`，基于 `fetch` API 实现统一请求封装（替代 Axios），包含：GET/POST/PUT/DELETE 便捷方法、统一 Content-Type 设置、统一 Result 格式解析（code === 200 成功，其他抛错）、网络错误处理。验证文件存在且 TypeScript 编译无错误
- [x] 4.4 创建 `frontend/lib/todo.ts`，迁移原 `src/api/todo.ts` 的 8 个 API 函数（createTodo、listTodos、getTodo、updateTodo、deleteTodo、toggleTodo、batchComplete、batchDelete），内部实现从 Axios 改为调用 `lib/request.ts`。验证文件存在且 TypeScript 编译无错误

## 5. 迁移 Todo 业务组件

- [x] 5.1 创建 `frontend/components/todo/TodoFilter.tsx`，迁移原组件逻辑，保持筛选条件（状态、优先级、排序字段、排序方向）和回调接口不变。添加 `'use client'` 指令。验证文件存在
- [x] 5.2 创建 `frontend/components/todo/TodoItem.tsx`，迁移原组件逻辑，保持待办项展示（勾选、标题、详情、优先级标签、截止时间、编辑/删除按钮）不变。验证文件存在
- [x] 5.3 创建 `frontend/components/todo/TodoForm.tsx`，迁移原组件逻辑，保持表单功能（标题、详情、优先级选择、截止时间、创建/编辑模式）不变。添加 `'use client'` 指令。验证文件存在
- [x] 5.4 创建 `frontend/components/todo/TodoList.tsx`，迁移原组件逻辑，保持列表展示（全选、批量操作、分页、筛选）不变。添加 `'use client'` 指令。验证文件存在

## 6. 创建 Todo 页面

- [x] 6.1 创建 `frontend/app/todos/page.tsx`，迁移原 `src/pages/Todo.tsx` 逻辑，整合 TodoList、TodoForm 组件，调用 `lib/todo.ts` API 函数。添加 `'use client'` 指令。验证文件存在且 TypeScript 编译无错误

## 7. 构建与启动验证

- [x] 7.1 运行 `cd frontend && npm run build`，验证 Next.js 生产构建成功，无 TypeScript 编译错误，输出 `.next/` 目录
- [x] 7.2 运行 `cd frontend && npm run dev`，验证 Next.js 开发服务器成功启动在 `http://localhost:3000`，首页正常渲染

## 8. 功能验证（需后端运行）

- [x] 8.1 验证首页：访问 `http://localhost:3000`，确认标题显示、Health Check 请求成功、"进入待办清单"链接可点击跳转到 `/todos`
- [x] 8.2 验证待办清单 CRUD：访问 `/todos`，测试创建待办、列表展示、编辑、删除、toggle 完成状态，确认所有操作与迁移前行为一致
- [x] 8.3 验证筛选与分页：测试按状态/优先级筛选、按创建时间/截止时间/优先级排序、翻页功能，确认结果正确
- [x] 8.4 验证批量操作：测试全选、批量完成、批量删除功能，确认操作成功

## Workflow follow-up

- 归档变更：实现完成并通过审查后执行 `openspec archive`
- 同步规范：归档时自动将 delta spec 合并到 `openspec/specs/frontend-framework/spec.md`
- 更新 config.yaml：前端技术栈信息已在 config.yaml 中更新为 Next.js
