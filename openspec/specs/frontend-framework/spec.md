# Frontend Framework Specification

## Purpose

提供前端 React + Vite + TypeScript 脚手架及开箱即用的基础设施能力，包括项目骨架、路由配置、HTTP 客户端封装和 API 对接基础配置，使后续所有前端页面开发都基于统一的前端框架约定。

## Requirements

### Requirement: React + Vite + TypeScript 项目结构
前端 SHALL 采用 React + Vite + TypeScript 技术栈，项目位于独立的 `frontend/` 目录，不参与 Maven 构建。

#### Scenario: 开发服务器启动
- **WHEN** 在 frontend/ 目录执行 `npm run dev`
- **THEN** Vite 开发服务器成功启动，默认监听 `http://localhost:5173`

#### Scenario: 生产构建成功
- **WHEN** 在 frontend/ 目录执行 `npm run build`
- **THEN** 生成生产构建产物到 `dist/` 目录，无 TypeScript 编译错误

### Requirement: 路由配置
前端 SHALL 集成 React Router，提供基础路由配置结构，支持页面扩展。

#### Scenario: 默认首页渲染
- **WHEN** 用户访问根路径 `/`
- **THEN** 展示默认首页组件，页面正常渲染

#### Scenario: 404 页面
- **WHEN** 用户访问不存在的路由路径
- **THEN** 展示 404 页面或重定向到首页

### Requirement: HTTP 客户端封装
前端 SHALL 封装 Axios HTTP 客户端，提供统一的请求和响应拦截器。

#### Scenario: 请求拦截器
- **WHEN** 发送任意 HTTP 请求
- **THEN** 请求经过统一拦截器处理（如添加通用请求头）

#### Scenario: 响应拦截器
- **WHEN** 收到后端 API 响应
- **THEN** 响应经过统一拦截器处理（如统一解析 Result 格式、提取 data 字段）

#### Scenario: 网络错误处理
- **WHEN** 请求因网络原因失败（如超时、断网）
- **THEN** 统一捕获错误并提供友好的错误提示

### Requirement: API 代理配置
前端开发环境 SHALL 配置代理，将 API 请求转发到后端开发服务器。

#### Scenario: API 请求代理
- **WHEN** 前端开发服务器收到以 `/api` 开头的请求
- **THEN** 请求被代理转发到后端服务地址（如 `http://localhost:8080`）
