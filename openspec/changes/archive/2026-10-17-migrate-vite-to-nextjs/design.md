# Design

## Context

当前前端基于 Vite + React 19 + React Router + Axios + TypeScript，位于 `frontend/` 子仓库。后端为 Spring Boot REST API，前后端通过 `/api` 前缀交互。迁移目标是 Next.js App Router，为后续 SEO 场景做准备，同时保持与后端 API 的对接方式不变。

现有前端资产：
- 3 个页面组件：Home、Todo、NotFound
- 4 个 Todo 业务组件：TodoList、TodoItem、TodoForm、TodoFilter
- 1 个 API 封装层：Axios request 实例 + todo API 函数
- 2 个类型定义文件：api.ts、todo.ts

## Goals / Non-Goals

**Goals:**
- 将前端构建框架从 Vite 迁移到 Next.js 14+（App Router）
- 引入 Tailwind CSS 替代内联 style
- 保持所有现有功能（Todo CRUD、筛选、分页、批量操作）完全不变
- 保持与后端 API 的对接方式兼容（`/api` 前缀代理）
- 目录结构符合 Next.js 约定

**Non-Goals:**
- 不实现 SSR/SSG/ISR 功能（后续需求再添加）
- 不新增前端页面或业务功能
- 不修改后端 API
- 不引入状态管理库（如 Zustand、Redux）

## Decisions

### 决策 1：使用 Next.js App Router（而非 Pages Router）

**理由**：App Router 是 Next.js 13+ 的推荐方向，支持 React Server Components、嵌套布局、自动代码分割。Pages Router 进入维护模式，新项目不应选择。

**替代方案**：Pages Router — 更成熟稳定，但已是过去式，社区资源向 App Router 倾斜。

### 决策 2：使用 `fetch` 封装替代 Axios

**理由**：Next.js Server Components 中不能使用 Axios（依赖浏览器 API）。统一使用 `fetch` 封装可以同时兼容 Server Components 和 Client Components，减少依赖。

**替代方案**：保留 Axios 仅用于 Client Components — 增加复杂度，且 Server Components 无法使用。

### 决策 3：引入 Tailwind CSS

**理由**：当前代码大量使用内联 `style={{}}`，可维护性差。Tailwind CSS 是 Next.js 生态的默认样式方案，`create-next-app` 原生支持。

**替代方案**：CSS Modules — 可行但需要额外配置文件映射，Tailwind 更简洁。

### 决策 4：Todo 页面使用 Client Components

**理由**：Todo 页面需要大量客户端交互（状态管理、表单、筛选、分页），使用 `'use client'` 指令标记为客户端组件是最直接的迁移路径，行为与当前 Vite SPA 一致。

**替代方案**：将数据获取移到 Server Components — 需要重构组件架构，增加迁移复杂度，当前阶段不值得。

### 决策 5：目录结构设计

```
frontend/
├── app/                          # Next.js App Router
│   ├── layout.tsx                # 根布局（HTML 结构、全局样式）
│   ├── page.tsx                  # 首页（原 Home.tsx）
│   ├── not-found.tsx             # 404 页面（原 NotFound.tsx）
│   └── todos/
│       └── page.tsx              # 待办清单页（原 Todo.tsx）
├── components/                   # 可复用组件
│   └── todo/
│       ├── TodoList.tsx
│       ├── TodoItem.tsx
│       ├── TodoForm.tsx
│       └── TodoFilter.tsx
├── lib/                          # 工具函数和 API 封装
│   └── request.ts                # fetch 封装（替代 api/request.ts）
├── types/                        # TypeScript 类型定义（保持不变）
│   ├── api.ts
│   └── todo.ts
├── next.config.ts                # Next.js 配置（rewrites 代理）
├── tailwind.config.ts            # Tailwind 配置
├── postcss.config.js             # PostCSS 配置
├── tsconfig.json                 # TypeScript 配置
└── package.json
```

## Risks / Trade-offs

- **[风险] Server Components 默认行为导致组件报错** → 所有需要 `useState`/`useEffect` 的组件必须添加 `'use client'` 指令。迁移时逐一检查，确保所有交互组件正确标记。
- **[风险] Tailwind 迁移内联样式的工作量** → 当前 Todo 组件使用大量内联 style，需逐一转换为 Tailwind class。策略：先迁移功能（保留内联 style），再逐步替换为 Tailwind。
- **[取舍] 不使用 SSR/SSG** → 迁移后所有页面仍为客户端渲染（CSR），与当前行为一致。SSR/SSG 能力预留但不启用，避免增加迁移复杂度。
- **[取舍] 保留 Axios vs 切换 fetch** → 选择 fetch 以兼容 Server Components，但需要重写 request 封装逻辑。工作量可控（约 30 行代码）。

## 关键接口定义

### `lib/request.ts` — HTTP 客户端封装

```typescript
// 统一的请求封装函数
export async function request<T>(url: string, options?: RequestInit): Promise<Result<T>>

// 便捷方法
export const api = {
  get<T>(url: string, params?: Record<string, string>): Promise<Result<T>>
  post<T>(url: string, body: unknown): Promise<Result<T>>
  put<T>(url: string, body: unknown): Promise<Result<T>>
  delete<T>(url: string): Promise<Result<T>>
}
```

### `next.config.ts` — API 代理配置

```typescript
const nextConfig = {
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'http://localhost:8080/api/:path*',
      },
    ]
  },
}
```

### Todo API 函数签名（保持不变）

```typescript
// lib/todo.ts — 函数签名与迁移前一致，仅内部实现从 Axios 改为 fetch
createTodo(data: CreateTodoRequest): Promise<Result<TodoItem>>
listTodos(params: TodoQueryRequest): Promise<Result<PageResult<TodoItem>>>
getTodo(id: number): Promise<Result<TodoItem>>
updateTodo(id: number, data: UpdateTodoRequest): Promise<Result<TodoItem>>
deleteTodo(id: number): Promise<Result<void>>
toggleTodo(id: number): Promise<Result<TodoItem>>
batchComplete(ids: number[]): Promise<Result<void>>
batchDelete(ids: number[]): Promise<Result<void>>
```
