# Tasks

## 1. 后端 Maven 多模块骨架搭建

- [x] 1.1 创建 `backend/pom.xml`（parent pom），声明 Spring Boot 3.2.x parent、Java 17、所有子模块列表、`<dependencyManagement>` 统一管理 MyBatis-Plus / SpringDoc / MySQL / Lombok 版本号。验证：`mvn validate` 在 backend/ 目录执行成功
- [x] 1.2 创建 `backend/moneyflow-common/pom.xml`，声明 parent 为 moneyflow-parent，添加 Lombok 依赖。创建基础包目录 `src/main/java/com/moneyflow/common/`。验证：`mvn compile` 通过
- [x] 1.3 创建 `backend/moneyflow-model/pom.xml`，声明 parent + 依赖 moneyflow-common。创建基础包目录 `src/main/java/com/moneyflow/model/`。验证：`mvn compile` 通过
- [x] 1.4 创建 `backend/moneyflow-dao/pom.xml`，声明 parent + 依赖 moneyflow-model，添加 MyBatis-Plus + MySQL Connector 依赖。创建基础包目录 `src/main/java/com/moneyflow/dao/`。验证：`mvn compile` 通过
- [x] 1.5 创建 `backend/moneyflow-service/pom.xml`，声明 parent + 依赖 moneyflow-dao。创建基础包目录 `src/main/java/com/moneyflow/service/`。验证：`mvn compile` 通过
- [x] 1.6 创建 `backend/moneyflow-web/pom.xml`，声明 parent + 依赖 moneyflow-service，添加 spring-boot-starter-web / spring-boot-starter-validation / springdoc-openapi 依赖，配置 spring-boot-maven-plugin。创建基础包目录 `src/main/java/com/moneyflow/web/` 和启动类 `MoneyFlowApplication.java`。验证：`mvn compile` 通过
- [x] 1.7 在 backend/ 根目录执行 `mvn clean package -DskipTests`，验证所有模块按依赖顺序编译打包成功，无循环依赖

## 2. 后端统一响应封装 + 异常处理

- [x] 2.1 在 moneyflow-model 中创建 `Result<T>` 类（`com.moneyflow.model.result.Result`），包含 code / message / data 字段和 success / error 静态工厂方法。验证：编译通过
- [x] 2.2 在 moneyflow-common 中创建 `BusinessException` 类（`com.moneyflow.common.exception.BusinessException`），包含 code / message 字段。验证：编译通过
- [x] 2.3 在 moneyflow-web 中创建 `GlobalExceptionHandler`（`com.moneyflow.web.handler.GlobalExceptionHandler`），使用 `@RestControllerAdvice`，处理 BusinessException（HTTP 200）、MethodArgumentNotValidException（HTTP 400）、Exception（HTTP 500）。验证：编译通过

## 3. 后端基础设施配置

- [x] 3.1 在 moneyflow-web 中创建 CORS 配置类 `CorsConfig`（`com.moneyflow.web.config.CorsConfig`），允许 `http://localhost:5173` 跨域。验证：编译通过
- [x] 3.2 在 moneyflow-web 中创建 SpringDoc 配置，设置 API 文档标题和版本。验证：编译通过
- [x] 3.3 在 moneyflow-web 的 `src/main/resources/` 下创建 `application.yml`（公共配置）、`application-dev.yml`（dev 环境：H2 内存数据库、debug 日志）、`application-prod.yml`（prod 环境：MySQL 连接、info 日志）。验证：文件存在且 YAML 格式正确
- [x] 3.4 在 moneyflow-web 中创建 Health Check 控制器 `HealthController`（`com.moneyflow.web.controller.HealthController`），提供 `GET /api/health` 接口返回 `Result.success("ok")`。验证：编译通过

## 4. 后端启动验证

- [x] 4.1 以 dev profile 启动后端应用（`mvn spring-boot:run -pl moneyflow-web -Dspring-boot.run.profiles=dev`），验证应用启动成功且 `/api/health` 接口返回 `{"code":200,"message":"success","data":"ok"}`。验证：curl 或浏览器访问返回正确 JSON

## 5. 前端项目骨架搭建

- [x] 5.1 在项目根目录执行 `npm create vite@latest frontend -- --template react-ts` 创建 React + TypeScript 项目，验证 `frontend/` 目录生成且 `npm install` 成功
- [x] 5.2 安装前端依赖：`npm install react-router-dom axios`，验证 package.json 包含新依赖

## 6. 前端路由 + 页面骨架

- [x] 6.1 创建 `frontend/src/pages/Home.tsx`（首页组件，展示"MoneyFlow 框架已就绪"）和 `frontend/src/pages/NotFound.tsx`（404 页面组件）。验证：文件存在且为有效 TSX
- [x] 6.2 修改 `frontend/src/App.tsx`，集成 React Router，配置路由：`/` 渲染 Home，`*` 渲染 NotFound。验证：`npm run dev` 启动后浏览器访问 `http://localhost:5173` 展示首页

## 7. 前端 HTTP 客户端封装

- [x] 7.1 创建 `frontend/src/types/api.ts`，定义 `Result<T>` 接口（与后端 Result 对应）。验证：TypeScript 编译无错误
- [x] 7.2 创建 `frontend/src/api/request.ts`，封装 Axios 实例，配置请求拦截器（添加 Content-Type 头）和响应拦截器（解析 Result 格式、统一错误处理）。验证：TypeScript 编译无错误

## 8. 前端 API 代理 + 前后端联调

- [x] 8.1 修改 `frontend/vite.config.ts`，配置 `server.proxy` 将 `/api` 请求代理到 `http://localhost:8080`。验证：配置语法正确
- [x] 8.2 在 Home.tsx 中添加 useEffect 调用 `/api/health` 接口并展示返回结果。验证：后端和前端同时启动时，首页展示 health check 响应数据

## Workflow follow-up

- 归档变更：实现完成并通过审查后执行 `openspec archive`
- 同步规范：执行 `openspec sync` 将 delta specs 合并到主规范库
- 更新 config.yaml：将已确定的技术栈信息更新到 `openspec/config.yaml` 的 context 字段
