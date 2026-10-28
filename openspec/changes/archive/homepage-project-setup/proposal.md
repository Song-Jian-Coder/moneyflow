# Proposal

## Why

MoneyFlow 首页模块开发需要完整的项目骨架基础设施。当前前端仅具备基础 Next.js 骨架和 Todo 模块，缺少首页开发所需的自定义主题系统（资金流向色）、shadcn/ui 组件库、功能页面路由、环境变量配置和标准化目录结构。本次变更搭建首页项目骨架，为后续 8 个首页业务模块提供开发基础。

## What Changes

- **增强** Tailwind CSS 主题配置：添加品牌色、资金流向色（红=流入/绿=流出）、shadcn/ui CSS 变量主题、自定义动画、响应式断点
- **增强** 全局样式 `globals.css`：添加 CSS 变量体系、深色模式支持、资金流向色工具类、自定义滚动条、文本截断工具类
- **新增** shadcn/ui 组件库初始化：安装配置 + 基础组件（Button、Card、Dialog、Input、Table、Badge 等）
- **新增** 功能页面路由占位：`/industry-flow`（行业资金流）、`/market-flow`（大盘资金流）、`/ai-assistant`（AI 助手）
- **新增** 目录结构扩展：`components/ui/`（shadcn/ui）、`components/homepage/`（首页业务组件）、`hooks/`、`public/images/`、`public/icons/`
- **新增** 环境变量配置：`.env.local`（本地开发）、`.env.example`（模板）
- **增强** `next.config.ts`：添加图片远程域名配置、ECharts 外部包声明

## Capabilities

### New Capabilities

（无新增能力）

### Modified Capabilities

- `frontend-framework`：扩展项目基础设施需求，新增以下行为变更：
  - "Next.js + TypeScript + Tailwind CSS 项目结构" → 增加 shadcn/ui 组件库、自定义主题系统、环境变量配置
  - "文件系统路由配置" → 新增 `/industry-flow`、`/market-flow`、`/ai-assistant` 三个功能页面路由占位
  - 新增 Tailwind CSS 主题扩展需求：品牌色、资金流向色、shadcn/ui CSS 变量主题、自定义动画关键帧
  - 新增目录结构需求：`components/ui/`、`components/homepage/`、`hooks/`、`public/images/`、`public/icons/`

## Impact

- **前端子仓库**（`frontend/`）：
  - 配置文件变更：`tailwind.config.ts`、`next.config.ts`、`app/globals.css`
  - 新增依赖：`shadcn/ui` 相关包、`clsx`、`tailwind-merge`、`lucide-react`、`date-fns`、`tailwindcss-animate`、`@tailwindcss/typography`
  - 新增文件：shadcn/ui 组件（`components/ui/`）、路由占位页面、环境变量文件
  - 新增目录：`components/ui/`、`components/homepage/`、`hooks/`、`public/images/`、`public/icons/`
- **后端 API**：零影响
- **Todo 模块**：零影响（现有功能保持不变）
- **现有页面**：`app/page.tsx`、`app/layout.tsx`、`app/todos/page.tsx` 保持现有功能不变，layout 和 globals 增强为向后兼容

## 回滚方案

- 所有变更集中在前端 `frontend/` 目录，通过 Git 可完整回滚
- 新增依赖通过 `package.json` 管理，`npm install` 可恢复
- 新增路由为占位页面，删除不影响现有 `/` 和 `/todos` 路由
- shadcn/ui 组件为独立目录 `components/ui/`，删除不影响现有 `components/todo/`
