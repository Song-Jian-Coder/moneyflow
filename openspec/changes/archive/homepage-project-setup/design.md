# Design

## Context

当前前端已完成 Vite → Next.js 迁移，具备基础骨架：Next.js 16.4.2 + React 19 + TypeScript 5.7 + Tailwind CSS 4.3.3。现有配置为最小化默认配置——Tailwind 无自定义主题、globals.css 仅含 `@import "tailwindcss"`、无 shadcn/ui、无环境变量文件、路由仅 `/` 和 `/todos`。

本次变更需在现有骨架上扩展首页开发所需的基础设施。关键约束：项目使用 Tailwind CSS v4（CSS-first 配置），而非 spec 文档中假设的 v3（JS config），需适配 v4 的主题配置方式。

## Goals / Non-Goals

**Goals:**
- 建立完整的 Tailwind CSS 主题系统（品牌色、资金流向色、shadcn/ui 语义色）
- 初始化 shadcn/ui 组件库并安装基础组件
- 创建功能页面路由占位（/industry-flow、/market-flow、/ai-assistant）
- 建立 CSS 变量主题体系（含深色模式预留）
- 扩展标准化目录结构
- 配置环境变量

**Non-Goals:**
- 不实现首页业务组件（由后续 homepage-* 变更负责）
- 不实现共享工具函数和类型定义（由 homepage-shared-foundation 变更负责）
- 不引入状态管理库（React Query / Zustand 后续按需添加）
- 不修改后端 API 或 Todo 模块

## Decisions

### 决策 1：适配 Tailwind CSS v4 主题配置方式

**理由**：项目实际使用 Tailwind v4.3.3，其配置模型与 v3 根本不同。v4 采用 CSS-first 配置，通过 `@theme` 指令在 CSS 文件中定义设计令牌（design tokens），而非 `tailwind.config.ts` 的 JS 对象。

**方案**：
- `app/globals.css`：使用 `@theme` 块定义颜色、动画、字体等设计令牌
- `tailwind.config.ts`：保留用于插件注册和 content 路径配置
- CSS 变量体系：在 `:root` 和 `.dark` 选择器中定义 shadcn/ui 主题变量

**替代方案**：降级到 Tailwind v3 — 增加技术债，v4 是官方推荐方向，且项目已安装 v4。

### 决策 2：shadcn/ui 初始化策略

**理由**：shadcn/ui 是首页 spec 中所有业务组件的 UI 基础。需要初始化配置并安装基础组件集。

**方案**：
- 使用 `npx shadcn@latest init` 初始化（自动检测 Tailwind v4 + App Router）
- 安装基础组件：button、card、dialog、input、table、badge、popover、select、tooltip
- 组件输出到 `components/ui/` 目录
- 主题色通过 CSS 变量注入，与 shadcn/ui 默认主题系统对齐

**替代方案**：手动搭建组件 — 工作量大且与 shadcn/ui 生态不兼容。

### 决策 3：路由占位页面设计

**理由**：需要为三个功能页面创建路由占位，但不应引入空壳页面导致用户体验差。

**方案**：
- 每个占位页面展示功能名称和"开发中"提示
- 提供返回首页链接
- 使用 Tailwind 基础样式，不依赖 shadcn/ui 组件（保持占位页面轻量）

**替代方案**：空白页面 — 不利于开发阶段验证路由是否生效。

### 决策 4：环境变量管理

**理由**：首页模块需要配置 API 地址等环境变量，且需区分开发/生产环境。

**方案**：
- `.env.local`：本地开发环境变量（Git 忽略）
- `.env.example`：环境变量模板（提交到 Git）
- 定义变量：`NEXT_PUBLIC_API_URL`（API 基础地址）、`NEXT_PUBLIC_APP_TITLE`（应用标题）
- 代码中通过 `process.env.NEXT_PUBLIC_*` 访问

### 决策 5：目录结构扩展策略

**理由**：首页模块需要独立的组件目录，同时保持与现有 Todo 组件目录的共存。

**方案**：
```
frontend/
├── app/
│   ├── industry-flow/page.tsx    # 新增：行业资金流占位
│   ├── market-flow/page.tsx      # 新增：大盘资金流占位
│   ├── ai-assistant/page.tsx     # 新增：AI 助手占位
│   ├── page.tsx                  # 现有：首页（保持）
│   ├── layout.tsx                # 增强：metadata + 全局样式
│   ├── globals.css               # 增强：CSS 变量主题
│   ├── not-found.tsx             # 现有：404（保持）
│   └── todos/page.tsx            # 现有：Todo（保持）
├── components/
│   ├── ui/                       # 新增：shadcn/ui 组件
│   ├── homepage/                 # 新增：首页业务组件（空目录占位）
│   └── todo/                     # 现有：Todo 组件（保持）
├── hooks/                        # 新增：自定义 Hooks（空目录占位）
├── lib/                          # 现有：工具函数（保持）
├── types/                        # 现有：类型定义（保持）
├── public/
│   ├── images/                   # 新增：图片资源
│   └── icons/                    # 新增：图标资源
├── .env.local                    # 新增：本地环境变量
├── .env.example                  # 新增：环境变量模板
└── components.json               # 新增：shadcn/ui 配置
```

## Risks / Trade-offs

- **[风险] Tailwind v4 与 shadcn/ui 兼容性** → shadcn/ui 官方已支持 Tailwind v4（2025 年后），初始化时自动检测。如遇兼容问题，参考 shadcn/ui 官方 v4 迁移指南。
- **[风险] 新增依赖版本冲突** → 安装前使用 `npm ls` 检查现有依赖树，确保新包版本与 React 19 / Next.js 16 兼容。
- **[取舍] 占位页面不引入 shadcn/ui 组件** → 保持占位页面轻量，但后续首页开发时需补充 UI 组件引用。
- **[取舍] CSS 变量方案增加 globals.css 复杂度** → globals.css 从 2 行扩展到约 100 行，但换来完整的主题化能力和深色模式预留。
- **[取舍] 新增目录暂为空** → `components/homepage/`、`hooks/` 等目录当前无内容，但为后续模块提供明确的文件归属。

## 关键接口定义

### 环境变量接口

```env
# .env.local
NEXT_PUBLIC_API_URL=http://localhost:8080
NEXT_PUBLIC_APP_TITLE=MoneyFlow
```

### shadcn/ui 配置

```json
// components.json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "default",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "app/globals.css",
    "baseColor": "neutral",
    "cssVariables": true
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  }
}
```

### Tailwind v4 主题令牌（globals.css @theme 块）

```css
@theme {
  /* 品牌色 */
  --color-brand-50: #eff6ff;
  --color-brand-100: #dbeafe;
  --color-brand-500: #3b82f6;
  --color-brand-600: #2563eb;
  --color-brand-700: #1d4ed8;
  --color-brand-900: #1e3a8a;

  /* 资金流向色 */
  --color-flow-inflow: #dc2626;
  --color-flow-outflow: #16a34a;
  --color-flow-neutral: #6b7280;

  /* 动画 */
  --animate-fade-in: fade-in 0.3s ease-out;
  --animate-slide-in-from-bottom: slide-in-from-bottom 0.3s ease-out;
}
```
