# Design

## Context

当前前端 `lib/request.ts` 已实现基础 HTTP 封装（fetch + Result 解析 + get/post/put/delete 方法），`types/api.ts` 定义了 `Result<T>` 泛型，`types/todo.ts` 定义了 Todo 模块专用类型。但首页模块所需的共享基础设施（工具函数、业务类型、常量、通用 UI 组件）完全缺失。

前置依赖：`homepage-project-setup` 变更负责安装 `clsx`、`tailwind-merge`、`date-fns` 等依赖包和初始化 shadcn/ui。本次变更消费这些依赖，不重复安装。

## Goals / Non-Goals

**Goals:**
- 建立首页模块共享的代码基础设施，使 8 个业务模块可统一导入工具、类型、常量和 UI 组件
- 在现有 `lib/request.ts` 基础上增强 API 封装（超时控制、便捷方法），保持向后兼容
- 所有新增代码 TypeScript 类型安全，零 `any` 类型

**Non-Goals:**
- 不实现具体业务模块（Hero、热力图、树状图等）
- 不修改 Todo 模块的现有代码和接口
- 不处理后端 API 的实际数据返回（本次仅定义前端封装层）
- 不安装新依赖（由 `homepage-project-setup` 负责）

## Decisions

### 决策 1：文件组织 — 按职责分文件

```
frontend/
├── lib/
│   ├── request.ts          # [修改] 增强超时控制 + apiGet/apiPost 便捷方法
│   ├── utils.ts            # [新增] cn() 类名合并工具
│   ├── format.ts           # [新增] formatMoney/formatPercent/formatDate/formatRelativeTime/formatNumber/truncateText
│   └── constants.ts        # [新增] BREAKPOINTS/ANIMATION_DURATION/DEBOUNCE_DELAY/PAGINATION/COLORS/API_PATHS
├── types/
│   ├── api.ts              # [不变] Result<T> 保持原样
│   ├── todo.ts             # [不变] Todo 类型保持原样
│   └── homepage.ts         # [新增] TimeRange/IndustryData/StockData/FlowDirection/PaginationParams/PaginatedResponse/SortOrder
└── components/
    ├── ui/                 # [由 project-setup 创建目录]
    │   ├── loading-skeleton.tsx   # [新增]
    │   ├── error-message.tsx      # [新增]
    │   └── empty-state.tsx        # [新增]
    └── todo/               # [不变]
```

**理由**：按职责分离而非按模块聚合。工具函数、类型、常量各自独立文件，便于 tree-shaking 和按需导入。UI 组件放在 `components/ui/` 与 shadcn/ui 组件共存。

**替代方案**：将所有工具函数放入单个 `lib/homepage.ts`。缺点是文件过大、无法按需导入、不利于 tree-shaking。

### 决策 2：API 封装增强策略 — 原地增强而非新建

在现有 `lib/request.ts` 上增加超时控制和 `apiGet`/`apiPost` 便捷方法，而非创建新的 `lib/homepage-api.ts`。

**理由**：
- 避免项目中存在两套 HTTP 客户端
- 现有 `api.get/post/put/delete` 接口保持不变，Todo 模块零影响
- `apiGet`/`apiPost` 作为额外导出，供首页模块使用

**超时实现**：使用 `AbortController` + `setTimeout`，fetch 原生支持 abort。

```typescript
// 关键接口定义
async function request<T>(
  url: string,
  options?: RequestInit,
  timeout?: number  // 默认 10000ms
): Promise<Result<T>>

export function apiGet<T>(url: string, params?: Record<string, string | number | boolean | undefined>): Promise<T>
export function apiPost<T>(url: string, data: unknown): Promise<T>
```

**注意**：`apiGet`/`apiPost` 直接返回 `T`（已解包 data 字段），而 `api.get`/`api.post` 返回 `Result<T>`（保留完整响应结构）。两者共存，首页模块用前者，Todo 模块用后者。

### 决策 3：日期格式化 — 使用 date-fns

使用 `date-fns` 库实现 `formatDate` 和 `formatRelativeTime`，而非手写正则或 Intl API。

**理由**：
- `homepage-project-setup` 已规划安装 `date-fns`
- date-fns 支持 tree-shaking（仅打包用到的函数）
- `formatDistanceToNow` 天然支持相对时间（"X分钟前"）
- 比 moment.js 轻量（date-fns ~7KB gzipped vs moment ~70KB）

### 决策 4：通用 UI 组件 — 纯 Tailwind 实现

LoadingSkeleton、ErrorMessage、EmptyState 使用纯 Tailwind CSS + React props 实现，不依赖 shadcn/ui 组件。

**理由**：
- 这三个组件高度定制化（骨架屏形状、错误图标、空状态插图），shadcn/ui 无对应组件
- 纯 Tailwind 实现更轻量，无需额外依赖
- 通过 `cn()` 工具支持 `className` 参数，保持与 Tailwind 生态一致

### 决策 5：常量使用 `as const` 断言

所有常量使用 `as const` 确保字面量类型推导。

```typescript
// 关键接口定义
export const BREAKPOINTS = {
  mobile: 768,
  tablet: 1024,
  desktop: 1024,
} as const

export const COLORS = {
  brand: { primary: '#2563eb', secondary: '#9333ea', accent: '#f97316' },
  flow: { inflow: '#dc2626', outflow: '#16a34a', neutral: '#6b7280' },
} as const
```

**理由**：`as const` 使常量值在编译时可知，TypeScript 可在使用处进行类型检查（如 `BREAKPOINTS.mobile` 推导为 `768` 而非 `number`）。

## Risks / Trade-offs

**[风险] `apiGet`/`apiPost` 与 `api.get`/`api.post` 双接口共存**
→ 缓解：在 JSDoc 中明确标注使用场景——首页模块用 `apiGet`/`apiPost`（返回解包后的 T），Todo 模块继续用 `api.get`/`api.post`（返回 Result<T>）。后续统一迁移时再废弃旧接口。

**[风险] `homepage-project-setup` 未执行时依赖缺失**
→ 缓解：本次变更的 tasks.md 中不重复安装依赖，但明确标注前置条件。若 project-setup 未执行，`cn()` 和 `formatDate` 会因缺少 `clsx`/`tailwind-merge`/`date-fns` 而编译失败。

**[风险] 类型定义可能与后端 API 实际返回不匹配**
→ 缓解：当前类型定义基于 spec 中的字段描述，后端 API 尚未实现。在首页模块联调时可能需要调整类型字段。保持类型文件集中管理，修改成本低。
