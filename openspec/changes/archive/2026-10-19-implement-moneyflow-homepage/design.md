# Design

## Context

MoneyFlow 首页模块需要实现 8 个业务单元（Hero、导航、时间选择器、热力图、树状图、详情面板、更新时间、响应式布局），为用户提供资金流向可视化分析的核心入口。

**当前状态**：
- 前端共享基础设施已就绪：`cn()` 工具、格式化工具函数、业务类型定义、共享常量、通用 UI 组件（LoadingSkeleton / ErrorMessage / EmptyState）
- 前端页面路由已占位：`/industry-flow`、`/market-flow`、`/ai-assistant` 均为占位页
- 后端尚未设计行业数据 / 资金流数据相关的实体与 API

**约束条件**：
- 团队 3 人，全员熟悉 React，有充足 Spring Boot 经验
- 8 周 MVP 周期，必须上线
- 数据来源未定（需 Mock 数据支撑前端开发）
- 图表库需支持 Treemap + Heatmap，且性能良好（100+ 节点）

## Goals / Non-Goals

**Goals:**
- 实现首页 8 个业务单元的前端展示与交互
- 建立清晰的前端组件架构，便于后续维护和扩展
- 通过 Mock 数据层解耦前后端开发，支持独立迭代
- 确保三端（PC / 平板 / 移动）响应式体验一致

**Non-Goals:**
- 不实现后端数据 API 和数据库表设计（另行变更）
- 不实现真实数据源接入（如 新浪财经 / 东方财富 API）
- 不实现用户认证和权限控制（MVP 阶段跳过）
- 不实现触摸手势的完整能力（MVP 仅支持基础缩放 / 拖动，高级手势延后）

## Decisions

### Decision 1: 图表库选型 — ECharts

**选择**: Apache ECharts (echarts-for-react)

**理由**:
- 内置 Treemap 和 Heatmap 图表类型，配置式开发，无需手写渲染逻辑
- 性能优秀：Canvas 渲染，100+ 节点流畅
- 社区活跃，中文文档完善，符合团队技术背景
- 支持丰富的交互事件（click / mouseover / datazoom），满足下钻和联动需求

**备选方案**:
- D3.js: 灵活度极高，但学习曲线陡峭，开发周期长，不适合 8 周 MVP
- Recharts: 基于 SVG，性能不如 Canvas，Treemap 支持较弱

### Decision 2: 状态管理 — React Context + 自定义 Hook

**选择**: 使用 React Context 管理全局 TimeRange 状态，配合自定义 Hook（`useTimeRange`）提供订阅和更新能力

**理由**:
- TimeRange 是跨组件共享的核心状态（时间选择器 → 热力图 / 树状图 / 详情面板）
- Context 轻量，无需引入额外依赖（如 Redux / Zustand）
- 符合 Next.js App Router 的 'use client' 组件模型

**备选方案**:
- Redux Toolkit: 功能强大但对 MVP 过重
- SWR / React Query: 适合服务端状态缓存，但 TimeRange 是客户端状态，不适合

### Decision 3: Mock 数据策略 — 前端 Mock Service 层

**选择**: 在 `frontend/lib/mock/` 下创建 Mock 数据服务，提供与真实 API 相同签名的函数，返回本地生成的 Mock 数据

**理由**:
- 解耦前后端开发，前端可独立迭代
- Mock 数据服务与真实 API 服务接口一致，后续仅需替换导入路径
- 便于单元测试和 Storybook 展示

**备选方案**:
- MSW (Mock Service Worker): 功能强大但配置复杂，MVP 阶段不必要
- 后端 Mock Controller: 增加后端工作量，延缓前端开发

### Decision 4: 组件架构 — Layout 组合模式

**选择**: 采用"页面 → Layout → Section → Component"四层架构

```
app/page.tsx (页面入口)
  └─ HomepageLayout (布局容器，管理 Header / Footer / 主内容区)
       ├─ HeroSection (品牌主视觉)
       ├─ NavigationCards (功能导航)
       ├─ UpdateTimestamp (更新时间)
       ├─ TimeSelector (时间选择器)
       ├─ IndustryHeatmap (热力图)
       ├─ IndustryTreemap (树状图)
       └─ SectorDetailPanel (详情面板，浮层)
```

**理由**:
- 职责清晰，每个组件独立开发和测试
- Layout 组件统一管理响应式断点和栅格布局
- 浮层组件（SectorDetailPanel）独立于主流程，通过事件触发

**备选方案**:
- 单文件大组件: 开发快但维护困难，不符合长期迭代需求

### Decision 5: 响应式实现 — Tailwind 断点 + CSS Grid/Flex

**选择**: 使用 Tailwind CSS 的 `sm:` / `md:` / `lg:` 断点前缀，配合 CSS Grid 和 Flexbox 实现自适应布局

**理由**:
- Tailwind 断点与 `BREAKPOINTS` 常量一致（mobile: 768, tablet: 1024, desktop: 1024）
- 无需手写媒体查询，开发效率高
- Grid 适合二维布局（热力图 / 树状图并排），Flex 适合一维布局（导航卡片）

**备选方案**:
- CSS-in-JS (styled-components): 增加运行时开销，Next.js 推荐 Tailwind
- Bootstrap: 与现有 Tailwind 体系冲突

## Architecture

### 前端组件关系图

```
┌─────────────────────────────────────────────────────────┐
│                    app/page.tsx                         │
│                  (首页入口，组装 Layout)                  │
└────────────────────┬────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌──────────────────┐    ┌──────────────────┐
│ HomepageLayout   │    │ TimeRangeContext │
│ (布局容器)        │    │ (全局状态)        │
└──────────────────┘    └──────────────────┘
        │                         │
        │  ┌──────────────────────┘
        │  │
        ▼  ▼
┌──────────────────────────────────────────────────────┐
│                   Section 组件层                      │
│  ┌────────────┐ ┌────────────┐ ┌──────────────────┐ │
│  │HeroSection │ │Navigation  │ │UpdateTimestamp   │ │
│  │            │ │Cards       │ │                  │ │
│  └────────────┘ └────────────┘ └──────────────────┘ │
│  ┌────────────┐ ┌────────────┐ ┌──────────────────┐ │
│  │TimeSelector│ │Industry    │ │IndustryTreemap   │ │
│  │            │ │Heatmap     │ │                  │ │
│  └────────────┘ └────────────┘ └──────────────────┘ │
└──────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────────────┐
│                   浮层组件                            │
│  ┌────────────────────────────────────────────────┐  │
│  │ SectorDetailPanel (详情面板，点击色块触发)       │  │
│  └────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────┘
        │
        ▼
┌──────────────────────────────────────────────────────┐
│                   数据服务层                          │
│  ┌────────────────┐  ┌────────────────────────────┐  │
│  │ lib/mock/      │  │ lib/api/ (未来真实 API)     │  │
│  │ (Mock 数据)     │  │                            │  │
│  └────────────────┘  └────────────────────────────┘  │
└──────────────────────────────────────────────────────┘
```

### 数据流向

```
用户操作 TimeSelector
    │
    ▼
TimeRangeContext 更新 state
    │
    ▼
Heatmap / Treemap 组件监听 context 变化
    │
    ▼
调用 Mock Service (或未来 API) 获取新数据
    │
    ▼
ECharts 重新渲染图表
    │
    ▼
用户点击色块 → 触发 onIndustryClick
    │
    ▼
SectorDetailPanel 打开，展示详情
```

## Key Interfaces

### 前端组件 Props 接口

```typescript
// TimeSelector
interface TimeSelectorProps {
  value?: TimeRange
  onChange?: (range: TimeRange) => void
}

// IndustryHeatmap
interface IndustryHeatmapProps {
  timeRange: TimeRange
  onIndustryClick?: (industryCode: string) => void
}

// IndustryTreemap
interface IndustryTreemapProps {
  timeRange: TimeRange
  initialLevel?: 1 | 2 | 3 | 4
  onStockClick?: (stockCode: string) => void
}

// SectorDetailPanel
interface SectorDetailPanelProps {
  industryCode: string | null
  timeRange: TimeRange
  onClose: () => void
}
```

### Mock Service 接口

```typescript
// frontend/lib/mock/homepage.ts

export interface MockHomepageService {
  getHeatmapData(timeRange: TimeRange): Promise<IndustryData[]>
  getTreemapData(timeRange: TimeRange, parentCode?: string): Promise<IndustryData[]>
  getIndustryDetail(industryCode: string, timeRange: TimeRange): Promise<IndustryDetail>
  getUpdateTime(): Promise<Date>
}
```

### 后端 API 接口（未来实现）

```
GET /api/industry/heatmap?timeRange=today
GET /api/industry/treemap?timeRange=today&parentCode=xxx
GET /api/industry/{code}/detail?timeRange=today
GET /api/data/update-time
```

响应格式统一为 `Result<T>`（code / message / data）。

## Risks / Trade-offs

**[风险 1] ECharts 包体积较大** → 通过 `next/dynamic` 动态导入，仅在首页加载

**[风险 2] Mock 数据与真实 API 不一致** → Mock Service 严格遵循 TypeScript 类型定义（`IndustryData` / `StockData`），后续替换时仅需修改导入路径

**[风险 3] 触摸手势实现复杂** → MVP 阶段仅支持基础缩放（双指）和拖动（单指），高级手势（滑动关闭面板）延后到 v1.1

**[风险 4] 大数据量渲染性能** → ECharts Canvas 渲染 + 分层加载（树状图逐级下钻），避免一次性渲染 100+ 节点

**[权衡] 使用 React Context 而非 Redux** → 轻量但状态调试工具较弱，MVP 阶段可接受，后续如需复杂状态管理再引入

## Migration Plan

**部署策略**: 前端独立部署，通过 Next.js rewrites 代理 `/api` 到后端

**回滚策略**:
- 前端回滚：Git 回退到变更前 commit，删除 `components/homepage/` 目录
- 后端回滚：无影响（本变更不涉及后端）

**上线检查清单**:
1. 首页三端响应式正常
2. 时间选择器切换正常，图表联动刷新
3. 热力图 / 树状图下钻交互流畅
4. 详情面板打开 / 关闭正常
5. 性能达标（首屏 < 3s，图表渲染 < 500ms）
