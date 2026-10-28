# Proposal

## Why

首页模块包含 8 个业务单元（Hero、导航、时间选择器、热力图、树状图、详情面板等），它们共享大量基础设施需求：工具函数（cn、格式化）、类型定义（TimeRange、IndustryData）、常量配置（断点、颜色、API 路径）、API 封装和通用 UI 组件（LoadingSkeleton、ErrorMessage、EmptyState）。当前前端项目中这些共享能力为零——`lib/request.ts` 仅有基础 HTTP 封装，无工具函数、无业务类型、无通用组件。若各模块独立实现，将导致大量重复代码和接口不一致。本次变更建立首页共享基础设施，使后续 8 个业务模块可在统一规范上开发，避免重复建设。

## What Changes

- **新增** 工具函数层：`cn()` 类名合并工具（clsx + tailwind-merge）、金额/百分比/日期/相对时间/千分位/文本截断格式化工具函数
- **新增** 类型定义层：TimeRange、IndustryData、StockData、FlowDirection、PaginationParams、PaginatedResponse、SortOrder 等首页业务类型
- **新增** 常量定义层：BREAKPOINTS（响应式断点）、ANIMATION_DURATION（动画时长）、DEBOUNCE_DELAY（防抖延迟）、PAGINATION（分页配置）、COLORS（品牌色+资金流向色）、API_PATHS（接口路径分组）
- **增强** API 封装层：在现有 `lib/request.ts` 基础上增加超时控制（10s）、便捷方法 `apiGet`/`apiPost`，统一 Result 格式解析和错误处理
- **新增** 通用 UI 组件层：LoadingSkeleton（card/chart/table/text 四种类型）、ErrorMessage（含重试按钮）、EmptyState（含自定义图标）

## Capabilities

### New Capabilities

（无新增 capability）

### Modified Capabilities

- `homepage-shared-foundation`：从当前的单需求 stub（仅定义"共享基础设施"概览）扩展为完整的 6 个需求规格——类名合并工具、格式化工具函数、API 请求封装、首页业务类型定义、共享常量定义、通用 UI 组件。覆盖首页所有业务模块的共享依赖。

## Impact

- **前端子仓库**（`frontend/`）：
  - 新增文件：`lib/utils.ts`（cn 工具）、`lib/format.ts`（格式化工具函数）、`lib/constants.ts`（共享常量）、`types/homepage.ts`（首页业务类型）、`components/ui/loading-skeleton.tsx`、`components/ui/error-message.tsx`、`components/ui/empty-state.tsx`
  - 修改文件：`lib/request.ts`（增强超时控制和便捷方法）
  - 新增依赖：`clsx`、`tailwind-merge`（cn 工具所需）、`date-fns`（日期格式化所需）
- **后端 API**：零影响（本次变更仅涉及前端共享基础设施）
- **Todo 模块**：零影响（现有 `lib/request.ts` 增强向后兼容，Todo 模块继续使用 `api.get/post/put/delete`）
- **依赖变更**：`homepage-project-setup` 中任务 1.1 已规划安装 `clsx`、`tailwind-merge`、`date-fns`，本次变更消费这些依赖
- **下游模块**：首页 8 个业务模块（hero、navigation、time-selector、heatmap、treemap、sector-detail-panel、update-timestamp、responsive-layout）均依赖本次变更提供的共享基础

## 回滚方案

- 所有新增文件为独立模块，删除不影响现有 Todo 功能和后端 API
- `lib/request.ts` 的增强向后兼容，回滚时恢复原文件即可
- 新增依赖（clsx、tailwind-merge、date-fns）通过 `package.json` 管理，`npm install` 可恢复
