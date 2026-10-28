# Frontend Framework Specification

## Purpose

提供前端 Next.js + TypeScript + Tailwind CSS 脚手架及开箱即用的基础设施能力，包括项目骨架、路由配置、主题系统、UI 组件库、HTTP 客户端封装和 API 对接基础配置，使后续所有前端页面开发都基于统一的前端框架约定。

## Requirements

### Requirement: Next.js + TypeScript + Tailwind CSS 项目结构
前端 SHALL 采用 Next.js (App Router) + TypeScript + Tailwind CSS 技术栈，项目位于独立的 `frontend/` 目录，不参与 Maven 构建。前端 SHALL 集成 shadcn/ui 组件库作为统一 UI 基础，并通过 CSS 变量体系支持主题化（含深色模式预留）。

#### Scenario: 开发服务器启动
- **WHEN** 在 frontend/ 目录执行 `npm run dev`
- **THEN** Next.js 开发服务器成功启动，默认监听 `http://localhost:3000`

#### Scenario: 生产构建成功
- **WHEN** 在 frontend/ 目录执行 `npm run build`
- **THEN** Next.js 成功完成生产构建，生成构建产物到 `.next/` 目录，无 TypeScript 编译错误

#### Scenario: shadcn/ui 组件可用
- **WHEN** 开发者在组件中导入 shadcn/ui 基础组件（如 Button、Card、Input）
- **THEN** 组件正常渲染，主题色通过 CSS 变量正确应用

#### Scenario: 环境变量加载
- **WHEN** Next.js 开发服务器或生产构建启动时
- **THEN** 系统读取 `.env.local` 中的环境变量（如 `NEXT_PUBLIC_API_URL`），代码中通过 `process.env.NEXT_PUBLIC_*` 可访问

### Requirement: 文件系统路由配置
前端 SHALL 集成 Next.js App Router 文件系统路由，提供基于目录结构的页面路由，支持页面扩展。前端 SHALL 提供首页及功能页面路由占位，支持后续业务模块扩展。

#### Scenario: 默认首页渲染
- **WHEN** 用户访问根路径 `/`
- **THEN** 展示默认首页组件（`app/page.tsx`），页面正常渲染

#### Scenario: 404 页面
- **WHEN** 用户访问不存在的路由路径
- **THEN** 展示 Next.js 内置 `not-found.tsx` 404 页面

#### Scenario: 待办清单页面
- **WHEN** 用户访问 `/todos`
- **THEN** 展示待办清单页面（`app/todos/page.tsx`），功能与迁移前一致

#### Scenario: 行业资金流页面
- **WHEN** 用户访问 `/industry-flow`
- **THEN** 展示行业资金流占位页面（`app/industry-flow/page.tsx`），页面正常渲染

#### Scenario: 大盘资金流页面
- **WHEN** 用户访问 `/market-flow`
- **THEN** 展示大盘资金流占位页面（`app/market-flow/page.tsx`），页面正常渲染

#### Scenario: AI 助手页面
- **WHEN** 用户访问 `/ai-assistant`
- **THEN** 展示 AI 助手占位页面（`app/ai-assistant/page.tsx`），页面正常渲染

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

### Requirement: Tailwind CSS 主题系统
前端 SHALL 配置完整的 Tailwind CSS 主题系统，包含品牌色、资金流向色、shadcn/ui 语义色和自定义动画，确保所有首页业务模块基于统一视觉规范开发。

#### Scenario: 资金流向色可用
- **WHEN** 组件使用 `text-flow-inflow`、`text-flow-outflow`、`bg-flow-inflow`、`bg-flow-outflow` 等 Tailwind 类名
- **THEN** 渲染颜色符合规范：流入为红色（red-600）、流出为绿色（green-600）、持平为灰色（gray-500）

#### Scenario: 品牌色可用
- **WHEN** 组件使用 `text-brand-600`、`bg-brand-50` 等品牌色 Tailwind 类名
- **THEN** 渲染颜色符合品牌色系定义（primary: blue-600, secondary: blue-600, accent: orange-500）

#### Scenario: 自定义动画可用
- **WHEN** 组件使用 `animate-fade-in`、`animate-slide-in-from-bottom` 等动画 Tailwind 类名
- **THEN** 动画效果正常播放（fade-in 0.3s、slide-in 0.3s）

#### Scenario: 深色模式 CSS 变量
- **WHEN** HTML 根元素添加 `dark` class
- **THEN** 所有通过 `hsl(var(--background))` 等 CSS 变量引用的颜色切换为深色主题值

### Requirement: 项目目录结构规范
前端 SHALL 维护标准化的目录结构，确保组件、工具函数、类型定义、自定义 Hooks 和静态资源按职责分离，支持多人协作和后续模块扩展。

#### Scenario: UI 组件目录存在
- **WHEN** 检查 `frontend/components/ui/` 目录
- **THEN** 目录存在且包含 shadcn/ui 基础组件文件（如 button.tsx、card.tsx）

#### Scenario: 首页业务组件目录存在
- **WHEN** 检查 `frontend/components/homepage/` 目录
- **THEN** 目录存在，用于存放首页业务组件

#### Scenario: 自定义 Hooks 目录存在
- **WHEN** 检查 `frontend/hooks/` 目录
- **THEN** 目录存在，用于存放自定义 React Hooks

#### Scenario: 静态资源目录存在
- **WHEN** 检查 `frontend/public/images/` 和 `frontend/public/icons/` 目录
- **THEN** 目录存在，用于存放图片和图标静态资源

### Requirement: 全局样式体系
前端 SHALL 在 `app/globals.css` 中建立完整的 CSS 变量体系，包含 shadcn/ui 主题变量、资金流向色变量、自定义滚动条样式和文本截断工具类。

#### Scenario: CSS 变量主题加载
- **WHEN** 页面加载时
- **THEN** `globals.css` 中定义的 `:root` CSS 变量（`--background`、`--foreground`、`--primary` 等）生效，shadcn/ui 组件通过 `hsl(var(--*))` 正确渲染

#### Scenario: 自定义滚动条样式
- **WHEN** 页面出现滚动条时
- **THEN** 滚动条使用 `globals.css` 中定义的自定义样式（轨道 bg-gray-100、滑块 bg-gray-300、圆角）

#### Scenario: 文本截断工具类可用
- **WHEN** 组件使用 `.text-truncate`、`.line-clamp-2`、`.line-clamp-3` 等 CSS 类名
- **THEN** 文本按预期截断并显示省略号
