# Backend Framework Specification

## Purpose

提供后端 Maven 多模块脚手架及开箱即用的基础设施能力，包括分层模块结构、统一响应封装、全局异常处理、参数校验、API 文档、跨域配置和多环境配置，使后续所有业务功能开发都基于统一的后端框架约定。

## Requirements

### Requirement: Maven 多模块项目结构
后端 SHALL 采用 Maven 多模块结构，包含 parent、common、model、dao、service、web 六个模块。模块间依赖关系 SHALL 遵循 web -> service -> dao -> model -> common 的分层约束。

#### Scenario: 项目构建成功
- **WHEN** 在 backend/ 目录执行 `mvn clean package`
- **THEN** 所有模块按依赖顺序编译打包成功，无循环依赖错误

#### Scenario: 模块依赖方向正确
- **WHEN** 检查各模块 pom.xml 的依赖声明
- **THEN** common 不依赖任何其他业务模块，model 仅依赖 common，dao 仅依赖 model，service 仅依赖 dao，web 依赖 service

### Requirement: 统一响应封装
后端所有 REST API SHALL 返回统一的 JSON 响应格式 `Result<T>`，包含 code（状态码）、message（提示信息）、data（业务数据）三个字段。

#### Scenario: 成功响应格式
- **WHEN** 调用任意 API 且请求成功
- **THEN** 返回 JSON 格式为 `{"code": 200, "message": "success", "data": <业务数据>}`

#### Scenario: 失败响应格式
- **WHEN** 调用任意 API 且发生业务异常
- **THEN** 返回 JSON 格式为 `{"code": <错误码>, "message": <错误描述>, "data": null}`

### Requirement: 全局异常处理
后端 SHALL 提供全局异常处理机制，捕获未处理异常并返回统一格式的错误响应，不暴露内部堆栈信息。

#### Scenario: 业务异常处理
- **WHEN** Controller 抛出已定义的业务异常
- **THEN** 返回对应错误码和错误信息，HTTP 状态码为 200

#### Scenario: 未知异常处理
- **WHEN** 发生未捕获的运行时异常
- **THEN** 返回通用错误码和"系统内部错误"提示，HTTP 状态码为 500，不暴露堆栈

#### Scenario: 参数校验异常处理
- **WHEN** 请求参数不满足校验规则
- **THEN** 返回参数错误码和具体校验失败信息，HTTP 状态码为 400

### Requirement: 参数校验
后端 SHALL 支持基于 JSR 380 注解的参数校验，在 Controller 层通过注解声明式校验请求参数。

#### Scenario: 校验通过
- **WHEN** 请求参数满足所有校验注解约束
- **THEN** 请求正常进入业务逻辑

#### Scenario: 校验失败
- **WHEN** 请求参数违反校验注解约束
- **THEN** 返回 400 状态码和具体违反的校验规则描述

### Requirement: API 文档
后端 SHALL 集成 SpringDoc OpenAPI，自动生成 API 文档并提供 Swagger UI 可访问界面。

#### Scenario: Swagger UI 可访问
- **WHEN** 后端服务启动后访问 `/swagger-ui.html` 或等效路径
- **THEN** 展示所有 REST API 的文档页面，包含接口路径、参数、响应格式

### Requirement: CORS 跨域配置
后端 SHALL 提供全局 CORS 配置，允许前端开发服务器的跨域请求。

#### Scenario: 开发环境跨域
- **WHEN** 前端开发服务器（如 `http://localhost:5173`）发送跨域请求
- **THEN** 后端返回正确的 CORS 响应头，请求不被浏览器拦截

### Requirement: 多环境配置
后端 SHALL 支持多环境配置文件（至少包含 dev 和 prod），通过 Spring Profile 切换。

#### Scenario: 开发环境配置激活
- **WHEN** 以 `--spring.profiles.active=dev` 启动
- **THEN** 加载 dev 环境配置（开发数据库连接、调试日志级别等）

#### Scenario: 生产环境配置激活
- **WHEN** 以 `--spring.profiles.active=prod` 启动
- **THEN** 加载 prod 环境配置（生产数据库连接、正式日志级别等）
