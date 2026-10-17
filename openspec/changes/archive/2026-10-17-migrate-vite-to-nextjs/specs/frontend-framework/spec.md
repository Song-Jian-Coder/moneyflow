# Spec Delta

## MODIFIED Requirements

### Requirement: Next.js + TypeScript + Tailwind CSS 项目结构
前端 SHALL 采用 Next.js (App Router) + TypeScript + Tailwind CSS 技术栈，项目位于独立的 `frontend/` 目录，不参与 Maven 构建。

#### Scenario: 开发服务器启动
- **WHEN** 在 frontend/ 目录执行 `npm run dev`
- **THEN** Next.js 开发服务器成功启动，默认监听 `http://localhost:3000`

#### Scenario: 生产构建成功
- **WHEN** 在 frontend/ 目录执行 `npm run build`
- **THEN** Next.js 成功完成生产构建，生成构建产物到 `.next/` 目录，无 TypeScript 编译错误

### Requirement: 文件系统路由配置
前端 SHALL 集成 Next.js App Router 文件系统路由，提供基于目录结构的页面路由，支持页面扩展。

#### Scenario: 默认首页渲染
- **WHEN** 用户访问根路径 `/`
- **THEN** 展示默认首页组件（`app/page.tsx`），页面正常渲染

#### Scenario: 404 页面
- **WHEN** 用户访问不存在的路由路径
- **THEN** 展示 Next.js 内置 `not-found.tsx` 404 页面

#### Scenario: 待办清单页面
- **WHEN** 用户访问 `/todos`
- **THEN** 展示待办清单页面（`app/todos/page.tsx`），功能与迁移前一致

### Requirement: HTTP 客户端封装
前端 SHALL 封装基于 `fetch` API 的 HTTP 客户端，提供统一的请求和响应处理，兼容 Next.js Server Components 和 Client Components。

#### Scenario: 请求封装
- **WHEN** 发送任意 HTTP 请求
- **THEN** 请求经过统一封装处理（如设置 Content-Type、超时控制）

#### Scenario: 响应拦截处理
- **WHEN** 收到后端 API 响应
- **THEN** 响应经过统一处理（如统一解析 Result 格式、提取 data 字段、错误码判断）

#### Scenario: 网络错误处理
- **WHEN** 请求因网络原因失败（如超时、断网）
- **THEN** 统一捕获错误并提供友好的错误提示

### Requirement: API 代理配置
前端开发环境 SHALL 通过 Next.js rewrites 配置代理，将 API 请求转发到后端开发服务器。

#### Scenario: API 请求代理
- **WHEN** 前端开发服务器收到以 `/api` 开头的请求
- **THEN** 请求通过 Next.js rewrites 被代理转发到后端服务地址（`http://localhost:8080`）
