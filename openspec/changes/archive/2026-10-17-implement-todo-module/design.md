# Design

## Context

框架基座已就绪（详见 proposal.md），后端提供 Result\<T\> 统一响应、BusinessException 业务异常、GlobalExceptionHandler 全局异常处理、MyBatis-Plus ORM、SpringDoc API 文档；前端提供 Axios 封装（含 Result 解析拦截器）、React Router 路由。Todo 模块是第一个业务模块，需遵循框架约定进行开发。

约束：
- 单用户，无需权限隔离
- dev 环境使用 H2 内存数据库，prod 使用 MySQL 8.0
- 字段命名前后端统一驼峰法

## Goals / Non-Goals

**Goals:**
- 实现 Todo Spec 中定义的 8 个 REST API 和 36 个场景
- 后端代码按分层架构落入对应模块（model/dao/service/web）
- 前端实现 Todo 页面，包含列表、创建、编辑、删除、状态切换、批量操作、筛选、分页
- 前后端联调通过 Vite proxy 验证

**Non-Goals:**
- 不实现认证鉴权（后续变更）
- 不实现 Spec 中明确排除的功能（协作、提醒、子任务、标签等）
- 不编写单元测试（后续引入测试框架后补充）
- 不做前端国际化

## Decisions

### 决策 1：数据库表设计

```sql
CREATE TABLE todo_item (
    id          BIGINT AUTO_INCREMENT PRIMARY KEY,
    title       VARCHAR(200)  NOT NULL,
    description VARCHAR(2000) NULL,
    completed   TINYINT(1)    NOT NULL DEFAULT 0,
    priority    TINYINT       NOT NULL DEFAULT 1,
    due_date    DATETIME      NULL,
    created_at  DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_completed (completed),
    INDEX idx_priority (priority),
    INDEX idx_due_date (due_date)
);
```

**理由**：
- `completed`、`priority`、`due_date` 是 Spec 中定义的筛选/排序字段，加索引加速查询
- `description` 用 VARCHAR(2000) 而非 TEXT，因为 MyBatis-Plus 对 VARCHAR 有更好的类型映射
- `priority` 用 TINYINT 存储 0/1/2，节省空间
- H2 dev 环境通过 `schema.sql` 自动建表，MySQL prod 通过同一 DDL 脚本手动执行

**备选方案**：
- 不加索引：数据量小时可接受，但 Spec 明确要求筛选排序，提前加索引避免后续遗漏
- 使用 TEXT 类型：H2 对 TEXT 的默认值支持不如 VARCHAR 一致

### 决策 2：后端分层代码结构

```
moneyflow-model/
└── src/main/java/com/moneyflow/model/
    ├── entity/
    │   └── TodoItem.java              (@TableName("todo_item") MyBatis-Plus 实体)
    ├── dto/
    │   ├── CreateTodoRequest.java     (JSR 380 校验注解)
    │   ├── UpdateTodoRequest.java     (所有字段可选)
    │   ├── TodoQueryRequest.java      (筛选+分页参数)
    │   └── BatchTodoRequest.java      (ids 列表)
    └── vo/
        ├── TodoItemVO.java            (响应 VO)
        └── PageResult.java            (通用分页响应，泛型)

moneyflow-dao/
└── src/main/java/com/moneyflow/dao/
    └── mapper/
        └── TodoItemMapper.java        (extends BaseMapper<TodoItem>)

moneyflow-service/
└── src/main/java/com/moneyflow/service/
    ├── TodoService.java               (接口)
    └── impl/
        └── TodoServiceImpl.java       (实现)

moneyflow-web/
└── src/main/java/com/moneyflow/web/
    └── controller/
        └── TodoController.java        (8 个 REST 端点)
```

**理由**：严格遵循框架阶段确立的分层约束。Entity 在 model 层，Mapper 在 dao 层，Service 在 service 层，Controller 在 web 层。DTO/VO 放在 model 层因为它们是数据模型的一部分。

**备选方案**：
- DTO 放在 web 层：违反分层原则，service 层也需要用到 DTO
- 每个功能一个包（todo/entity, todo/dto...）：当前只有一个业务模块，按层分包更简洁；后续模块增多时可重构为按领域分包

### 决策 3：分页方案

使用 MyBatis-Plus 内置的 `Page<T>` 分页插件。

```java
// TodoQueryRequest → Page<TodoItem>
Page<TodoItem> page = new Page<>(query.getPageNum(), query.getPageSize());
LambdaQueryWrapper<TodoItem> wrapper = new LambdaQueryWrapper<>();
// 动态条件拼接...
Page<TodoItem> result = todoItemMapper.selectPage(page, wrapper);
// 转换为 PageResult<TodoItemVO>
```

**理由**：MyBatis-Plus 分页插件已随 3.5.7 版本引入，无需额外依赖。自动处理 COUNT 查询和分页 SQL 改写。

### 决策 4：异常处理策略

复用框架已有的 `BusinessException` + `GlobalExceptionHandler` 机制：

```java
// Service 层抛出业务异常
if (todoItem == null) {
    throw new BusinessException(404, "待办不存在");
}

// Controller 层参数校验由 JSR 380 注解自动处理
@PostMapping
public Result<TodoItemVO> create(@Valid @RequestBody CreateTodoRequest request) { ... }
```

错误码约定（与 Spec 对齐）：
- `200` = 成功
- `207` = 批量操作部分成功
- `400` = 参数校验失败（由 GlobalExceptionHandler 处理，HTTP 400）
- `404` = 待办不存在（由 BusinessException 抛出，HTTP 200）
- `500` = 系统内部错误（由 GlobalExceptionHandler 处理，HTTP 500）

**理由**：不引入新的异常类，保持框架一致性。BusinessException 的 code 字段天然支持自定义错误码。

### 决策 5：前端页面结构

```
frontend/src/
├── types/
│   └── todo.ts              (TodoItem, CreateTodoRequest, TodoQueryRequest 等类型)
├── api/
│   └── todo.ts              (8 个 API 调用函数)
├── pages/
│   └── Todo.tsx             (Todo 页面主组件)
├── components/
│   └── todo/
│       ├── TodoList.tsx      (列表组件：筛选 + 表格 + 分页)
│       ├── TodoItem.tsx      (单条待办组件：展示 + 切换 + 编辑 + 删除)
│       ├── TodoForm.tsx      (创建/编辑表单弹窗)
│       └── TodoFilter.tsx    (筛选栏组件)
└── App.tsx                   (新增 /todos 路由)
```

**理由**：
- 页面拆分为 List/Item/Form/Filter 四个组件，职责清晰，便于后续维护
- API 层独立文件封装，与类型定义分离
- 使用 `components/todo/` 子目录避免组件目录膨胀

**备选方案**：
- 单文件 Todo.tsx：功能太多（列表+表单+筛选+批量），单文件超 500 行不可维护
- 使用 UI 组件库（Ant Design / MUI）：增加依赖体积，当前阶段用原生 HTML+CSS 足够

### 决策 6：前端状态管理

使用 React `useState` + `useEffect` 管理页面状态，不引入 Redux/Zustand。

**理由**：Todo 页面状态简单（列表数据 + 筛选条件 + 表单状态），组件间通过 props 传递即可。引入状态管理库增加复杂度但收益不大。后续如状态复杂化可平滑迁移到 Zustand。

## Risks / Trade-offs

- **[风险] H2 与 MySQL 的 SQL 差异**：H2 的 `ON UPDATE CURRENT_TIMESTAMP` 行为与 MySQL 不完全一致。→ 缓解：dev 环境在 Service 层手动设置 `updatedAt`，不依赖数据库自动更新。
- **[风险] 批量操作的事务一致性**：批量删除/标记时部分 ID 不存在，需要部分成功语义（code=207）。→ 缓解：逐条处理并收集成功/失败 ID，不使用事务回滚全部。
- **[权衡] 不引入 UI 组件库**：原生 HTML+CSS 开发效率较低，但避免了大体积依赖。→ 对于第一个业务模块可接受，后续可引入。
- **[权衡] 不编写单元测试**：Spec 验收依赖手动测试。→ 后续引入 JUnit 5 + MockMvc 后补充。
