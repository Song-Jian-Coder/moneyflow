# Tasks

## 1. 数据库与 MyBatis-Plus 配置

- [x] 1.1 在 `backend/moneyflow-web/src/main/resources/` 下创建 `schema.sql`，包含 `todo_item` 建表 DDL（含索引）。验证：文件存在且 SQL 语法正确
- [x] 1.2 在 `backend/moneyflow-web/src/main/resources/application-dev.yml` 中添加 `spring.sql.init.mode: always` 和 `spring.sql.init.schema-locations: classpath:schema.sql`。验证：YAML 格式正确
- [x] 1.3 在 `backend/moneyflow-web/` 中创建 MyBatis-Plus 分页插件配置类 `MybatisPlusConfig.java`（`com.moneyflow.web.config.MybatisPlusConfig`），注册 `MybatisPlusInterceptor` + `PaginationInnerInterceptor`。验证：编译通过

## 2. 后端 Model 层 — 实体与 DTO/VO

- [x] 2.1 在 `moneyflow-model` 中创建 `TodoItem.java`（`com.moneyflow.model.entity.TodoItem`），使用 `@TableName("todo_item")`、`@TableId(type = IdType.AUTO)`，字段与 spec 对齐，使用 Lombok `@Data`。验证：编译通过
- [x] 2.2 在 `moneyflow-model` 中创建 `CreateTodoRequest.java`（`com.moneyflow.model.dto.CreateTodoRequest`），添加 JSR 380 注解：`@NotBlank` + `@Size(max=200)` on title、`@Size(max=2000)` on description、`@Min(0) @Max(2)` on priority、`@Future` on dueDate。验证：编译通过
- [x] 2.3 在 `moneyflow-model` 中创建 `UpdateTodoRequest.java`（`com.moneyflow.model.dto.UpdateTodoRequest`），所有字段可选，约束同 2.2 但无 `@NotBlank`。验证：编译通过
- [x] 2.4 在 `moneyflow-model` 中创建 `TodoQueryRequest.java`（`com.moneyflow.model.dto.TodoQueryRequest`），包含 completed/priority/sortBy/sortOrder/pageNum/pageSize 字段及默认值。验证：编译通过
- [x] 2.5 在 `moneyflow-model` 中创建 `BatchTodoRequest.java`（`com.moneyflow.model.dto.BatchTodoRequest`），包含 `@NotEmpty @Size(max=100) List<@Positive Long> ids`。验证：编译通过
- [x] 2.6 在 `moneyflow-model` 中创建 `TodoItemVO.java`（`com.moneyflow.model.vo.TodoItemVO`），包含 8 个字段与 TodoItem 对齐。验证：编译通过
- [x] 2.7 在 `moneyflow-model` 中创建 `PageResult.java`（`com.moneyflow.model.vo.PageResult<T>`），包含 records/total/pageNum/pageSize/totalPages 字段，提供 `from(Page<T>, Function)` 静态转换方法。验证：编译通过

## 3. 后端 DAO 层

- [x] 3.1 在 `moneyflow-dao` 中创建 `TodoItemMapper.java`（`com.moneyflow.dao.mapper.TodoItemMapper`），继承 `BaseMapper<TodoItem>`，添加 `@Mapper` 注解。验证：编译通过
- [x] 3.2 在 `moneyflow-web` 启动类上添加 `@MapperScan("com.moneyflow.dao.mapper")`。验证：编译通过

## 4. 后端 Service 层

- [x] 4.1 在 `moneyflow-service` 中创建 `TodoService.java` 接口（`com.moneyflow.service.TodoService`），声明 8 个方法：create/getById/update/delete/toggle/list/batchComplete/batchDelete。验证：编译通过
- [x] 4.2 在 `moneyflow-service` 中创建 `TodoServiceImpl.java`（`com.moneyflow.service.impl.TodoServiceImpl`），实现 create 方法：构建 TodoItem、设置默认值、调用 mapper.insert、返回 TodoItemVO。验证：编译通过
- [x] 4.3 实现 getById 方法：mapper.selectById，不存在抛 `BusinessException(404, "待办不存在")`。验证：编译通过
- [x] 4.4 实现 update 方法：先 getById 校验存在，合并非空字段，手动设置 updatedAt，调用 mapper.updateById。验证：编译通过
- [x] 4.5 实现 delete 方法：先 getById 校验存在，调用 mapper.deleteById。验证：编译通过
- [x] 4.6 实现 toggle 方法：先 getById，翻转 completed，手动设置 updatedAt，调用 mapper.updateById。验证：编译通过
- [x] 4.7 实现 list 方法：构建 LambdaQueryWrapper 动态条件（completed/priority），处理排序（sortBy/sortOrder），调用 mapper.selectPage，转换为 PageResult。验证：编译通过
- [x] 4.8 实现 batchComplete 方法：逐条 selectById，存在的标记 completed=true 并 updateById，收集成功/失败 ID，返回部分成功结果。验证：编译通过
- [x] 4.9 实现 batchDelete 方法：逐条 selectById，存在的调用 deleteById，收集成功/失败 ID。验证：编译通过

## 5. 后端 Controller 层

- [x] 5.1 在 `moneyflow-web` 中创建 `TodoController.java`（`com.moneyflow.web.controller.TodoController`），添加 `@RestController` + `@RequestMapping("/api/todos")`。实现 POST `/` 创建待办，使用 `@Valid @RequestBody`。验证：编译通过
- [x] 5.2 实现 GET `/` 列表查询，参数为 TodoQueryRequest。验证：编译通过
- [x] 5.3 实现 GET `/{id}` 单条查询，`@PathVariable Long id`，id<=0 时抛 BusinessException。验证：编译通过
- [x] 5.4 实现 PUT `/{id}` 更新，使用 `@Valid @RequestBody UpdateTodoRequest`，请求体全空时抛异常。验证：编译通过
- [x] 5.5 实现 DELETE `/{id}` 删除。验证：编译通过
- [x] 5.6 实现 PUT `/{id}/toggle` 切换状态。验证：编译通过
- [x] 5.7 实现 PUT `/batch/complete` 批量标记完成。验证：编译通过
- [x] 5.8 实现 DELETE `/batch` 批量删除。验证：编译通过

## 6. 后端构建与启动验证

- [x] 6.1 在 backend/ 根目录执行 `mvn clean install -DskipTests`，验证所有模块编译打包成功。验证：BUILD SUCCESS
- [x] 6.2 以 dev profile 启动应用，验证 `todo_item` 表自动创建。验证：启动日志无报错，H2 控制台可查表
- [x] 6.3 使用 curl/Postman 测试 POST `/api/todos` 创建待办，验证返回 `Result<TodoItemVO>` 格式正确。验证：返回 code=200
- [x] 6.4 测试 GET `/api/todos` 列表查询，验证分页结构正确。验证：返回 records/total/pageNum 等字段
- [x] 6.5 测试参数校验：title 为空返回 HTTP 400，priority=5 返回 HTTP 400。验证：校验异常格式正确
- [x] 6.6 测试 ID 不存在场景：GET `/api/todos/99999` 返回 code=404。验证：返回"待办不存在"
- [x] 6.7 访问 `/swagger-ui.html` 验证 Todo API 文档已自动生成。验证：8 个端点均可见

## 7. 前端类型定义与 API 层

- [x] 7.1 创建 `frontend/src/types/todo.ts`，定义 TodoItem、CreateTodoRequest、UpdateTodoRequest、TodoQueryRequest、BatchTodoRequest、PageResult 类型接口。验证：TypeScript 编译无错误
- [x] 7.2 创建 `frontend/src/api/todo.ts`，封装 8 个 API 函数（createTodo/getTodo/updateTodo/deleteTodo/toggleTodo/listTodos/batchComplete/batchDelete），使用 request 实例。验证：TypeScript 编译无错误

## 8. 前端组件开发

- [x] 8.1 创建 `frontend/src/components/todo/TodoFilter.tsx`，包含完成状态筛选（全部/未完成/已完成）、优先级筛选、排序选择。验证：组件渲染正常，筛选条件变化时触发回调
- [x] 8.2 创建 `frontend/src/components/todo/TodoForm.tsx`，创建/编辑表单弹窗，包含 title（必填）、description、priority（下拉选择）、dueDate（日期选择）。验证：title 为空时提交按钮禁用，校验提示可见
- [x] 8.3 创建 `frontend/src/components/todo/TodoItem.tsx`，单条待办展示组件，包含复选框切换、标题、优先级标签、截止时间、编辑/删除按钮。验证：点击复选框触发 toggle 回调
- [x] 8.4 创建 `frontend/src/components/todo/TodoList.tsx`，列表组件，集成 TodoFilter + TodoItem 列表 + 分页控件 + 批量操作栏。验证：列表渲染正常，分页翻页正确

## 9. 前端页面与路由

- [x] 9.1 创建 `frontend/src/pages/Todo.tsx` 页面主组件，管理状态（列表数据、筛选条件、loading、表单弹窗），调用 API 函数，集成 TodoList + TodoForm。验证：页面渲染正常
- [x] 9.2 修改 `frontend/src/App.tsx`，新增 `/todos` 路由指向 Todo 页面。验证：访问 `http://localhost:5173/todos` 展示 Todo 页面
- [x] 9.3 修改 `frontend/src/pages/Home.tsx`，添加"进入待办清单"链接指向 `/todos`。验证：首页可点击跳转

## 10. 前后端联调验证

- [x] 10.1 同时启动后端（dev）和前端（dev），在 Todo 页面创建一条待办，验证列表刷新显示新待办。验证：创建→列表可见
- [x] 10.2 测试切换完成状态，验证 UI 即时反馈（复选框勾选/取消）。验证：状态变化正确
- [x] 10.3 测试编辑待办，验证修改后列表更新。验证：编辑→列表刷新
- [x] 10.4 测试删除待办（含确认弹窗），验证删除后列表刷新。验证：删除→列表刷新
- [x] 10.5 测试筛选和分页功能，验证前后端参数传递正确。验证：筛选结果与预期一致
- [x] 10.6 测试批量选择和批量操作，验证批量删除/标记完成功能正常。验证：批量操作结果正确
- [x] 10.7 测试异常场景：title 为空提交被前端拦截、网络断开时显示友好错误提示。验证：异常处理符合 Spec

## Workflow follow-up

- 归档变更：实现完成并通过审查后执行 `openspec archive`
- 同步规范：本次 skip_specs，无需同步
- 更新 config.yaml：将技术栈信息更新到 `openspec/config.yaml` 的 context 字段
