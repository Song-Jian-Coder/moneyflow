# Design

## Context

后端当前采用 Maven 多模块结构（parent → common → model → dao → service → web），数据访问层基于 MyBatis-Plus 3.5.x。Todo 模块已完整实现（52 个任务全部完成），包含 Entity、DTO、VO、Mapper、Service、Controller 全套代码。本次变更仅替换数据访问层的 ORM 框架，不改变任何 API 行为。约束：保持 Maven 多模块结构不变，保持前后端接口不变。

## Goals / Non-Goals

**Goals:**
- 将数据访问层从 MyBatis-Plus 完整迁移到 Spring Data JPA（Hibernate）
- Entity 使用 JPA 注解体系，DAO 层使用 `JpaRepository` 接口
- Service 层使用 Spring Data 查询机制（方法名派生查询、`Specification`、`@Query`）
- 分页使用 Spring Data `Pageable` + `Page<T>`
- 迁移后所有 8 个 Todo API 行为与迁移前完全一致
- `PageResult<T>` 统一分页响应结构保持不变

**Non-Goals:**
- 不改变任何 REST API 路径、请求/响应格式
- 不改变前端代码
- 不改变 Maven 多模块结构（common/model/dao/service/web 分层不变）
- 不改变统一响应封装 `Result<T>`、全局异常处理、参数校验机制
- 不引入新的业务功能
- 不改变数据库类型（dev 环境 H2，prod 环境 MySQL 8.0）

## Decisions

### 决策 1：依赖替换方案

**移除：**
- `mybatis-plus-spring-boot3-starter`（dao 模块）

**新增：**
- `spring-boot-starter-data-jpa`（dao 模块）— 自动引入 `hibernate-core`、`spring-data-jpa`、`jakarta.persistence-api`

**保留不变：**
- `spring-boot-starter-web`、`spring-boot-starter-validation`
- `springdoc-openapi-starter-webmvc-ui`
- `lombok`
- `h2`（dev）、`mysql-connector-j`（prod）

**理由：** `spring-boot-starter-data-jpa` 是 Spring Boot 官方 JPA starter，自动管理 Hibernate 版本兼容性，无需手动引入 `hibernate-core`。

**备选方案：**
- 手动引入 `hibernate-core` + `spring-orm`：过于底层，缺少 Repository 抽象
- 使用 `spring-data-jpa` 但不引入 starter：需要手动管理更多依赖版本

### 决策 2：Entity 注解映射

```
MyBatis-Plus                          Spring Data JPA
─────────────────                     ─────────────────
@TableName("todo_item")          →    @Entity @Table(name = "todo_item")
@TableId(type = IdType.AUTO)     →    @Id @GeneratedValue(strategy = IDENTITY)
@TableField("title")             →    @Column(name = "title") 或省略（字段名匹配列名时）
无                                 →    @Column(nullable = false, length = 200)
无                                 →    @Column(precision = 3) 用于毫秒精度时间
```

**具体映射（TodoItem 实体）：**

```java
@Entity
@Table(name = "todo_item", indexes = {
    @Index(name = "idx_completed", columnList = "completed"),
    @Index(name = "idx_priority", columnList = "priority"),
    @Index(name = "idx_created_at", columnList = "created_at"),
    @Index(name = "idx_due_date", columnList = "due_date")
})
public class TodoItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(length = 2000)
    private String description;

    @Column(nullable = false)
    private Boolean completed = false;

    @Column(nullable = false)
    private Integer priority = 1;

    @Column(name = "due_date")
    private LocalDateTime dueDate;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;
}
```

**理由：** 使用 `@Column` 显式声明约束（nullable、length），让 JPA 在 `ddl-auto=update` 模式下自动生成正确的表结构，替代手动 `schema.sql`。索引通过 `@Table(indexes = ...)` 声明，同样由 JPA 自动创建。

### 决策 3：DAO 层 — Repository 模式

```
当前（MyBatis-Plus）                    迁移后（Spring Data JPA）
─────────────────────                   ─────────────────────────
TodoItemMapper extends                TodoItemRepository extends
    BaseMapper<TodoItem>                  JpaRepository<TodoItem, Long>
@Mapper 注解                           @Repository 注解（或省略，Spring Data 自动识别）
```

**关键接口定义：**

```java
public interface TodoItemRepository extends JpaRepository<TodoItem, Long>,
                                            JpaSpecificationExecutor<TodoItem> {
    // 方法名派生查询 — 简单场景
    List<TodoItem> findByCompleted(Boolean completed);
    List<TodoItem> findByPriority(Integer priority);

    // 复杂动态查询通过 JpaSpecificationExecutor<TodoItem> 的 findAll(Specification, Pageable)
}
```

**理由：** `JpaRepository` 提供标准 CRUD 方法（`save`、`findById`、`deleteById`、`findAll`），零代码实现单表操作。`JpaSpecificationExecutor` 提供 `Specification` 接口支持动态条件组合（替代 `LambdaQueryWrapper`），用于 Todo 列表的筛选+排序+分页组合查询。

**备选方案：**
- 全部用 `@Query` 手写 JPQL：对于简单查询过于冗余，方法名派生更简洁
- 使用 QueryDSL：引入额外注解处理器和 Q 类生成，对当前项目过度

### 决策 4：Service 层查询迁移

| 当前 MyBatis-Plus 写法 | 迁移后 Spring Data JPA 写法 |
|---|---|
| `mapper.insert(entity)` | `repository.save(entity)` |
| `mapper.selectById(id)` | `repository.findById(id).orElseThrow(...)` |
| `mapper.updateById(entity)` | `repository.save(entity)`（JPA 自动合并） |
| `mapper.deleteById(id)` | `repository.deleteById(id)` |
| `LambdaQueryWrapper` + `selectPage` | `Specification<TodoItem>` + `PageRequest` |
| `Page<TodoItem>` (MyBatis-Plus) | `Page<TodoItem>` (Spring Data) |

**动态查询示例（替代 LambdaQueryWrapper）：**

```java
// Service 层构建 Specification
Specification<TodoItem> spec = (root, query, cb) -> {
    List<Predicate> predicates = new ArrayList<>();
    if (request.getCompleted() != null) {
        predicates.add(cb.equal(root.get("completed"), request.getCompleted()));
    }
    if (request.getPriority() != null) {
        predicates.add(cb.equal(root.get("priority"), request.getPriority()));
    }
    // 排序
    if (request.getSortBy() != null) {
        Order order = "asc".equalsIgnoreCase(request.getSortOrder())
            ? cb.asc(root.get(request.getSortBy()))
            : cb.desc(root.get(request.getSortBy()));
        query.orderBy(order);
    }
    return cb.and(predicates.toArray(new Predicate[0]));
};

Page<TodoItem> page = repository.findAll(spec, pageRequest);
```

**理由：** `Specification` 是 Spring Data JPA 标准的动态查询方式，类型安全且支持条件组合，功能等价于 MyBatis-Plus 的 `LambdaQueryWrapper`。

### 决策 5：分页迁移

```
当前（MyBatis-Plus）                    迁移后（Spring Data JPA）
─────────────────────                   ─────────────────────────
MybatisPlusConfig 中注册              无需手动注册分页插件
PaginationInnerInterceptor            Spring Data 自动处理
Page<TodoItem> page =                PageRequest pageRequest =
  new Page<>(pageNum, pageSize);        PageRequest.of(pageNum-1, pageSize, sort);
                                      Page<TodoItem> page =
                                        repository.findAll(spec, pageRequest);
```

**关键差异：** Spring Data 分页从 0 开始（`pageNum - 1`），MyBatis-Plus 从 1 开始。Service 层需要做 `pageNum - 1` 的转换，对外 API 仍保持 `pageNum >= 1` 的语义。

**PageResult 转换：**

```java
// 当前：PageResult.from(mybatisPlusPage, converter)
// 迁移后：
public static <T, R> PageResult<R> from(Page<T> springPage, Function<T, R> converter) {
    PageResult<R> result = new PageResult<>();
    result.setRecords(springPage.getContent().stream().map(converter).toList());
    result.setTotal(springPage.getTotalElements());
    result.setPageNum(springPage.getNumber() + 1);  // 0-based → 1-based
    result.setPageSize(springPage.getSize());
    result.setTotalPages(springPage.getTotalPages());
    return result;
}
```

### 决策 6：建表策略

**选择：保留 `schema.sql`，`ddl-auto` 设为 `none`**

```yaml
# application-dev.yml
spring:
  jpa:
    hibernate:
      ddl-auto: none    # 不使用 JPA 自动建表
    show-sql: true      # 开发环境显示 SQL
  sql:
    init:
      mode: always
      schema-locations: classpath:schema.sql
```

**理由：** 保留 `schema.sql` 手动建表方式，与当前行为一致，表结构完全可控。`ddl-auto=none` 避免 JPA 意外修改表结构。索引定义在 `schema.sql` 中维护，Entity 上的 `@Index` 注解仅作文档用途。

**备选方案：**
- `ddl-auto=update`：开发便利，但生产环境风险高，且索引管理不可控
- `ddl-auto=create-drop`：仅适合测试，每次启动清空数据

### 决策 7：配置类变更

```
移除：MybatisPlusConfig（分页插件注册）
新增：无（Spring Data JPA 自动配置，无需手动注册分页）
```

启动类变更：
```
移除：@MapperScan("com.moneyflow.dao.mapper")
新增：@EnableJpaRepositories(basePackages = "com.moneyflow.dao.repository")
```

**模块结构变化：**

```
moneyflow-dao/
├── src/main/java/com/moneyflow/dao/
│   ├── mapper/                    ← 移除（MyBatis-Plus Mapper）
│   └── repository/                ← 新增（Spring Data Repository）
│       └── TodoItemRepository.java
```

### 架构关系（迁移后）

```
Controller 层 (moneyflow-web)
    │  @Valid, @RequestBody, @PathVariable
    │  不变
    v
Service 层 (moneyflow-service)
    │  TodoService / TodoServiceImpl
    │  LambdaQueryWrapper → Specification
    │  Page → Page (Spring Data)
    v
DAO 层 (moneyflow-dao)
    │  TodoItemRepository extends JpaRepository + JpaSpecificationExecutor
    │  ← 替代 TodoItemMapper extends BaseMapper
    v
Spring Data JPA / Hibernate
    │  自动生成 SQL
    │  自动管理 EntityManager、事务
    v
Database (H2 / MySQL)
    │  表结构不变
```

## Risks / Trade-offs

- **[风险] N+1 查询问题**：JPA 懒加载可能在关联查询时产生 N+1 问题。→ 缓解：当前 Todo 模块无关联关系，暂无风险。后续模块引入关联时使用 `@EntityGraph` 或 `JOIN FETCH` 解决。
- **[风险] Specification 代码可读性**：相比 `LambdaQueryWrapper`，`Specification` 的 Criteria API 写法更冗长。→ 缓解：封装通用的 Specification 构建工具方法，减少重复代码。
- **[权衡] schema.sql vs ddl-auto**：保留 schema.sql 意味着需要手动维护 SQL 与 Entity 的一致性。→ 对于当前项目规模（单表），手动维护成本可接受，且提供了更好的表结构控制力。
- **[权衡] 分页 0-based vs 1-based**：Spring Data 分页从 0 开始，需要在 Service 层做转换。→ 一次性适配，对外 API 语义不变，前端无感知。
