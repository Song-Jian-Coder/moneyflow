# Todo 待办清单 Specification

## Purpose

提供单用户私有的待办事项管理能力，支持待办的创建、查询、更新、删除、状态切换和批量操作。Todo 模块是 MoneyFlow 的第一个业务模块，验证前后端框架的基础设施能力（统一响应、异常处理、参数校验、前后端联调）。

## 模块边界

### 包含

| 功能 | 说明 |
|------|------|
| 待办 CRUD | 创建、查询（单条 + 列表）、更新、删除 |
| 状态切换 | 标记完成 / 标记未完成 |
| 列表筛选 | 按完成状态、优先级筛选；按创建时间、截止时间排序 |
| 批量操作 | 批量删除、批量标记完成 |
| 分页查询 | 列表接口支持分页 |

### 不包含

| 功能 | 说明 |
|------|------|
| 多用户 / 协作 | 不做共享、分配、权限隔离 |
| 提醒 / 通知 | 不做定时推送、邮件通知 |
| 重复任务 | 不做周期性 / 循环任务 |
| 子任务 | 不做任务嵌套、层级关系 |
| 标签 / 分类 | 不做多维度归类（后续可扩展） |
| 文件附件 | 不做附件上传 |
| 操作日志 | 不做审计追踪 |

## 数据结构

### 待办实体（TodoItem）

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| `id` | Long | 主键，自增 | 唯一标识 |
| `title` | String | 非空，1-200 字符 | 待办标题 |
| `description` | String | 可空，最大 2000 字符 | 待办详情 |
| `completed` | Boolean | 非空，默认 false | 是否已完成 |
| `priority` | Integer | 非空，枚举值 0/1/2，默认 1 | 优先级：0=低，1=中，2=高 |
| `dueDate` | LocalDateTime | 可空，必须为未来时间 | 截止时间 |
| `createdAt` | LocalDateTime(3) | 非空，自动生成，毫秒精度 | 创建时间 |
| `updatedAt` | LocalDateTime(3) | 非空，自动生成，毫秒精度 | 最后更新时间 |

> 数据库表字符集：`utf8mb4`，排序规则：`utf8mb4_unicode_ci`。

#### 索引定义

| 索引名 | 字段 | 类型 | 说明 |
|--------|------|------|------|
| `idx_completed` | `completed` | 普通索引 | 按完成状态筛选 |
| `idx_priority` | `priority` | 普通索引 | 按优先级筛选 |
| `idx_created_at` | `created_at` | 普通索引 | 默认排序 |
| `idx_due_date` | `due_date` | 普通索引 | 按截止时间排序 |

### 请求 / 响应 DTO

#### 创建待办 — CreateTodoRequest

```json
{
  "title": "完成项目文档",
  "description": "编写 Todo 模块的 Spec 文档",
  "priority": 2,
  "dueDate": "2026-10-20T18:00:00"
}
```

| 字段 | 类型 | 必填 | 约束 |
|------|------|------|------|
| `title` | String | 是 | 去首尾空格后 1-200 字符 |
| `description` | String | 否 | 最大 2000 字符 |
| `priority` | Integer | 否 | 0 / 1 / 2，默认 1 |
| `dueDate` | String (ISO 8601) | 否 | 格式 `yyyy-MM-ddTHH:mm:ss`，必须为未来时间 |

#### 更新待办 — UpdateTodoRequest

与 CreateTodoRequest 结构相同，所有字段均为可选（部分更新语义）。

> **"可选"语义说明**："可选"指字段可以不出现在 JSON 请求体中（即不传）。一旦字段出现在请求体中，则必须满足与创建时相同的校验规则（非空、长度、范围等）。例如 `title` 可以不传（保持不变），但传了空字符串 `""` 则校验失败返回 400。

#### 待办响应 — TodoItemVO

```json
{
  "id": 1,
  "title": "完成项目文档",
  "description": "编写 Todo 模块的 Spec 文档",
  "completed": false,
  "priority": 2,
  "dueDate": "2026-10-20T18:00:00",
  "createdAt": "2026-10-15T10:30:00",
  "updatedAt": "2026-10-15T10:30:00"
}
```

#### 列表查询参数 — TodoQueryRequest

| 字段 | 类型 | 必填 | 约束 | 默认值 |
|------|------|------|------|--------|
| `completed` | Boolean | 否 | — | 不过滤 |
| `priority` | Integer | 否 | 0 / 1 / 2 | 不过滤 |
| `sortBy` | String | 否 | 枚举：`createdAt` / `dueDate` / `priority` | `createdAt` |
| `sortOrder` | String | 否 | 枚举：`asc` / `desc` | `desc` |
| `pageNum` | Integer | 否 | >= 1 | 1 |
| `pageSize` | Integer | 否 | 1-100 | 20 |

#### 分页响应 — PageResult\<TodoItemVO\>

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "records": [ ... ],
    "total": 42,  // 类型：Long
    "pageNum": 1,
    "pageSize": 20,
    "totalPages": 3
  }
}
```

#### 批量操作请求 — BatchTodoRequest

```json
{
  "ids": [1, 2, 3]
}
```

| 字段 | 类型 | 必填 | 约束 |
|------|------|------|------|
| `ids` | List\<Long\> | 是 | 非空，1-100 个元素，每个 > 0，不允许重复 |

#### 批量操作响应 — BatchTodoResponse

批量操作（标记完成 / 删除）在部分成功时返回以下结构：

```json
{
  "successIds": [1, 2],
  "failedIds": [99999]
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `successIds` | List\<Long\> | 成功处理的 ID 列表 |
| `failedIds` | List\<Long\> | 未找到 / 处理失败的 ID 列表 |

- 全部成功：HTTP 200，`code=200`，`data` 为上述结构（`failedIds` 为空数组）
- 部分成功：HTTP 200，`code=207`，`data` 为上述结构
- 全部失败：HTTP 200，`code=207`，`data` 为上述结构（`successIds` 为空数组）

### 全局约束

- **请求体大小上限**：所有接口的 HTTP 请求体不超过 **64KB**
- **限流策略**：批量操作接口（`/batch/complete`、`/batch`）限流 **10 次/分钟/用户**；其余接口暂不限流（单用户场景）
- **并发控制**：采用"最后写入胜出"策略，不加乐观锁/悲观锁。前端须通过禁用按钮防止重复提交

### API 接口清单

| 方法 | 路径 | 说明 | 请求体 | 响应 data | 限流 |
|------|------|------|--------|-----------|------|
| POST | `/api/todos` | 创建待办 | CreateTodoRequest | TodoItemVO | — |
| GET | `/api/todos` | 分页查询列表 | Query Params | PageResult\<TodoItemVO\> | — |
| GET | `/api/todos/{id}` | 查询单条待办 | 路径参数 | TodoItemVO | — |
| PUT | `/api/todos/{id}` | 更新待办 | UpdateTodoRequest | TodoItemVO | — |
| DELETE | `/api/todos/{id}` | 删除待办 | 路径参数 | null | — |
| PUT | `/api/todos/{id}/toggle` | 切换完成状态 | 无 | TodoItemVO | — |
| PUT | `/api/todos/batch/complete` | 批量标记完成 | BatchTodoRequest | BatchTodoResponse | 10次/分 |
| DELETE | `/api/todos/batch` | 批量删除 | BatchTodoRequest | BatchTodoResponse | 10次/分 |

---

## Requirements

### Requirement: 创建待办
用户 SHALL 能够创建一条待办事项，标题为必填项，详情、优先级、截止时间为可选项。

#### Scenario: 创建成功 — 仅填标题
- **WHEN** 用户提交 `{"title": "买菜"}`
- **THEN** 返回 HTTP 200，`data` 包含新建待办的完整信息
- **AND** `data.completed` 为 false
- **AND** `data.priority` 为 1（默认中优先级）
- **AND** `data.dueDate` 为 null
- **AND** `data.createdAt` 和 `data.updatedAt` 为当前时间
- **AND** `data.title` 为去首尾空格后的值（如输入 `" 买菜 "` 则存储为 `"买菜"`）

#### Scenario: 创建成功 — 填写全部字段
- **WHEN** 用户提交 `{"title": "写报告", "description": "Q3 季度报告", "priority": 2, "dueDate": "2026-10-20T18:00:00"}`
- **THEN** 返回 HTTP 200，`data` 中各字段与提交值一致
- **AND** `data.title` 为去首尾空格后的值

#### Scenario: 异常 — 标题为空
- **WHEN** 用户提交 `{"title": ""}`
- **THEN** 返回 HTTP 400，`message` 包含"标题不能为空"

#### Scenario: 异常 — 标题超长
- **WHEN** 用户提交 `title` 字段超过 200 字符
- **THEN** 返回 HTTP 400，`message` 包含"标题长度不能超过200个字符"

#### Scenario: 异常 — 标题仅含空格
- **WHEN** 用户提交 `{"title": "   "}`
- **THEN** 返回 HTTP 400，`message` 包含"标题不能为空"（去空格后校验）

#### Scenario: 异常 — 详情超长
- **WHEN** 用户提交 `description` 字段超过 2000 字符
- **THEN** 返回 HTTP 400，`message` 包含"详情长度不能超过2000个字符"

#### Scenario: 异常 — 优先级值非法
- **WHEN** 用户提交 `{"title": "测试", "priority": 5}`
- **THEN** 返回 HTTP 400，`message` 包含"优先级值不合法"

#### Scenario: 异常 — 截止时间为过去时间
- **WHEN** 用户提交 `dueDate` 为早于当前时间的值
- **THEN** 返回 HTTP 400，`message` 包含"截止时间不能早于当前时间"

#### Scenario: 异常 — 截止时间格式错误
- **WHEN** 用户提交 `dueDate` 为非法日期字符串（如 `"abc"` 或 `"2026-13-01"`）
- **THEN** 返回 HTTP 400，`message` 包含"日期格式不正确"

### Requirement: 查询待办列表
用户 SHALL 能够分页查询待办列表，支持按完成状态和优先级筛选，按指定字段排序。

#### Scenario: 查询成功 — 无条件分页
- **WHEN** 用户请求 `GET /api/todos?pageNum=1&pageSize=20`
- **THEN** 返回 HTTP 200，`data.records` 为待办列表数组
- **AND** `data.total` 为总记录数
- **AND** `data.pageNum` 为 1，`data.pageSize` 为 20
- **AND** 默认按 `createdAt` 降序排列

#### Scenario: 查询成功 — 按完成状态筛选
- **WHEN** 用户请求 `GET /api/todos?completed=false`
- **THEN** 返回的 `data.records` 中所有待办 `completed` 均为 false

#### Scenario: 查询成功 — 按优先级筛选
- **WHEN** 用户请求 `GET /api/todos?priority=2`
- **THEN** 返回的 `data.records` 中所有待办 `priority` 均为 2

#### Scenario: 查询成功 — 自定义排序
- **WHEN** 用户请求 `GET /api/todos?sortBy=dueDate&sortOrder=asc`
- **THEN** 返回的 `data.records` 按 `dueDate` 升序排列
- **AND** `dueDate` 为 null 的记录始终排在最后（无论升序/降序）

#### Scenario: 异常 — 页码非法
- **WHEN** 用户请求 `pageNum=0` 或 `pageNum=-1`
- **THEN** 返回 HTTP 400，`message` 包含"页码必须大于0"

#### Scenario: 异常 — 页大小超限
- **WHEN** 用户请求 `pageSize=200`
- **THEN** 返回 HTTP 400，`message` 包含"每页条数不能超过100"

#### Scenario: 异常 — 排序字段非法
- **WHEN** 用户请求 `sortBy=invalidField`
- **THEN** 返回 HTTP 400，`message` 包含"排序字段不合法"

#### Scenario: 边界 — 空列表
- **WHEN** 数据库中无任何待办记录，用户请求列表
- **THEN** 返回 HTTP 200，`data.records` 为空数组 `[]`，`data.total` 为 0

#### Scenario: 边界 — 页码超出总页数
- **WHEN** 用户请求 `pageNum=9999`，实际总页数为 3
- **THEN** 返回 HTTP 200，`data.records` 为空数组 `[]`，`data.total` 为实际总记录数

### Requirement: 查询单条待办
用户 SHALL 能够根据 ID 查询单条待办的详细信息。

#### Scenario: 查询成功
- **WHEN** 用户请求 `GET /api/todos/1`，且 ID=1 的待办存在
- **THEN** 返回 HTTP 200，`data` 为该待办的完整信息

#### Scenario: 异常 — ID 不存在
- **WHEN** 用户请求 `GET /api/todos/99999`，且该 ID 不存在
- **THEN** 返回 HTTP 200，`code` 为 404，`message` 为"待办不存在"，`data` 为 null

#### Scenario: 异常 — ID 格式非法
- **WHEN** 用户请求 `GET /api/todos/abc`
- **THEN** 返回 HTTP 400，`message` 包含"ID 格式不正确"

#### Scenario: 异常 — ID 为负数或零
- **WHEN** 用户请求 `GET /api/todos/0` 或 `GET /api/todos/-1`
- **THEN** 返回 HTTP 400，`message` 包含"ID 必须大于0"

### Requirement: 更新待办
用户 SHALL 能够更新一条已存在的待办，支持部分更新（仅传需要修改的字段）。

#### Scenario: 更新成功 — 修改标题
- **WHEN** 用户提交 `PUT /api/todos/1`，body 为 `{"title": "买菜（更新）"}`
- **THEN** 返回 HTTP 200，`data.title` 为 "买菜（更新）"
- **AND** `data.updatedAt` 晚于原记录的 `updatedAt`
- **AND** 其他未传字段保持不变

#### Scenario: 更新成功 — 同时修改多个字段
- **WHEN** 用户提交 `{"title": "新标题", "priority": 0, "dueDate": "2026-11-01T12:00:00"}`
- **THEN** 返回 HTTP 200，三个字段均已更新

#### Scenario: 异常 — ID 不存在
- **WHEN** 用户提交 `PUT /api/todos/99999`
- **THEN** 返回 HTTP 200，`code` 为 404，`message` 为"待办不存在"

#### Scenario: 异常 — 标题为空
- **WHEN** 用户提交 `{"title": ""}`
- **THEN** 返回 HTTP 400，`message` 包含"标题不能为空"

#### Scenario: 异常 — 请求体全为空
- **WHEN** 用户提交 `{}`
- **THEN** 返回 HTTP 400，`message` 包含"至少需要提供一个更新字段"

#### Scenario: 异常 — 标题超长
- **WHEN** 用户提交 `title` 超过 200 字符
- **THEN** 返回 HTTP 400，`message` 包含"标题长度不能超过200个字符"

#### Scenario: 异常 — 截止时间为过去时间
- **WHEN** 用户提交 `dueDate` 为早于当前时间的值
- **THEN** 返回 HTTP 400，`message` 包含"截止时间不能早于当前时间"

#### Scenario: 异常 — 标题仅含空格
- **WHEN** 用户提交 `{"title": "   "}`
- **THEN** 返回 HTTP 400，`message` 包含"标题不能为空"（去空格后校验）

### Requirement: 删除待办
用户 SHALL 能够删除一条待办。删除操作为物理删除，不可恢复。前端必须在删除操作前进行二次确认。

#### Scenario: 删除成功
- **WHEN** 用户请求 `DELETE /api/todos/1`，且 ID=1 的待办存在
- **THEN** 返回 HTTP 200，`code` 为 200，`data` 为 null
- **AND** 再次查询该 ID 返回"待办不存在"

#### Scenario: 异常 — ID 不存在
- **WHEN** 用户请求 `DELETE /api/todos/99999`
- **THEN** 返回 HTTP 200，`code` 为 404，`message` 为"待办不存在"

#### Scenario: 异常 — ID 格式非法
- **WHEN** 用户请求 `DELETE /api/todos/abc`
- **THEN** 返回 HTTP 400，`message` 包含"ID 格式不正确"

> 删除为物理删除且不可恢复，前端必须在删除前弹出确认对话框。批量删除同理。

### Requirement: 切换完成状态
用户 SHALL 能够通过一次请求切换待办的完成 / 未完成状态。

> **并发说明**：快速连续发送两次 toggle 请求会将状态翻转两次，最终回到原始状态。前端须在 toggle 请求完成前禁用对应操作按钮，防止重复提交。后端不保证幂等性。

#### Scenario: 切换为已完成
- **WHEN** 待办当前 `completed` 为 false，用户请求 `PUT /api/todos/1/toggle`
- **THEN** 返回 HTTP 200，`data.completed` 为 true
- **AND** `data.updatedAt` 已更新

#### Scenario: 切换为未完成
- **WHEN** 待办当前 `completed` 为 true，用户请求 `PUT /api/todos/1/toggle`
- **THEN** 返回 HTTP 200，`data.completed` 为 false

#### Scenario: 异常 — ID 不存在
- **WHEN** 用户请求 `PUT /api/todos/99999/toggle`
- **THEN** 返回 HTTP 200，`code` 为 404，`message` 为"待办不存在"

### Requirement: 批量标记完成
用户 SHALL 能够一次性将多条待办标记为已完成。

#### Scenario: 批量标记成功
- **WHEN** 用户提交 `{"ids": [1, 2, 3]}`，三条待办均存在且未完成
- **THEN** 返回 HTTP 200，三条待办 `completed` 均为 true

#### Scenario: 部分 ID 不存在
- **WHEN** 用户提交 `{"ids": [1, 99999]}`，ID=1 存在，ID=99999 不存在
- **THEN** 返回 HTTP 200，`code` 为 207（部分成功）
- **AND** `message` 包含"1条待办不存在，其余已标记完成"
- **AND** `data` 包含成功处理的 ID 列表 `[1]`

#### Scenario: 异常 — ids 为空数组
- **WHEN** 用户提交 `{"ids": []}`
- **THEN** 返回 HTTP 400，`message` 包含"ID 列表不能为空"

#### Scenario: 异常 — ids 超过 100 条
- **WHEN** 用户提交 `ids` 包含 101 个元素
- **THEN** 返回 HTTP 400，`message` 包含"批量操作不能超过100条"

#### Scenario: 异常 — ids 包含非法值
- **WHEN** 用户提交 `{"ids": [1, 0, -1]}`
- **THEN** 返回 HTTP 400，`message` 包含"ID 必须大于0"

#### Scenario: 异常 — ids 包含重复 ID
- **WHEN** 用户提交 `{"ids": [1, 1, 2]}`
- **THEN** 返回 HTTP 400，`message` 包含"ID 列表包含重复值"

### Requirement: 批量删除
用户 SHALL 能够一次性删除多条待办。

#### Scenario: 批量删除成功
- **WHEN** 用户提交 `{"ids": [1, 2, 3]}`，三条待办均存在
- **THEN** 返回 HTTP 200，三条待办已被物理删除

#### Scenario: 部分 ID 不存在
- **WHEN** 用户提交 `{"ids": [1, 99999]}`，ID=1 存在，ID=99999 不存在
- **THEN** 返回 HTTP 200，`code` 为 207（部分成功）
- **AND** `message` 包含"1条待办不存在，其余已删除"
- **AND** `data` 包含成功处理的 ID 列表 `[1]`

#### Scenario: 异常 — ids 为空数组
- **WHEN** 用户提交 `{"ids": []}`
- **THEN** 返回 HTTP 400，`message` 包含"ID 列表不能为空"

#### Scenario: 异常 — ids 包含重复 ID
- **WHEN** 用户提交 `{"ids": [1, 1, 2]}`
- **THEN** 返回 HTTP 400，`message` 包含"ID 列表包含重复值"

---

## 验收标准

### 后端验收

- [ ] 所有 8 个 API 接口可通过 Swagger UI 查看并调用
- [ ] 创建待办：仅传 title 可成功，返回完整 TodoItemVO
- [ ] 创建待办：title 为空 / 超长 / 纯空格 返回 HTTP 400
- [ ] 创建待办：priority 非法值返回 HTTP 400
- [ ] 创建待办：dueDate 为过去时间返回 HTTP 400
- [ ] 创建待办：dueDate 格式错误返回 HTTP 400
- [ ] 列表查询：无条件分页返回正确分页结构
- [ ] 列表查询：按 completed / priority 筛选结果正确
- [ ] 列表查询：按 dueDate 升序 / createdAt 降序排序结果正确，dueDate 为 null 的记录排在最后
- [ ] 列表查询：pageNum<=0 或 pageSize>100 返回 HTTP 400
- [ ] 列表查询：空数据库返回 records=[]，total=0
- [ ] 单条查询：存在的 ID 返回完整信息
- [ ] 单条查询：不存在的 ID 返回 code=404
- [ ] 单条查询：ID 为负数 / 非数字 返回 HTTP 400
- [ ] 更新待办：部分更新只修改传入字段，其余不变
- [ ] 更新待办：updatedAt 在更新后自动变化
- [ ] 更新待办：空请求体 `{}` 返回 HTTP 400
- [ ] 更新待办：不存在的 ID 返回 code=404
- [ ] 删除待办：删除后再次查询返回 code=404
- [ ] 删除待办：不存在的 ID 返回 code=404
- [ ] 切换状态：false→true 和 true→false 均正确
- [ ] 批量标记：全部存在时全部标记成功
- [ ] 批量标记：部分不存在时返回 code=207 和部分成功信息
- [ ] 批量标记：空 ids 返回 HTTP 400
- [ ] 批量标记：超过 100 条返回 HTTP 400
- [ ] 批量删除：全部存在时全部删除成功
- [ ] 批量删除：部分不存在时返回 code=207 和部分成功信息
- [ ] 批量删除：重复 ID 返回 HTTP 400
- [ ] 所有接口的响应格式符合 `Result<T>` 统一封装
- [ ] 所有接口的字段命名使用驼峰法（camelCase）
- [ ] 数据库表结构与实体字段一一对应

### 前端验收

- [ ] Todo 页面可通过路由 `/todos` 访问
- [ ] 页面展示待办列表，包含标题、优先级标签、完成状态、截止时间
- [ ] 列表支持按完成状态和优先级筛选
- [ ] 列表支持分页，翻页数据正确
- [ ] 创建表单：title 为必填，其他字段可选
- [ ] 创建表单：title 为空时提交按钮禁用或显示校验提示
- [ ] 创建成功后列表自动刷新，新待办出现在列表中
- [ ] 点击待办可切换完成 / 未完成状态，UI 即时反馈
- [ ] 点击编辑可修改待办内容，保存后列表更新
- [ ] 点击删除弹出确认提示，确认后删除并刷新列表
- [ ] 批量选择 + 批量删除 / 批量标记完成功能可用
- [ ] 网络超时（>10s）展示"网络请求超时，请重试"
- [ ] 服务端 5xx 错误展示"服务暂时不可用，请稍后重试"
- [ ] 业务校验失败（code!=200）展示后端返回的 message 内容
- [ ] 空列表展示空状态提示（如"暂无待办事项"）
- [ ] 页面加载中有 loading 状态指示
- [ ] 所有操作按钮在请求进行中禁用，防止重复点击

### 性能验收

- [ ] 列表接口（100 条数据）响应时间 < 500ms
- [ ] 创建 / 更新 / 删除接口响应时间 < 200ms
- [ ] 批量操作（100 条）响应时间 < 2s

### 前后端联调验收

- [ ] 前端创建 → 后端数据库可查到记录
- [ ] 后端数据库手动插入记录 → 前端列表可展示
- [ ] 前端切换状态 → 后端 completed 字段变化
- [ ] 前端删除 → 后端记录被物理删除
- [ ] 前端分页参数正确传递到后端
- [ ] 前端筛选参数正确传递到后端
