# Spec Delta — homepage-shared-foundation

## MODIFIED Requirements

### Requirement: 共享基础设施
前端 SHALL 为所有首页业务模块提供共享的工具函数、类型定义、常量配置、API 封装和通用 UI 组件，确保代码一致性和开发效率。

#### Scenario: 共享工具可用
- **WHEN** 首页业务模块需要格式化工具、类名合并等通用能力
- **THEN** 可从共享基础设施导入对应工具函数

## ADDED Requirements

### Requirement: 类名合并工具
前端 SHALL 提供 `cn()` 工具函数，基于 clsx + tailwind-merge 实现 Tailwind CSS 类名的安全合并。

#### Scenario: 多类名合并
- **WHEN** 调用 `cn('px-4', 'py-2', isActive && 'bg-blue-500')`
- **THEN** 返回合并后的类名字符串，falsy 值被过滤

#### Scenario: 冲突类名处理
- **WHEN** 调用 `cn('text-red-500', 'text-blue-500')`
- **THEN** 返回 `'text-blue-500'`，后者覆盖前者，不产生冲突

### Requirement: 格式化工具函数
前端 SHALL 提供金额、日期、数字、文本等格式化工具函数，支持中文本地化。

#### Scenario: 金额格式化
- **WHEN** 调用 `formatMoney(value)`
- **THEN** 绝对值 >= 1万亿 显示为 "X.XX万亿"
- **AND** 绝对值 >= 1亿 显示为 "X.XX亿"
- **AND** 绝对值 >= 1万 显示为 "X.XX万"
- **AND** 其余显示为 "X.XX"（保留 2 位小数）

#### Scenario: 百分比格式化
- **WHEN** 调用 `formatPercent(0.1234)`
- **THEN** 返回 `"12.34%"`

#### Scenario: 日期格式化
- **WHEN** 调用 `formatDate(date, formatStr)`
- **THEN** 按指定格式返回日期字符串，默认格式为 `yyyy-MM-dd HH:mm`

#### Scenario: 相对时间格式化
- **WHEN** 调用 `formatRelativeTime(date)`
- **THEN** 小于 1 分钟返回"刚刚"，小于 60 分钟返回"X分钟前"，小于 24 小时返回"X小时前"，其余返回格式化日期

#### Scenario: 数字千分位格式化
- **WHEN** 调用 `formatNumber(1234567)`
- **THEN** 返回 `"1,234,567"`

#### Scenario: 文本截断
- **WHEN** 调用 `truncateText(text, maxLength)` 且文本超出长度
- **THEN** 截断并追加省略号 `…`

### Requirement: API 请求封装
前端 SHALL 提供统一的 API 请求封装，支持超时控制、Result 格式解析和错误处理，与现有 `lib/request.ts` 向后兼容。

#### Scenario: 统一请求处理
- **WHEN** 发送任意 HTTP 请求
- **THEN** 请求自动设置 `Content-Type: application/json`
- **AND** 默认超时 10 秒，超时后抛出超时错误

#### Scenario: 响应格式解析
- **WHEN** 收到后端响应且 `result.code === 200`
- **THEN** 返回 `result.data` 字段值

#### Scenario: 业务错误处理
- **WHEN** 收到后端响应且 `result.code !== 200`
- **THEN** 抛出包含 code 和 message 的错误对象

#### Scenario: HTTP 错误处理
- **WHEN** HTTP 响应状态码非 2xx
- **THEN** 抛出包含 HTTP 状态码的错误对象

#### Scenario: 便捷方法
- **WHEN** 调用 `apiGet(url, params)` 或 `apiPost(url, data)`
- **THEN** 自动拼接查询参数（GET）或序列化请求体（POST）

#### Scenario: 向后兼容
- **WHEN** 现有 Todo 模块调用 `api.get()`、`api.post()`、`api.put()`、`api.delete()`
- **THEN** 调用方式与返回值格式保持不变

### Requirement: 首页业务类型定义
前端 SHALL 定义首页模块共享的 TypeScript 类型接口，覆盖时间范围、行业数据、股票数据、资金流向和分页。

#### Scenario: 时间范围类型
- **WHEN** 定义 TimeRange 类型
- **THEN** SHALL 包含 type 字段（枚举：today、yesterday、recent7、recent30、custom）
- **AND** SHALL 包含 startDate 和 endDate（ISO 8601: yyyy-MM-dd）
- **AND** SHALL 包含 displayName 字段

#### Scenario: 行业数据类型
- **WHEN** 定义 IndustryData 类型
- **THEN** SHALL 包含 industryCode、industryName、parentIndustry、level（1=板块, 2=一级, 3=二级, 4=三级）
- **AND** SHALL 包含 turnover、netInflow、changePercent、stockCount 数值字段

#### Scenario: 股票数据类型
- **WHEN** 定义 StockData 类型
- **THEN** SHALL 包含 stockCode、stockName、industryCode 标识字段
- **AND** SHALL 包含 turnover、netInflow、changePercent、marketCap、peRatio、pbRatio 数值字段

#### Scenario: 资金流向类型
- **WHEN** 定义 FlowDirection 类型
- **THEN** SHALL 为 `'inflow' | 'outflow' | 'neutral'` 联合类型
- **AND** SHALL 提供 `getFlowDirection(netInflow)` 函数：正值为 inflow，负值为 outflow，零为 neutral

#### Scenario: 分页与排序类型
- **WHEN** 定义分页相关类型
- **THEN** PaginationParams SHALL 包含 pageNum 和 pageSize
- **AND** PaginatedResponse<T> SHALL 包含 records、total、pageNum、pageSize、totalPages
- **AND** SortOrder SHALL 为 `'asc' | 'desc'`

### Requirement: 共享常量定义
前端 SHALL 定义首页模块共享的常量配置，使用 `as const` 确保类型安全。

#### Scenario: 响应式断点常量
- **WHEN** 引用 BREAKPOINTS 常量
- **THEN** SHALL 包含 mobile: 768, tablet: 1024, desktop: 1024

#### Scenario: 动画时长常量
- **WHEN** 引用 ANIMATION_DURATION 常量
- **THEN** SHALL 包含 fast: 150ms, normal: 300ms, slow: 500ms

#### Scenario: 防抖延迟常量
- **WHEN** 引用 DEBOUNCE_DELAY 常量
- **THEN** SHALL 包含 search: 300ms, resize: 200ms, scroll: 100ms

#### Scenario: 分页配置常量
- **WHEN** 引用 PAGINATION 常量
- **THEN** SHALL 包含 defaultPageSize: 20, maxPageSize: 100, pageSizeOptions: [10, 20, 50, 100]

#### Scenario: 颜色配置常量
- **WHEN** 引用 COLORS 常量
- **THEN** SHALL 包含品牌色（primary: #2563eb, secondary: #9333ea, accent: #f97316）
- **AND** SHALL 包含资金流向色（inflow: #dc2626, outflow: #16a34a, neutral: #6b7280）

#### Scenario: API 路径常量
- **WHEN** 引用 API_PATHS 常量
- **THEN** SHALL 按资源分组：industry（treemap/heatmap/detail）、stock（list/detail）、data（updateTime）

### Requirement: 通用 UI 组件
前端 SHALL 提供 LoadingSkeleton、ErrorMessage、EmptyState 三个通用 UI 组件，用于统一首页的加载、错误和空状态展示。

#### Scenario: 加载骨架屏
- **WHEN** 渲染 LoadingSkeleton 组件
- **THEN** SHALL 支持 type 参数：card（默认）、chart、table、text
- **AND** table/text 类型 SHALL 支持 rows 参数控制行数
- **AND** 使用 Tailwind animate-pulse 动画效果

#### Scenario: 错误提示
- **WHEN** 渲染 ErrorMessage 组件
- **THEN** SHALL 展示错误信息文本（默认："数据加载失败，请稍后重试"）
- **AND** 可选展示重试按钮，点击触发 onRetry 回调

#### Scenario: 空状态提示
- **WHEN** 渲染 EmptyState 组件
- **THEN** SHALL 展示标题（默认："暂无数据"）和描述文本
- **AND** SHALL 支持自定义图标

#### Scenario: 组件样式自定义
- **WHEN** 使用任意共享 UI 组件
- **THEN** SHALL 支持 className 参数，可与 Tailwind CSS 类名合并
