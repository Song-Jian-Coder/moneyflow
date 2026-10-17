# Tasks

## 1. Maven 依赖替换

- [x] 1.1 在 `backend/pom.xml`（parent）的 `<dependencyManagement>` 中移除 `mybatis-plus-spring-boot3-starter` 版本声明，新增 `spring-boot-starter-data-jpa` 版本声明。验证：pom.xml 格式正确
- [x] 1.2 在 `backend/moneyflow-dao/pom.xml` 中将 `mybatis-plus-spring-boot3-starter` 依赖替换为 `spring-boot-starter-data-jpa`。验证：pom.xml 格式正确
- [x] 1.3 执行 `mvn clean compile -pl moneyflow-dao`，验证 Spring Data JPA 依赖正确引入、编译通过。验证：BUILD SUCCESS

## 2. Entity 注解迁移（moneyflow-model）

- [x] 2.1 修改 `TodoItem.java`：移除 `@TableName`、`@TableId`、`@TableField` 注解，替换为 `@Entity`、`@Table(name = "todo_item", indexes = {...})`、`@Id`、`@GeneratedValue(strategy = IDENTITY)`、`@Column` 注解。字段约束（nullable、length）通过 `@Column` 声明。验证：编译通过
- [x] 2.2 确认 `createdAt` 字段添加 `@Column(nullable = false, updatable = false)`，`updatedAt` 字段添加 `@Column(nullable = false)`，`dueDate` 字段添加 `@Column(name = "due_date")`。验证：编译通过

## 3. DAO 层迁移（moneyflow-dao）

- [x] 3.1 删除 `TodoItemMapper.java`（`com.moneyflow.dao.mapper` 包），创建 `TodoItemRepository.java`（`com.moneyflow.dao.repository` 包），继承 `JpaRepository<TodoItem, Long>` 和 `JpaSpecificationExecutor<TodoItem>`。验证：编译通过
- [x] 3.2 修改启动类：移除 `@MapperScan("com.moneyflow.dao.mapper")`，添加 `@EnableJpaRepositories(basePackages = "com.moneyflow.dao.repository")`。验证：编译通过

## 4. 配置类迁移（moneyflow-web）

- [x] 4.1 删除 `MybatisPlusConfig.java`（分页插件配置类）。验证：编译通过
- [x] 4.2 修改 `application-dev.yml`：添加 `spring.jpa.hibernate.ddl-auto: none`、`spring.jpa.show-sql: true`，保留 `spring.sql.init` 配置。验证：YAML 格式正确
- [x] 4.3 确认 `schema.sql` 中的 `todo_item` 建表 DDL 与 JPA Entity 字段定义一致（字段名、类型、约束、索引）。验证：SQL 与 Entity 对齐

## 5. Service 层迁移（moneyflow-service）

- [x] 5.1 修改 `TodoServiceImpl.java`：将 `TodoItemMapper` 注入替换为 `TodoItemRepository` 注入。验证：编译通过
- [x] 5.2 迁移 `create` 方法：`mapper.insert(entity)` → `repository.save(entity)`。验证：编译通过
- [x] 5.3 迁移 `getById` 方法：`mapper.selectById(id)` → `repository.findById(id).orElseThrow(() -> new BusinessException(404, "待办不存在"))`。验证：编译通过
- [x] 5.4 迁移 `update` 方法：`mapper.updateById(entity)` → `repository.save(entity)`，保持存在性校验和字段合并逻辑不变。验证：编译通过
- [x] 5.5 迁移 `delete` 方法：`mapper.deleteById(id)` → `repository.deleteById(id)`，保持存在性校验不变。验证：编译通过
- [x] 5.6 迁移 `toggle` 方法：保持状态翻转逻辑，`save` 替代 `updateById`。验证：编译通过
- [x] 5.7 迁移 `list` 方法：将 `LambdaQueryWrapper` + `selectPage` 替换为 `Specification<TodoItem>` + `PageRequest`。构建动态条件（completed/priority 筛选），处理排序（sortBy/sortOrder），注意分页 `pageNum - 1` 转换。验证：编译通过
- [x] 5.8 修改 `PageResult.from()` 静态方法：将 MyBatis-Plus `Page<T>` 参数替换为 Spring Data `Page<T>` 参数，使用 `getContent()`、`getTotalElements()`、`getNumber() + 1`、`getSize()`、`getTotalPages()`。验证：编译通过
- [x] 5.9 迁移 `batchComplete` 方法：`selectById` → `findById`，`updateById` → `save`。验证：编译通过
- [x] 5.10 迁移 `batchDelete` 方法：`selectById` → `findById`，`deleteById` → `deleteById`。验证：编译通过

## 6. 构建与启动验证

- [x] 6.1 在 backend/ 根目录执行 `mvn clean install -DskipTests`，验证所有模块编译打包成功。验证：BUILD SUCCESS
- [x] 6.2 以 dev profile 启动应用，验证 `todo_item` 表通过 schema.sql 自动创建，JPA 正常初始化。验证：启动日志无报错
- [x] 6.3 确认无 MyBatis-Plus 相关启动日志（如 MybatisPlusInterceptor），确认 Hibernate 初始化日志正常。验证：日志中有 `HHH000400: Using dialect` 或类似 Hibernate 初始化信息

## 7. API 功能验证

- [x] 7.1 使用 curl/Postman 测试 POST `/api/todos` 创建待办，验证返回 `Result<TodoItemVO>` 格式正确。验证：返回 code=200
- [x] 7.2 测试 GET `/api/todos` 列表查询，验证分页结构（records/total/pageNum/pageSize/totalPages）正确。验证：分页结构与迁移前一致
- [x] 7.3 测试 GET `/api/todos` 筛选和排序参数（completed、priority、sortBy、sortOrder），验证结果正确。验证：筛选排序行为与迁移前一致
- [x] 7.4 测试 GET `/api/todos/{id}` 单条查询，验证存在和不存在场景。验证：存在返回 200，不存在返回 code=404
- [x] 7.5 测试 PUT `/api/todos/{id}` 更新，验证部分更新和全空请求体场景。验证：更新成功，全空返回 400
- [x] 7.6 测试 DELETE `/api/todos/{id}` 删除，验证存在和不存在场景。验证：删除成功，不存在返回 code=404
- [x] 7.7 测试 PUT `/api/todos/{id}/toggle` 切换状态，验证 false→true 和 true→false。验证：状态正确翻转
- [x] 7.8 测试 PUT `/api/todos/batch/complete` 批量标记完成，验证全部成功和部分成功场景。验证：批量操作结果正确
- [x] 7.9 测试 DELETE `/api/todos/batch` 批量删除，验证全部成功和部分成功场景。验证：批量删除结果正确
- [x] 7.10 测试参数校验：title 为空返回 HTTP 400，priority=5 返回 HTTP 400。验证：校验异常格式与迁移前一致
- [x] 7.11 访问 `/swagger-ui.html` 验证 Todo API 文档仍正常生成，8 个端点均可见。验证：Swagger UI 可访问

## Workflow follow-up

- 归档变更：实现完成并通过审查后执行 `openspec archive`
- 同步规范：更新 `openspec/specs/backend-framework/spec.md` 中数据访问层相关描述
- 更新 config.yaml：将技术栈信息更新到 `openspec/config.yaml` 的 context 字段
