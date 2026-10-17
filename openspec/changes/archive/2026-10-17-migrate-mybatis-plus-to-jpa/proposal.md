# Proposal

## Why

当前后端数据访问层使用 MyBatis-Plus，其优势集中在单表 CRUD 场景，但在多表关联查询时需要退化到原生 MyBatis 手写 SQL，缺乏对象关系映射、级联查询、Fetch 策略等能力。随着后续业务模块（如账单-分类-标签-用户等关联密集的场景）的引入，MyBatis-Plus 在多表查询场景的实用性不足。Spring Data JPA 提供声明式 Repository、方法名派生查询、JPQL/Criteria API、对象关系映射（@OneToMany/@ManyToOne）、JOIN FETCH 等能力，更适合多表关联查询密集的业务场景。在 Todo 模块刚完成、尚未引入更多业务模块的窗口期进行切换，改造成本最低。

## What Changes

- **BREAKING** 后端数据访问层从 MyBatis-Plus 迁移到 Spring Data JPA（Hibernate）
- 移除 MyBatis-Plus 相关依赖（`mybatis-plus-spring-boot3-starter`），引入 Spring Data JPA 依赖（`spring-boot-starter-data-jpa`）
- Entity 注解从 MyBatis-Plus 风格（`@TableName`、`@TableId`、`@TableField`）替换为 JPA 风格（`@Entity`、`@Table`、`@Id`、`@GeneratedValue`、`@Column`）
- DAO 层从 `BaseMapper<T>` + `@Mapper` 替换为 `JpaRepository<T, ID>` + `@Repository`
- Service 层查询方式从 `LambdaQueryWrapper` 替换为 Spring Data `Specification` / 方法名派生查询 / `@Query`
- 分页从 MyBatis-Plus `PaginationInnerInterceptor` + `Page<T>` 替换为 Spring Data `Pageable` + `Page<T>`
- 配置类从 `MybatisPlusConfig`（分页插件）替换为 JPA 相关配置（如需要）
- `schema.sql` 建表方式可选保留或切换为 JPA `ddl-auto`（由 JPA 自动建表）
- 前端不受影响——所有 REST API 接口路径、请求/响应格式保持不变
- DTO、VO、统一响应封装 `Result<T>`、全局异常处理、参数校验等不受影响

## Capabilities

### New Capabilities

（无新增能力）

### Modified Capabilities

- `backend-framework`: 数据访问层技术栈从 MyBatis-Plus 变更为 Spring Data JPA，涉及 DAO 层模块依赖、Entity 注解体系、分页机制、配置类的变更

## Impact

- **代码**：后端 `moneyflow-dao`、`moneyflow-model`（Entity 注解）、`moneyflow-service`（查询方式）、`moneyflow-web`（配置类、启动类注解）四个模块需要修改
- **依赖**：移除 `mybatis-plus-spring-boot3-starter`，新增 `spring-boot-starter-data-jpa`、`hibernate-core`（由 starter 传递依赖）
- **API**：无变化——所有 REST API 路径、请求/响应格式完全不变，前端零改动
- **数据库**：表结构不变，数据不变；可选从 `schema.sql` 手动建表切换为 JPA `ddl-auto=update` 自动建表
- **构建**：Maven 构建流程不变
- **回滚方案**：本次变更集中在后端数据访问层，可通过 Git 回退后端子仓库到变更前状态。建议在独立分支上实施，验证通过后再合并
