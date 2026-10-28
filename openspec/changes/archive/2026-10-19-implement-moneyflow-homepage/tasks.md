# Tasks

## 1. 依赖安装与基础设施

- [x] 1.1 在 `frontend/` 目录执行 `npm install echarts echarts-for-react`，验证 `package.json` 包含这两个依赖
- [x] 1.2 创建 `frontend/lib/mock/homepage.ts`，导出 `MockHomepageService` 对象，包含 `getHeatmapData` / `getTreemapData` / `getIndustryDetail` / `getUpdateTime` 四个方法，方法体返回符合 `IndustryData` / `StockData` 类型的 Mock 数据（使用 `types/homepage.ts` 中已定义的类型），验证 TypeScript 编译无错误
- [x] 1.3 在 `frontend/lib/mock/homepage.ts` 中为 `getHeatmapData` 生成 100+ 条 Mock 行业数据（覆盖申万二级行业，包含 turnover / netInflow / changePercent 字段），验证数据量符合热力图渲染需求
- [x] 1.4 在 `frontend/lib/mock/homepage.ts` 中为 `getTreemapData` 生成层级 Mock 数据（板块 → 一级 → 二级 → 三级 → 股票），验证 `parentIndustry` 字段关联正确
- [x] 1.5 创建 `frontend/context/TimeRangeContext.tsx`，定义 `TimeRangeProvider` 和 `useTimeRange` Hook，管理全局 `TimeRange` 状态（默认值为“今日”），验证组件可正确订阅和更新状态

## 2. 布局与静态组件（P0）

- [x] 2.1 创建 `frontend/components/homepage/HomepageLayout.tsx`，实现响应式布局容器（Header + 主内容区 + Footer），使用 Tailwind `sm:` / `md:` / `lg:` 断点，验证三端布局正常（PC 居中最大宽度 1280px，平板 padding 24px，移动 padding 16px）
- [x] 2.2 创建 `frontend/components/homepage/HeroSection.tsx`，实现品牌背景图 + 主副标语 + 搜索框，背景图加载失败时降级为渐变色（使用 `COLORS.brand`），验证 Desktop / Tablet / Mobile 三端高度分别为 500-600px / 400-500px / 300-400px
- [x] 2.3 创建 `frontend/components/homepage/NavigationCards.tsx`，实现三张功能导航卡片（行业资金流 / 大盘资金流 / AI 助手），每张卡片包含图标（lucide-react）+ 标题 + 描述 + 跳转链接，验证点击跳转正确（`/industry-flow` / `/market-flow` / `/ai-assistant`），hover 和 focus 状态视觉反馈明确
- [x] 2.4 创建 `frontend/components/homepage/UpdateTimestamp.tsx`，调用 `MockHomepageService.getUpdateTime()` 获取时间戳，使用 `formatDate` 格式化为 "yyyy-MM-dd HH:mm"，展示"数据更新时间：xxx（每日收盘后更新）"，验证时间正确展示，API 失败时显示"更新时间未知"
- [x] 2.5 修改 `frontend/app/page.tsx`，替换占位内容为 `HomepageLayout` 组合（HeroSection + NavigationCards + UpdateTimestamp），验证访问 `http://localhost:3000` 看到完整静态首页

## 3. 时间选择器（P1）

- [x] 3.1 创建 `frontend/components/homepage/TimeSelector.tsx`，实现四个快捷按钮（今日 / 昨日 / 近 7 日 / 近 30 日），点击后更新 `TimeRangeContext`，验证默认选中“今日”，切换后 `displayName` 正确更新
- [x] 3.2 在 `TimeSelector.tsx` 中实现自定义日期范围选择器（两个 `<input type="date">`），验证起始日期晚于结束日期时显示错误提示“起始日期不能晚于结束日期”，不触发 onChange
- [x] 3.3 在 `TimeSelector.tsx` 中实现时间范围校验：超过 365 天显示提示“时间范围不能超过 365 天”，早于 minDate（2020-01-01）显示提示“数据最早可追溯至 2020-01-01”，验证校验逻辑正确
- [x] 3.4 在 `TimeSelector.tsx` 中实现交易日处理：今天为非交易日（周末）时自动回退到最近交易日，显示提示““今日为非交易日，已显示最近交易日数据”，验证回退逻辑正确

## 4. 热力图（P1）

- [x] 4.1 创建 `frontend/components/homepage/IndustryHeatmap.tsx`，使用 `echarts-for-react` 渲染热力图，消费 `MockHomepageService.getHeatmapData(timeRange)` 数据，验证 100+ 行业色块正确渲染
- [x] 4.2 在 `IndustryHeatmap.tsx` 中实现面积编码（色块面积 ∝ turnover）和颜色编码（净流入 > 0 红色系，< 0 绿色系，使用 `COLORS.flow`），验证颜色映射正确
- [x] 4.3 在 `IndustryHeatmap.tsx` 中实现交互：点击色块触发 `onIndustryClick(industryCode)`，hover 显示 tooltip（行业名称 + 净流入额），鼠标指针变为 pointer，验证事件回调正确
- [x] 4.4 在 `IndustryHeatmap.tsx` 中实现图例：底部或侧边展示颜色图例（红=流入，绿=流出）和面积说明（色块面积代表成交额），验证图例文案清晰
- [x] 4.5 在 `IndustryHeatmap.tsx` 中实现异常处理：数据为空时显示 `<EmptyState title="暂无行业数据" />`，色块面积 < 40x40px 时不显示文字，验证降级展示正常

## 5. 树状图（P1）

- [x] 5.1 创建 `frontend/components/homepage/IndustryTreemap.tsx`，使用 `echarts-for-react` 渲染树状图，消费 `MockHomepageService.getTreemapData(timeRange)` 数据，验证初始视图展示板块级（一级行业）
- [x] 5.2 在 `IndustryTreemap.tsx` 中实现逐级下钻：点击一级 → 展示二级，点击二级 → 展示三级，点击三级 → 展示股票，动画过渡 300ms（使用 `ANIMATION_DURATION.normal`），验证下钻逻辑正确
- [x] 5.3 创建 `frontend/components/homepage/Breadcrumb.tsx`，实现面包屑导航（“申万板块 > 一级行业 > 二级行业”），点击上层节点返回对应层级，验证导航与树状图状态同步
- [x] 5.4 在 `IndustryTreemap.tsx` 中实现异常处理：某层级无子节点时显示 `<EmptyState title="该行业下暂无数据" />`，点击股票色块不触发下钻而是触发 `onStockClick(stockCode)`，验证降级和回调正确
- [x] 5.5 在 `IndustryTreemap.tsx` 中实现面积编码和颜色编码（逻辑同热力图），色块最小面积 20x20px，验证视觉编码一致

## 6. 详情面板（P2）

- [x] 6.1 创建 `frontend/components/homepage/SectorDetailPanel.tsx`，接收 `industryCode` 和 `timeRange` props，调用 `MockHomepageService.getIndustryDetail(industryCode, timeRange)` 获取数据，验证面板正确展示行业名称、代码、层级、成交额、净流入、涨跌幅、成分股数量
- [x] 6.2 在 `SectorDetailPanel.tsx` 中实现成分股列表，使用 `<Table>` 组件（shadcn/ui），默认按主力净流入降序排列，验证列表数据正确
- [x] 6.3 在 `SectorDetailPanel.tsx` 中实现表头排序：点击“成交额” / “净流入”表头切换排序字段和方向（首次降序，再次升序），显示排序箭头标识，验证排序逻辑正确
- [x] 6.4 在 `SectorDetailPanel.tsx` 中实现关闭交互：点击 X 按钮关闭、点击遮罩层关闭、按 ESC 键关闭，动画流畅（使用 `ANIMATION_DURATION.normal`），验证三种关闭方式均生效
- [x] 6.5 在 `SectorDetailPanel.tsx` 中实现异常处理：板块不存在时显示“板块不存在”，成分股为 0 时显示 `<EmptyState title="该板块暂无成分股数据" />`，验证降级展示正常

## 7. 图表联动与集成（P2）

- [x] 7.1 修改 `frontend/app/page.tsx`，将 `TimeSelector` / `IndustryHeatmap` / `IndustryTreemap` / `SectorDetailPanel` 组装到 `HomepageLayout` 中，验证所有组件正确渲染
- [x] 7.2 在 `frontend/app/page.tsx` 中实现联动逻辑：时间选择器切换 → 热力图和树状图重新请求数据并渲染，点击热力图 / 树状图色块 → 打开详情面板，验证联动正确
- [x] 7.3 在 `frontend/app/page.tsx` 中使用 `<LoadingSkeleton type="chart" />` 作为图表加载占位，使用 `<ErrorMessage />` 作为数据加载失败提示，验证加载和错误状态展示正常

## 8. 响应式与移动端适配（P2）

- [x] 8.1 验证 `HomepageLayout` 在 PC 端（>= 1024px）布局正常：Header 固定顶部 60-80px，Hero 区 400-600px，热力图和树状图并排或上下排列，间距 24px
- [x] 8.2 验证 `HomepageLayout` 在平板端（768px - 1023px）布局正常：Header 高度 50-60px，Hero 区 300-400px，内容区 padding 24px
- [x] 8.3 验证 `HomepageLayout` 在移动端（< 768px）布局正常：Header 包含汉堡菜单，Hero 区 250-350px，导航卡片纵向堆叠，主内容区占满宽度 padding 16px
- [x] 8.4 在 `IndustryHeatmap.tsx` 和 `IndustryTreemap.tsx` 中实现基础触摸手势：双指缩放（0.5x - 3x）、单指拖动，使用 ECharts 内置 dataZoom 或自定义手势处理，验证移动端交互流畅

## 9. 构建与验收

- [x] 9.1 执行 `cd frontend; npx tsc --noEmit`，验证所有新增文件 TypeScript 编译无错误，无 `any` 类型
- [x] 9.2 执行 `cd frontend; npm run build`，验证 Next.js 生产构建成功，无编译错误
- [x] 9.3 执行 `cd frontend; npm run dev`，访问 `http://localhost:3000`，按 proposal 验收标准逐项检查功能、视觉、交互、响应式、性能，验证所有验收标准通过

## Workflow follow-up

- 归档变更：实现完成并通过审查后执行 `openspec archive`
- 同步规范：归档时将 9 个 delta spec 合并到 `openspec/specs/homepage-*/spec.md`
- 后端 API 变更：另行发起 `openspec propose` 创建后端数据 API 变更（行业数据 / 资金流数据实体与接口）
