# Tasks

> **前置条件**：`homepage-project-setup` 变更已完成，`clsx`、`tailwind-merge`、`date-fns` 已安装在 `frontend/node_modules` 中，`components/ui/` 目录已创建。

## 1. 类型定义

- [x] 1.1 创建 `frontend/types/homepage.ts`：定义 TimeRange 类型（type 枚举 today/yesterday/recent7/recent30/custom + startDate + endDate + displayName），验证 TypeScript 编译无错误
- [x] 1.2 在 `frontend/types/homepage.ts` 中定义 IndustryData 类型（industryCode + industryName + parentIndustry + level 1-4 + turnover + netInflow + changePercent + stockCount），验证 TypeScript 编译无错误
- [x] 1.3 在 `frontend/types/homepage.ts` 中定义 StockData 类型（stockCode + stockName + industryCode + turnover + netInflow + changePercent + marketCap + peRatio + pbRatio），验证 TypeScript 编译无错误
- [x] 1.4 在 `frontend/types/homepage.ts` 中定义 FlowDirection 联合类型（'inflow' | 'outflow' | 'neutral'）和 getFlowDirection 函数（正值→inflow，负值→outflow，零→neutral），验证 getFlowDirection(100) === 'inflow'、getFlowDirection(-50) === 'outflow'、getFlowDirection(0) === 'neutral'
- [x] 1.5 在 `frontend/types/homepage.ts` 中定义 PaginationParams（pageNum + pageSize）、PaginatedResponse<T>（records + total + pageNum + pageSize + totalPages）、SortOrder（'asc' | 'desc'），验证 TypeScript 编译无错误

## 2. 共享常量

- [x] 2.1 创建 `frontend/lib/constants.ts`：定义 BREAKPOINTS 常量（mobile: 768, tablet: 1024, desktop: 1024）使用 `as const`，验证 `typeof BREAKPOINTS.mobile` 推导为字面量类型 `768`
- [x] 2.2 在 `frontend/lib/constants.ts` 中定义 ANIMATION_DURATION（fast: 150, normal: 300, slow: 500）和 DEBOUNCE_DELAY（search: 300, resize: 200, scroll: 100）使用 `as const`，验证常量值正确
- [x] 2.3 在 `frontend/lib/constants.ts` 中定义 PAGINATION（defaultPageSize: 20, maxPageSize: 100, pageSizeOptions: [10, 20, 50, 100]）使用 `as const`，验证常量值正确
- [x] 2.4 在 `frontend/lib/constants.ts` 中定义 COLORS 常量（brand: primary #2563eb / secondary #9333ea / accent #f97316，flow: inflow #dc2626 / outflow #16a34a / neutral #6b7280）使用 `as const`，验证颜色值正确
- [x] 2.5 在 `frontend/lib/constants.ts` 中定义 API_PATHS 常量（按 industry/stock/data 分组：treemap/heatmap/detail、list/detail、updateTime），验证路径字符串正确

## 3. 工具函数

- [x] 3.1 创建 `frontend/lib/utils.ts`：实现 cn() 函数（clsx + tailwind-merge），导入 `import { clsx } from 'clsx'` 和 `import { twMerge } from 'tailwind-merge'`，验证 `cn('px-4', 'py-2')` 返回 `'px-4 py-2'`、`cn('text-red-500', 'text-blue-500')` 返回 `'text-blue-500'`

## 4. 格式化工具函数

- [x] 4.1 创建 `frontend/lib/format.ts`：实现 formatMoney 函数（>=1万亿→"X.XX万亿"，>=1亿→"X.XX亿"，>=1万→"X.XX万"，其余→"X.XX"），验证 `formatMoney(150000000)` 返回 `"1.50亿"`、`formatMoney(5000)` 返回 `"5000.00"`
- [x] 4.2 在 `frontend/lib/format.ts` 中实现 formatPercent 函数（0.1234 → "12.34%"）和 formatNumber 函数（1234567 → "1,234,567"），验证输出格式正确
- [x] 4.3 在 `frontend/lib/format.ts` 中实现 formatDate 函数（使用 date-fns 的 format），默认格式 `yyyy-MM-dd HH:mm`，验证 `formatDate(new Date('2026-10-22T07:30:00Z'))` 返回正确格式
- [x] 4.4 在 `frontend/lib/format.ts` 中实现 formatRelativeTime 函数（使用 date-fns 的 formatDistanceToNow），<1分钟→"刚刚"，<60分钟→"X分钟前"，<24小时→"X小时前"，其余→格式化日期，验证各区间输出正确
- [x] 4.5 在 `frontend/lib/format.ts` 中实现 truncateText 函数（超出 maxLength 截断并追加 `…`），验证 `truncateText('hello world', 5)` 返回 `"hello…"`

## 5. API 封装增强

- [x] 5.1 修改 `frontend/lib/request.ts`：在 request 函数中增加超时控制参数（默认 10000ms），使用 AbortController + setTimeout 实现，超时后抛出包含 "请求超时" 信息的 Error，验证超时逻辑正确
- [x] 5.2 在 `frontend/lib/request.ts` 中导出 apiGet<T> 和 apiPost<T> 便捷方法：apiGet 自动拼接 URLSearchParams 查询参数，apiPost 自动 JSON.stringify 请求体，两者均返回解包后的 `T`（而非 Result<T>），验证返回值已解包
- [x] 5.3 验证向后兼容：确认 `frontend/lib/todo.ts` 中所有 `api.get/post/put/delete` 调用无 TypeScript 编译错误，确认 Todo 模块功能不受影响

## 6. 通用 UI 组件

- [x] 6.1 创建 `frontend/components/ui/loading-skeleton.tsx`：实现 LoadingSkeleton 组件，支持 type 参数（card/chart/table/text，默认 card），table/text 类型支持 rows 参数控制行数，使用 Tailwind animate-pulse，支持 className 参数，验证各 type 渲染正常
- [x] 6.2 创建 `frontend/components/ui/error-message.tsx`：实现 ErrorMessage 组件，展示错误信息文本（默认"数据加载失败，请稍后重试"），可选展示重试按钮（点击触发 onRetry 回调），支持 className 参数，验证默认文案和重试按钮功能正常
- [x] 6.3 创建 `frontend/components/ui/empty-state.tsx`：实现 EmptyState 组件，展示标题（默认"暂无数据"）和描述文本，支持自定义图标（icon prop），支持 className 参数，验证默认展示和自定义图标功能正常

## 7. 构建验证

- [x] 7.1 运行 `cd frontend; npx tsc --noEmit`，验证所有新增文件 TypeScript 编译无错误，无 `any` 类型
- [x] 7.2 运行 `cd frontend; npm run build`，验证 Next.js 生产构建成功，无编译错误
- [x] 7.3 验证导入路径：在首页临时导入 cn()、formatMoney、BREAKPOINTS、TimeRange、LoadingSkeleton 等，确认各模块可从正确路径导入，验证后移除测试代码

## Workflow follow-up

- 归档变更：实现完成并通过审查后执行 `openspec archive`
- 同步规范：归档时将 delta spec 合并到 `openspec/specs/homepage-shared-foundation/spec.md`
- 确认与 `2026-10-19-implement-moneyflow-homepage` 变更中 `homepage-shared-foundation` delta spec 的一致性，避免重复定义
