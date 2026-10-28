# Design

## Context

当前项目前端为 Next.js App Router + TypeScript + Tailwind CSS，后端为 Spring Boot 3.2.5 + Maven 多模块 + Spring Data JPA。前端已有首页（`/`）、行业资金流（`/industry-flow`）、大盘资金流（`/market-flow`）、AI 助手（`/ai-assistant`）四个页面，以及 `components/homepage/` 下的可复用组件。后端已有 Todo 模块作为参考实现。

本次变更将首页从功能仪表盘重构为品牌落地页，新建 `/research` 和 `/policy-reports` 两个路由，删除三个旧路由。

## Goals / Non-Goals

**Goals:**
- 首页重构为极简品牌落地页（左右分栏 Hero 区）
- 新建 `/research` 资金流研究平台（左侧导航 + 右侧内容区）
- 新建 `/policy-reports` 政策报告页面（后端爬取 + 前端展示）
- 删除旧路由 `/industry-flow`、`/market-flow`、`/ai-assistant`
- 原有数据可视化组件（热力图、树状图等）迁移至研究平台

**Non-Goals:**
- 不实现实时数据推送（WebSocket/SSE），实时资金流向通过定时轮询实现
- 不实现政策报告 PDF 在线预览，仅提供外链/下载
- 不实现策略库功能，仅占位
- 不实现用户登录/权限系统

## Decisions

### 1. 前端路由结构

```
frontend/app/
├── layout.tsx              # 根布局（全局 Header + Footer）
├── page.tsx                # 首页（品牌落地页）
├── research/
│   ├── layout.tsx          # 研究平台布局（左侧导航）
│   └── page.tsx            # 研究平台默认页（宏观数据）
└── policy-reports/
    └── page.tsx            # 政策报告页面
```

**决策理由**：
- 使用 Next.js App Router 的嵌套布局（Nested Layouts），`research/layout.tsx` 提供左侧导航框架，子页面只需关注右侧内容
- 根 `layout.tsx` 保留全局 Header（含导航栏）和 Footer，所有页面共享
- 旧路由目录直接删除，不保留重定向（MVP 阶段无需兼容）

### 2. 研究平台左侧导航实现

**方案**：使用 React state 管理当前选中的导航项，右侧内容区根据选中项条件渲染对应子模块组件。

```
ResearchLayout
├── Sidebar (固定宽度 240px)
│   ├── Logo
│   └── NavItems[] (宏观数据、申万行业分类、资金流向(实时)、资金流向、策略库)
└── Content (flex-1, overflow-auto)
    └── 根据 activeTab 渲染对应子模块
```

**替代方案**：每个子模块使用独立路由（`/research/macro`、`/research/industry` 等）。
**选择理由**：单页面 + state 切换更轻量，避免路由跳转的加载延迟，左侧导航状态保持更简单。MVP 阶段五个子模块足够用 state 管理，后续若子模块复杂度增加可再拆分为独立路由。

### 3. 组件迁移策略

原有 `components/homepage/` 下的组件按以下方式处理：

| 组件 | 处理方式 |
|------|---------|
| `HeroSection.tsx` | 重写为品牌落地页 Hero（移除搜索框，改为左右分栏） |
| `HomepageLayout.tsx` | 重写 Header 导航链接；Footer 保留 |
| `NavigationCards.tsx` | 废弃，不再使用 |
| `IndustryHeatmap.tsx` | 迁移至 `components/research/`，行为不变 |
| `IndustryTreemap.tsx` | 迁移至 `components/research/`，行为不变 |
| `TimeSelector.tsx` | 迁移至 `components/research/`，行为不变 |
| `SectorDetailPanel.tsx` | 迁移至 `components/research/`，行为不变 |
| `UpdateTimestamp.tsx` | 迁移至 `components/research/`，行为不变 |
| `Breadcrumb.tsx` | 迁移至 `components/research/`，用于树状图面包屑 |
| `TimeRangeContext.tsx` | 迁移至 `context/` 保留，研究平台复用 |

### 4. 政策报告后端设计

**数据模型**：
```java
@Entity
public class PolicyReport {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;          // 报告标题
    private String summary;        // 报告摘要
    private String category;       // 分类（产业政策/工作会议/宏观经济/...）
    private String source;         // 来源机构
    private LocalDate publishDate; // 发布日期
    private String sourceUrl;      // 原文链接
    private String documentUrl;    // PDF/文档URL
    // 公共字段：createdAt, updatedAt
}
```

**模块归属**：
- `moneyflow-model`: `PolicyReport.java`（Entity）、`PolicyReportVO.java`、`PolicyReportQueryRequest.java`（DTO）
- `moneyflow-dao`: `PolicyReportRepository.java`
- `moneyflow-service`: `PolicyReportService.java` + `PolicyReportServiceImpl.java`、`PolicyReportCrawlerService.java`（爬取逻辑）
- `moneyflow-web`: `PolicyReportController.java`

**爬取方案**：使用 Spring `@Scheduled` 定时任务 + `RestTemplate`/`Jsoup` 爬取。MVP 阶段先实现手动触发接口，定时任务后续接入。

**替代方案**：使用 Python 爬虫微服务。
**选择理由**：MVP 阶段报告源有限，Java 内实现减少运维复杂度。后续若爬取逻辑复杂可拆分为独立服务。

### 5. 关键接口定义

**政策报告 API**：
```
GET  /api/policy-reports              # 分页查询报告列表（支持 category 筛选）
GET  /api/policy-reports/{id}         # 查询报告详情
POST /api/policy-reports/crawl        # 手动触发爬取（管理接口）
```

**研究平台数据 API**（复用现有接口，路径不变）：
```
GET /api/industries/heatmap           # 申万二级行业热力图数据
GET /api/industries/treemap           # 申万行业树状图数据
GET /api/market/flow                  # 大盘资金流向数据
```

## Risks / Trade-offs

- **[爬取稳定性]** → 政策报告爬取依赖外部网站结构，网站改版可能导致爬取失败。Mitigation: 爬取逻辑封装为独立 Service，失败时记录日志并告警，支持手动重试。
- **[组件迁移兼容性]** → 原有组件从 homepage 目录迁移到 research 目录，import 路径全部变更。Mitigation: 使用 IDE 全局重构功能批量更新 import，迁移后运行 TypeScript 编译检查。
- **[左侧导航 state 管理]** → 五个子模块用 state 切换，若后续子模块增多可能导致单页面过重。Mitigation: 每个子模块独立组件文件，按需懒加载（`React.lazy`）。

## Migration Plan

1. 新建 `/research` 和 `/policy-reports` 路由及布局组件
2. 迁移首页组件至 research 组件目录，更新 import 路径
3. 重写首页 `page.tsx` 为品牌落地页
4. 重写 `HomepageLayout.tsx` 导航栏
5. 删除旧路由目录（`industry-flow`、`market-flow`、`ai-assistant`）
6. 后端新增 PolicyReport 模块（Entity → Repository → Service → Controller）
7. 前端新增政策报告页面
8. 全量 TypeScript 编译检查 + 页面功能验证

**回滚**：Git revert 本次变更的所有 commit 即可恢复。
