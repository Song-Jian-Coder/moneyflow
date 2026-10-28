# Tasks

## 1. 前端组件迁移

- [x] 1.1 创建 `frontend/components/research/` 目录，将 `IndustryHeatmap.tsx`、`IndustryTreemap.tsx`、`TimeSelector.tsx`、`SectorDetailPanel.tsx`、`UpdateTimestamp.tsx`、`Breadcrumb.tsx` 从 `components/homepage/` 复制到 `components/research/`，验证文件复制成功
- [x] 1.2 更新 `components/research/` 下所有组件的 import 路径（如有引用 `@/components/homepage/` 的改为 `@/components/research/`），验证 TypeScript 编译无错误
- [x] 1.3 删除 `components/homepage/NavigationCards.tsx`，验证无其他文件引用该组件

## 2. 研究平台布局组件

- [x] 2.1 创建 `frontend/components/research/ResearchLayout.tsx`，实现左侧固定导航栏（240px）+ 右侧内容区布局，包含五个导航项（宏观数据、申万行业分类、资金流向(实时)、资金流向、策略库），验证组件渲染正常
- [x] 2.2 在 `ResearchLayout.tsx` 中实现导航项点击切换逻辑（React state），点击不同导航项时右侧内容区显示对应标识，验证切换交互正常
- [x] 2.3 为 `ResearchLayout.tsx` 添加响应式适配：移动端（<768px）导航栏收起为汉堡菜单，验证移动端布局正确

## 3. 研究平台路由和页面

- [x] 3.1 创建 `frontend/app/research/layout.tsx`，嵌套布局包裹 ResearchLayout 组件，验证 `/research` 路由可访问
- [x] 3.2 创建 `frontend/app/research/page.tsx`，默认展示「宏观数据」子模块（占位内容），验证页面渲染正常
- [x] 3.3 在 `research/page.tsx` 中集成「申万行业分类」子模块（使用迁移后的 IndustryTreemap 组件），验证树状图正常展示
- [x] 3.4 在 `research/page.tsx` 中集成「资金流向」子模块（使用迁移后的 IndustryHeatmap + TimeSelector 组件），验证热力图和时间选择器正常展示
- [x] 3.5 在 `research/page.tsx` 中集成「资金流向（实时）」子模块（占位，展示「实时数据接入中」提示），验证占位内容正常展示
- [x] 3.6 在 `research/page.tsx` 中集成「策略库」子模块（占位，展示「策略库功能即将上线」提示），验证占位内容正常展示

## 4. 首页重构

- [x] 4.1 重写 `frontend/components/homepage/HeroSection.tsx`，改为左右分栏布局：左侧展示「更专业、更专注」主标语 + 「资金流汇聚网站」副标语，右侧展示静态渐变背景，移除搜索框，验证组件渲染正常
- [x] 4.2 重写 `frontend/components/homepage/HomepageLayout.tsx` 的导航栏，将导航链接改为「首页 | 政策报告 | 资金流研究平台」，验证导航链接正确
- [x] 4.3 重写 `frontend/app/page.tsx`，仅包含 HeroSection + Footer，移除所有数据可视化组件引用，验证首页为极简品牌落地页

## 5. 删除旧路由

- [x] 5.1 删除 `frontend/app/industry-flow/` 目录，验证目录已删除
- [x] 5.2 删除 `frontend/app/market-flow/` 目录，验证目录已删除
- [x] 5.3 删除 `frontend/app/ai-assistant/` 目录，验证目录已删除
- [x] 5.4 运行 `npm run build`（或 `npx tsc --noEmit`），验证无编译错误和无用引用

## 6. 后端政策报告模块

- [x] 6.1 在 `moneyflow-model` 中创建 `PolicyReport.java` 实体类（字段：id、title、summary、category、source、publishDate、sourceUrl、documentUrl、createdAt、updatedAt），验证实体类编译通过
- [x] 6.2 在 `moneyflow-model` 中创建 `PolicyReportVO.java` 和 `PolicyReportQueryRequest.java`（DTO），验证 DTO 类编译通过
- [x] 6.3 在 `moneyflow-dao` 中创建 `PolicyReportRepository.java`（继承 JpaRepository），包含按分类分页查询方法，验证 Repository 编译通过
- [x] 6.4 在 `moneyflow-service` 中创建 `PolicyReportService.java` 接口和 `PolicyReportServiceImpl.java` 实现类（分页查询、详情查询），验证 Service 编译通过
- [x] 6.5 在 `moneyflow-service` 中创建 `PolicyReportCrawlerService.java`（爬取逻辑框架，MVP 阶段先实现手动触发接口），验证 Service 编译通过
- [x] 6.6 在 `moneyflow-web` 中创建 `PolicyReportController.java`，实现 `GET /api/policy-reports`（分页列表）、`GET /api/policy-reports/{id}`（详情）、`POST /api/policy-reports/crawl`（手动爬取）三个接口，验证 Controller 编译通过
- [x] 6.7 在 H2 开发数据库中创建 policy_report 表（通过 JPA auto-ddl 或手动 SQL），验证表创建成功

## 7. 前端政策报告页面

- [x] 7.1 创建 `frontend/app/policy-reports/page.tsx`，实现政策报告页面布局（分类筛选栏 + 报告列表），验证页面可访问
- [x] 7.2 在政策报告页面中集成分类筛选功能（产业政策、工作会议、宏观经济等），点击分类后列表刷新，验证筛选交互正常
- [x] 7.3 在政策报告页面中实现报告列表项展示（标题、摘要、分类标签、来源、日期），验证列表渲染正常
- [x] 7.4 实现报告详情查看（点击列表项展示完整详情），验证详情展示正常
- [x] 7.5 实现空状态展示（无数据时显示「暂无相关政策报告」），验证空状态正常

## 8. 全局导航更新

- [x] 8.1 更新根 `layout.tsx` 或全局 Header 组件，确保所有页面（首页、研究平台、政策报告、Todos）的顶部导航栏一致显示「首页 | 政策报告 | 资金流研究平台」，验证各页面导航正确
- [x] 8.2 更新 `not-found.tsx` 页面中的返回链接，确保指向正确的首页路径，验证 404 页面正常

## 9. 集成验证

- [x] 9.1 启动前端开发服务器（`npm run dev`），访问首页验证品牌落地页展示正常（标语、渐变背景、无搜索框）
- [x] 9.2 访问 `/research` 验证左侧导航切换五个子模块功能正常
- [x] 9.3 访问 `/policy-reports` 验证政策报告页面展示正常（空状态或 mock 数据）
- [x] 9.4 验证旧路由 `/industry-flow`、`/market-flow`、`/ai-assistant` 返回 404
- [x] 9.5 启动后端服务（`mvn spring-boot:run`），验证政策报告 API 接口可访问
- [x] 9.6 运行全量 TypeScript 编译检查（`npx tsc --noEmit`），验证无编译错误
