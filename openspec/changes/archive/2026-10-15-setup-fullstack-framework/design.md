# Design

## Context

项目当前为空仓库，仅有 OpenSpec + Superpowers 框架目录。需要从零搭建前后端开发框架基座。约束条件：后端 Java 17 + Spring Boot 3.x + Maven 多模块，前端 React + Vite + TypeScript，两者独立构建。认证鉴权不在本次范围内。

## Goals / Non-Goals

**Goals:**
- 搭建可运行的 Maven 多模块后端骨架，内置统一响应封装、异常处理、参数校验、API 文档、CORS、多环境配置
- 搭建可运行的 React + Vite + TypeScript 前端骨架，内置路由、HTTP 客户端封装、API 代理
- 后端启动后提供一个 Health Check 接口用于验证框架工作正常
- 前端启动后能成功通过代理调用后端 Health Check 接口

**Non-Goals:**
- 不实现认证鉴权（后续变更）
- 不实现具体业务功能
- 不配置 CI/CD 流水线
- 不引入 Docker 容器化

## Decisions

### 决策 1：Maven 多模块分层结构

```
backend/
├── pom.xml                          (parent: 统一版本管理 + 依赖管理)
├── moneyflow-common/                (公共层: 工具类、常量、通用异常定义)
│   └── pom.xml
├── moneyflow-model/                 (数据模型层: 实体类、DTO、VO、统一响应 Result<T>)
│   └── pom.xml                      依赖: common
├── moneyflow-dao/                   (数据访问层: MyBatis-Plus Mapper、数据源配置)
│   └── pom.xml                      依赖: model
├── moneyflow-service/               (业务逻辑层: Service 接口与实现)
│   └── pom.xml                      依赖: dao
└── moneyflow-web/                   (Web 层: Controller、全局配置、启动类)
    └── pom.xml                      依赖: service
```

**理由**：三层拆分（common/model/dao/service/web）在关注点分离和复杂度之间取得平衡。parent pom 通过 `<dependencyManagement>` 统一管理所有第三方依赖版本，子模块不重复声明版本号。后续如需拆微服务，可从三层结构平滑升级到领域模块结构。

**备选方案**：
- 单模块：过于简单，无法提供清晰的分层约定
- 领域拆分（auth/account/transaction）：当前无业务，过度设计

### 决策 2：统一响应封装设计

```java
// 位于 moneyflow-model 模块
public class Result<T> {
    private int code;        // 状态码：200=成功，其他=各类错误
    private String message;  // 提示信息
    private T data;          // 业务数据

    // 静态工厂方法
    public static <T> Result<T> success(T data);
    public static <T> Result<T> error(int code, String message);
}
```

**理由**：统一封装让前端可以用一致的方式处理所有 API 响应，减少前端判断逻辑。

### 决策 3：全局异常处理设计

```
Controller 层
    │
    ├─ 业务异常 (BusinessException) → @ExceptionHandler → Result.error(code, msg), HTTP 200
    ├─ 参数校验异常 (MethodArgumentNotValidException) → Result.error(400, 校验详情), HTTP 400
    └─ 未知异常 (Exception) → Result.error(500, "系统内部错误"), HTTP 500, 记录日志
```

位于 moneyflow-web 模块，通过 `@RestControllerAdvice` 实现。自定义 `BusinessException` 定义在 moneyflow-common 模块。

### 决策 4：Spring Boot 版本与核心依赖

| 依赖 | 版本 | 理由 |
|------|------|------|
| Spring Boot | 3.2.x | 当前稳定主线，Java 17 原生支持 |
| MyBatis-Plus | 3.5.x | 国内主流 ORM 增强，代码生成器内置 |
| SpringDoc OpenAPI | 2.x | Spring Boot 3 兼容的 API 文档方案 |
| Lombok | latest | 减少样板代码 |
| MySQL Connector | 8.x | MySQL 8.0 官方驱动 |
| Validation | spring-boot-starter-validation | JSR 380 参数校验 |

**备选方案**：
- JPA/Hibernate：标准化 ORM，但复杂 SQL 场景灵活性不如 MyBatis-Plus
- Springfox Swagger：不兼容 Spring Boot 3，已弃用

### 决策 5：前端项目结构

```
frontend/
├── package.json
├── vite.config.ts               (Vite 配置 + API 代理)
├── tsconfig.json
├── index.html
└── src/
    ├── main.tsx                  (入口)
    ├── App.tsx                   (根组件 + 路由配置)
    ├── api/
    │   └── request.ts            (Axios 封装 + 拦截器)
    ├── pages/
    │   ├── Home.tsx              (首页)
    │   └── NotFound.tsx          (404 页面)
    └── types/
        └── api.ts                (API 类型定义，含 Result<T>)
```

**理由**：最小化骨架，只提供路由 + HTTP 封装 + API 代理三个核心基础设施。后续业务页面按 `pages/` 目录扩展。

### 决策 6：前后端联调方案

```
开发环境数据流：

Browser ──> Vite Dev Server (:5173) ──proxy──> Spring Boot (:8080)
                    │                                │
                    │  /api/** 代理转发                │  /api/health
                    └────────────────────────────────┘
```

Vite `server.proxy` 配置将 `/api` 前缀的请求代理到 `http://localhost:8080`。后端 Controller 统一使用 `/api` 前缀路径。

## Risks / Trade-offs

- **[风险] MySQL 未安装**：部分开发者本地可能没有 MySQL 8.0。→ 缓解：dev 环境可配置 H2 内存数据库作为临时替代（后续可选）。
- **[风险] Maven + npm 双构建**：前后端独立构建，没有一键全量构建。→ 缓解：当前阶段可接受，后续可加 Makefile 或脚本统一。
- **[权衡] 三层 vs 单模块**：三层增加了 pom.xml 数量，但提供了清晰的分层约定。→ 对于框架脚手架，约定比简洁更重要。
