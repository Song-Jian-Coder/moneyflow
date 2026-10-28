# Proposal

## Why

当前首页是一个功能仪表盘，直接堆叠了热力图、树状图、时间选择器等数据组件，缺乏品牌传达能力。参考聚宽量化研究平台的设计，将首页重构为极简品牌落地页，同时将数据可视化功能统一迁移到「资金流研究平台」子页面（左侧导航布局），提升产品的专业感和信息架构清晰度。此外，新增「政策报告」模块，通过后端自动爬取经济类报告并分类入库，为用户提供政策研究支持。

## What Changes

- **首页重构为品牌落地页**：移除所有数据可视化组件（热力图、树状图、时间选择器、导航卡片、详情面板、更新时间），替换为左右分栏 Hero 区（左侧大标语 + 右侧静态渐变背景），标语为"更专业、更专注 / 资金流汇聚网站"
- **新增资金流研究平台**（`/research`）：左侧固定导航 + 右侧内容区布局，包含五个子模块：宏观数据、申万行业分类（树状图）、资金流向（实时）（沪深两市实时资金流向图 + 申万一级行业实时资金流向图）、资金流向（沪深两市历史资金流向图 + 申万二级行业热力图）、策略库（占位）
- **新增政策报告页面**（`/policy-reports`）：后端自动爬取产业政策报告、工作会议报告等经济类报告，分类入库；前端展示报告列表、分类筛选、详情
- **导航栏重构**：顶部导航从「首页 | 行业资金流 | 大盘资金流 | AI 助手」改为「首页 | 政策报告 | 资金流研究平台」
- **BREAKING**：删除旧路由 `/industry-flow`、`/market-flow`、`/ai-assistant` 及其对应页面代码
- **BREAKING**：首页不再展示任何数据可视化内容，原首页组件（IndustryHeatmap、IndustryTreemap、TimeSelector、SectorDetailPanel、UpdateTimestamp、NavigationCards）迁移至 `/research` 子页面或废弃

## Capabilities

### New Capabilities

- `research-platform`: 资金流研究平台，提供左侧导航 + 右侧内容区的布局框架，包含宏观数据、申万行业分类、资金流向（实时）、资金流向、策略库五个子模块，各子模块通过左侧导航切换
- `policy-reports`: 政策报告聚合，后端自动爬取经济类报告（产业政策、工作会议报告等），分类入库，前端提供报告列表展示、分类筛选和详情查看

### Modified Capabilities

- `homepage-hero`: Hero 区从「渐变背景 + 搜索框 + 副标语」改为「静态渐变背景 + 大标语（更专业、更专注 / 资金流汇聚网站）+ 副标语」，移除搜索框
- `homepage-navigation`: 导航栏从「首页 | 行业资金流 | 大盘资金流 | AI 助手」改为「首页 | 政策报告 | 资金流研究平台」，移除导航卡片组件
- `homepage-responsive-layout`: 响应式布局适配新的首页极简设计和研究平台左侧导航布局
- `homepage-heatmap`: 热力图从首页迁移至 `/research` 的「资金流向」子模块，行为不变但上下文变化
- `homepage-industry-treemap`: 树状图从首页迁移至 `/research` 的「申万行业分类」子模块，行为不变但上下文变化
- `homepage-time-selector`: 时间选择器从首页迁移至 `/research`，行为不变但上下文变化
- `homepage-update-timestamp`: 更新时间组件从首页迁移至 `/research`，行为不变但上下文变化
- `homepage-sector-detail-panel`: 详情面板从首页迁移至 `/research`，行为不变但上下文变化

## Impact

**前端影响**：
- `frontend/app/page.tsx` — 完全重写为品牌落地页
- `frontend/app/` — 新增 `research/` 路由目录（左侧导航布局 + 五个子模块页面）
- `frontend/app/` — 新增 `policy-reports/` 路由目录
- `frontend/app/industry-flow/`、`market-flow/`、`ai-assistant/` — 删除
- `frontend/components/homepage/` — HeroSection、HomepageLayout、NavigationCards 重写；IndustryHeatmap、IndustryTreemap、TimeSelector、SectorDetailPanel、UpdateTimestamp 迁移至 research 组件目录
- `frontend/components/` — 新增 `research/ResearchLayout.tsx`（左侧导航布局组件）
- `frontend/context/TimeRangeContext.tsx` — 迁移至 research 上下文

**后端影响**：
- 新增政策报告爬取、分类、存储相关 API（Entity、Repository、Service、Controller）
- 新增政策报告数据表

**规范影响**：
- 8 个现有 homepage-* spec 需要更新（行为变更）
- 2 个新 spec（research-platform、policy-reports）

**回滚方案**：
- 前端：通过 Git 回退 commit 即可恢复旧首页和旧路由
- 后端：政策报告相关表和数据可通过数据库迁移回滚脚本清除
