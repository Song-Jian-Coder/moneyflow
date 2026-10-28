# Database Design Specification

## Purpose

定义项目数据库设计的统一规范，涵盖主键策略、公共字段、字段约束、索引策略和迁移文件规范，确保所有数据表设计一致、可维护、可扩展。技术栈为 Spring Boot + JPA + MySQL + Flyway。

## Requirements

### Requirement: 主键策略
所有表 SHALL 采用双 ID 策略：内部主键使用自增 BIGINT，对外业务 ID 使用 UUID（VARCHAR(36)）。

#### Scenario: 内部主键定义
- **WHEN** 创建新表
- **THEN** 主键字段为 `id BIGINT AUTO_INCREMENT PRIMARY KEY`，用于表间关联和内部索引

#### Scenario: 对外业务ID定义
- **WHEN** 创建新表
- **THEN** 必须包含 `biz_id VARCHAR(36) NOT NULL UNIQUE` 字段，用于 API 接口暴露，防止主键枚举攻击

#### Scenario: 业务ID自动生成
- **WHEN** 插入新记录且未指定 biz_id
- **THEN** 通过 JPA `@PrePersist` 自动生成 UUID 作为 biz_id

### Requirement: 公共字段
每张业务表 SHALL 包含以下 8 个公共字段：`id`、`biz_id`、`created_at`、`updated_at`、`created_by`、`updated_by`、`is_deleted`、`version`。

#### Scenario: 审计时间字段
- **WHEN** 创建新表
- **THEN** 必须包含 `created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP` 和 `updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP`

#### Scenario: 审计操作人字段
- **WHEN** 创建新表
- **THEN** 必须包含 `created_by VARCHAR(64) NOT NULL DEFAULT 'system'` 和 `updated_by VARCHAR(64) NOT NULL DEFAULT 'system'`

#### Scenario: 逻辑删除字段
- **WHEN** 创建新表
- **THEN** 必须包含 `is_deleted TINYINT(1) NOT NULL DEFAULT 0`，0 表示正常，1 表示已删除

#### Scenario: 乐观锁版本字段
- **WHEN** 创建新表
- **THEN** 必须包含 `version INT NOT NULL DEFAULT 0`，通过 JPA `@Version` 注解实现乐观锁控制

### Requirement: 字段约束原则
字段约束 SHALL 遵循"默认 NOT NULL"原则，仅在明确"确实无值且无法给默认值"的场景允许 NULL。

#### Scenario: NOT NULL 为默认选择
- **WHEN** 设计表字段
- **THEN** 主键、业务ID、时间字段、状态枚举、布尔字段、数值字段、必填字符串均 MUST 定义为 NOT NULL，并使用合理默认值

#### Scenario: 允许 NULL 的场景
- **WHEN** 字段确实表示"没有值"且无法给默认值
- **THEN** 允许定义为 NULL，仅限以下场景：可选外键（如 reply_to_id）、表示"从未发生"的时间（如 deleted_at）、可选的大文本/JSON 扩展字段（如 metadata）

#### Scenario: 字符串空值处理
- **WHEN** 字符串字段业务上可以为空
- **THEN** MUST 使用空串 `DEFAULT ''` 代替 NULL

#### Scenario: 状态枚举字段
- **WHEN** 设计状态或类型字段
- **THEN** MUST 使用 VARCHAR + NOT NULL + DEFAULT 定义，如 `status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE'`

### Requirement: 索引策略
索引设计 SHALL 遵循组合索引优先、外键必建、单表不超过 5 个索引的原则。

#### Scenario: 组合索引设计
- **WHEN** 查询同时包含 WHERE 和 ORDER BY 条件
- **THEN** MUST 创建组合索引，高选择性字段在前，排序字段在后，如 `INDEX idx_conversation_user_created (user_id, created_at DESC)`

#### Scenario: 外键字段索引
- **WHEN** 表中存在外键关联字段
- **THEN** MUST 为外键字段创建索引（MySQL 不自动为外键建索引）

#### Scenario: 业务ID唯一索引
- **WHEN** 创建新表
- **THEN** MUST 为 biz_id 创建唯一索引 `UNIQUE INDEX uk_{table}_biz_id (biz_id)`

#### Scenario: 低基数字段处理
- **WHEN** 字段基数很低（如 status 仅有 2-3 个值）
- **THEN** 不单独建索引，可放在组合索引末尾作为辅助筛选条件

#### Scenario: 索引数量限制
- **WHEN** 为表设计索引
- **THEN** 单表索引数量 MUST NOT 超过 5 个，超过时需审视查询模式是否合理

### Requirement: Flyway 迁移文件规范
数据库变更 SHALL 通过 Flyway 迁移文件管理，遵循版本化、不可变、单一职责原则。

#### Scenario: 文件命名规范
- **WHEN** 创建迁移文件
- **THEN** 文件 MUST 放置在 `src/main/resources/db/migration/` 目录，命名格式为 `V{版本号}__{描述}.sql`（双下划线分隔）

#### Scenario: 迁移文件不可变
- **WHEN** 迁移文件已执行过
- **THEN** MUST NOT 修改已执行的迁移文件内容（Flyway 会校验 checksum），变更只能通过新增迁移文件实现

#### Scenario: 单一职责
- **WHEN** 编写迁移文件
- **THEN** 每个文件 SHOULD 只做一件事（建表、加字段、加索引分开写），便于定位问题和回滚

#### Scenario: 回滚注释
- **WHEN** 编写迁移文件
- **THEN** 文件头部 MUST 包含回滚 SQL 注释，格式为 `-- 回滚：{回滚SQL语句}`

### Requirement: 字符集与引擎
所有表 SHALL 使用 InnoDB 引擎和 utf8mb4 字符集。

#### Scenario: 字符集配置
- **WHEN** 创建新表
- **THEN** MUST 指定 `DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci`，支持完整 Unicode（含 emoji）

#### Scenario: 存储引擎
- **WHEN** 创建新表
- **THEN** MUST 使用 `ENGINE=InnoDB`，以支持事务和行级锁
